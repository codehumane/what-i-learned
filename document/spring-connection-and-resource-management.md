# Connection and Resource Management

https://docs.spring.io/spring-amqp/reference/html/#connections

리소스 관리 중에서도 "spring-rabbit" 모듈에 관한 내용임을 언급.

- AMQP 모델은 제너릭<sup>generic</sup>하지만,
- 리소스 관리에 있어서 세부사항은 브로커 구현체에 따라 다름.
- 여기서는 "spring-rabbit" 모듈의 구현 내용을 이야기.

가장 먼저 `CachingConnectionFactory` 간단 소개.

- RabbitMQ 브로커의 연결을 관리하는 중앙 컴포넌트는 `ConnectionFactory` 인터페이스.
- 이 구현체들은 `org.springframework.amqp.rabbit.connection.Connection`(`com.rabbitmq.client.Connection`의 래퍼 클래스) 인스턴스들을 제공하는 책임을 가짐.
- `CachingConnectionFactory`가 유일하게 제공되는 구현체.
- 이는 기본으로 싱글 커넥션 프록시를 생성.
- AMQP에서의 메시징 작업 단위<sup>unit of work</sup>는 사실상 "channel"이므로 커넥션 공유가 가능.
- 이는 JMS에서의 커넥션과 세션의 관계와 유사.
- 커넥션 인스턴스는 `createChannel` 메서드를 제공.
- `CachingConnectionFactory` 구현체는 이들 채널의 캐싱을 지원하고, 트랜잭셔널 여부에 따라 별도의 캐시를 유지.
- 이 내용은 `CachingConnectionFactor#getChannel` 코드 참고.

```java
private Channel getChannel(ChannelCachingConnectionProxy connection, boolean transactional) {
    // ...
    LinkedList<ChannelProxy> channelList;
    if (this.cacheMode == CacheMode.CHANNEL) {
        channelList = transactional ? this.cachedChannelsTransactional
                : this.cachedChannelsNonTransactional;
    }
    // ...
    ChannelProxy channel = null;
    if (connection.isOpen()) {
        synchronized (channelList) {
            while (!channelList.isEmpty()) {
                channel = channelList.removeFirst();
                if (logger.isTraceEnabled()) {
                    logger.trace(channel + " retrieved from cache");
                }
                if (channel.isOpen()) {
                    break;
                }
                else {
                    try {
                        Channel target = channel.getTargetChannel();
                        if (target != null) {
                            target.close();
                            /*
                                *  To remove it from auto-recovery if so configured,
                                *  and nack any pending confirms if PublisherCallbackChannel.
                                */
                        }
                    }
}
```

- `CachingConnectionFactory` 인스턴스를 만들 때 `hostname`, `username`, `password`, `channelCacheSize` 설정이 가능.

당연한 얘기지만, channel과 connection 혼동 X.

- 채널에 대한 내용은 [Channels Guide](https://www.rabbitmq.com/channels.html) 참고.
- 커넥션에 대한 내용은 [Connections Guide](https://www.rabbitmq.com/connections.html) 참고.

> Some applications need multiple logical connections to the broker. However, it is undesirable to keep many TCP connections open at the same time because doing so consumes system resources and makes it more difficult to configure firewalls. AMQP 0-9-1 connections are multiplexed with channels that can be thought of as "lightweight connections that share a single TCP connection".
> 
> Every protocol operation performed by a client happens on a channel. Communication on a particular channel is completely separate from communication on another channel, therefore every protocol method also carries a channel ID (a.k.a. channel number), an integer that both the broker and clients use to figure out which channel the method is for.

채널 뿐만 아니라 커넥션도 캐시 가능.

- `CachingConnectionFactory`는 1.3 버전부터 커넥션 캐시 설정이 가능.
- `createConnection()` 호출을 통해 새로운 커넥션을 만들고,
- 커넥션 클로즈는 이를 캐시에 반환(캐시 사이즈가 꽉 차지 않았다면).
- 이렇게 분리된 연결을 가지는 것이 유용할 때가 있음.
- 예컨대, HA 클러스터로부터의 컨슘하고, 로드밸런서를 통해 여러 클러스터 멤버로 연결해야 할 때.
- 이 기능을 사용하려면 `cacheMode`를 `CacheMode.CONNECTION`으로 설정.
- 이 값은 커넥셧 갯수를 제한하는 것이 아니라 얼마나 많은 유휴 오픈 커넥션을 허용할 것인지에 대한 것임에 유의.
- 그리고 `CONNECTION` 모드를 사용하면 [큐, 바인딩, 익스체인지의 자동 선언](https://docs.spring.io/spring-amqp/reference/html/#automatic-declaration)이 불가.

연결 갯수 제한도 가능.

- 1.5.5 버전부터 `connectionLimit` 프로퍼티 제공.
- 이는 연결의 총 갯수를 제한함.
- 제한 수치에 도달하면 `channelCheckoutTimeLimit` 동안 유휴 연결이 나타날 때까지 기다림.
- 이 시간을 넘어가면 `AmqpTimeoutException`이 던져짐.

채널 캐시 크기에 대한 부연 설명.

- 캐시 사이즈는 제한이 아니라, 단지 캐시될 수 있는 채널의 갯수임에 주의.
- 1.6 버전부터는 캐시 사이즈의 기본 값이 1에서 25로 늘어남.
- 대용량 환경에서의 적은 캐시는 채널의 생성과 닫힘을 자주 일으킴.
- 이 값을 늘려 이런 오버헤드를 줄이는 것.
- RabbitMQ 어드민 UI를 통해 이를 모니터링하고 적절하게 변경해 줄 것.

아래는 `connection`을 생성하는 간단한 예시.

```java
CachingConnectionFactory connectionFactory = new CachingConnectionFactory("somehost");
connectionFactory.setUsername("guest");
connectionFactory.setPassword("guest");
Connection connection = connectionFactory.createConnection();
```

`SingleConnectionFactory`는 단위 테스트에서만 사용 가능.

- `CachingConnectionFactory`에 비해 단순.
- 채널을 캐시하지 않기 때문.
- 그리고 성능이나 장애 내성이 약함.
- 단위 테스트에서 사용하는 용도.
- 직접 `ConnectionFactory`를 만들어야 한다면, `AbstractConnectionFactory`를 출발점으로 삼기를 권장.

## AddressResolver

- 2.1.15부터 `AddressResolver` 사용 가능.
- 연결 주소를 리졸브 하기 위함.
- 이를 사용하면 `address`, `host`, `port` 프로퍼티 오버라이드 됨.

## Naming Connections

- 1.7부터 `ConnectionNameStrategy` 제공.
- 여기서 만들어진 이름으로 RabbitMQ 연결 아이디로 사용.
- 이 이름은 고유할 필요 없으며, 커넥션 식별자로 사용할 수는 없음.
- 관리 UI에도 노출되는 이름.

```java
connectionFactory.setConnectionNameStrategy(connectionFactory -> "MY_CONNECTION");

// 혹은 아래 방식으로 어플리케이션 프로퍼티를 주입할 수도 있다.
@Bean
public ConnectionNameStrategy cns() {
    return new SimplePropertyValueConnectionNameStrategy("spring.application.name");
}
@Bean
public ConnectionFactory rabbitConnectionFactory(ConnectionNameStrategy cns) {
    CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
    ...
    connectionFactory.setConnectionNameStrategy(cns);
    return connectionFactory;
}
```

## Blocked Connections and Resource Constraints

- 브로커와의 연결이 블럭킹 될 수 있음.
- 2.0부터는 `com.rabbitmq.client.BlockedListener`가 제공됨.
- 이를 통해 `AbstractConnectionFactory`가 보내주는 `ConnectionBlockedEvent`와 `ConnectionUnblockedEvent`를 받을 수 있음.
- 애플리케이션에서는 이를 받아 적절한 처리를 진행.
- 참고로, 스프링 부트 오토 컨피그레이션에 의해 만들어진 `CachingConnectionFactory`를 사용한다면, 커넥션이 블럭 당한 경우 애플리케이션이 동작을 중단하고, 브로커에 의해 블럭 당한 경우는 모든 클라이언트들이 작업을 멈춤.
- 만약 하나의 애플리케이션에서 프로듀서와 컨슈머 모두를 가지고 있다면, 프로듀서가 커넥션을 블럭킹(브로커에 더 이상 리소스가 없을 때)하고, 이로 인해 컨슈머가 연결을 풀어주지 못할 때(커넥션이 블럭 당했으므로), 데드락에 빠질 수 있음.
- 이를 피하고자 `CachingConnectionFactory` 인스턴스를 2개 이상 사용하길 권장.
- 1.7.7부터는 `AmqpResourceNotAvailableException`이 제공. `SimpleConnection#createChannel()`이 `Channel`을 생성하지 못할 때(`channelMax` 임계치에 다다르는 등의 이유로) 일어나며, `RetryPolicy`로 일정 백오프가 지난 뒤 연산을 복구하는 등의 처리 가능.

## Connecting to a Cluster

- 클러스터에 연결하려면 `CachingConnectionFactory`의 `addresses` 프로퍼티를 설정.

```java
@Bean
public CachingConnectionFactory ccf() {
    CachingConnectionFactory ccf = new CachingConnectionFactory();
    ccf.setAddresses("host1:5672,host2:5672,host3:5672");
    return ccf;
}
```

- 위 방식에서는, 새로운 연결이 만들어질 때마다, 각 호스트에 차례로 연결을 시도함.
- `shuffleAddresses` 속성으로 매 연결 시마다 이 순서를 섞을 수 있음.

## Routing Connection Factory

- 1.3부터는 `AbstractRoutingConnectionFactory`가 도입됨.
- 이는 여러 `ConnectionFactories`에 대한 매핑을 설정하고,
- `lookupKey`로 대상 `ConnectionFactory`를 런타임에 결정하는 역할을 담당.
- 일반적으로, 스레드 바운드 컨텍스트를 확인하는 구현 방식을 가지는데,
- 스프링 AMQP가 제공하는 `SimpleRoutingConnectionFactory`는,
- `SimpleResourceHolder`로부터 현재 스레드에 바운드 된 `lookupKey`를 얻어냄.
- 아래는 이 팩토리를 설정하는 방법 예시.

```xml
<bean id="connectionFactory"
      class="org.springframework.amqp.rabbit.connection.SimpleRoutingConnectionFactory">
	<property name="targetConnectionFactories">
		<map>
			<entry key="#{connectionFactory1.virtualHost}" ref="connectionFactory1"/>
			<entry key="#{connectionFactory2.virtualHost}" ref="connectionFactory2"/>
		</map>
	</property>
</bean>

<rabbit:template id="template" connection-factory="connectionFactory" />
```

```java
public class MyService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void service(String vHost, String payload) {
        SimpleResourceHolder.bind(rabbitTemplate.getConnectionFactory(), vHost);
        rabbitTemplate.convertAndSend(payload);
        SimpleResourceHolder.unbind(rabbitTemplate.getConnectionFactory());
    }

}
```

- 당연히, 사용 후 리소스를 바인드 해제시키는 것은 중요. 자세한 내용은 `AbstractRoutingConnectionFactory` 참고.
- 1.4부터는 `RabbitTemplate`이 SpEL `sendConnectionFactorySelectorExpression`과 `receiveConnectionFactorySelectorExpression` 프로퍼티를 지원.
- 이는 매 AMQP 프로토콜 인터택션 연산(`send`, `sendAndReceive`, `receive`, `receiveAndReply`) 마다 평가되어 `lookupKey`를 리졸브 함.
- `@vHostResolver.getVHost(#root)`와 같은 빈 참조를 표현식에 사용할 수도.
- `send` 연산에서는 보내지는 메시지가 루트 평가 객체가 되고,
- `receive` 연산에서는 `queueName`이 루트 평가 객체가 됨.
- 라우팅 알고리즘은 아래와 같음.
    - 셀렉터 표현식이 `null`이거나, `null`로 평가되거나, 제공된 `ConnectionFactory`가 `AbstractRoutingConnectionFactory` 인스턴스가 아니면, 기존처럼 `ConnectionFactory` 구현체에 의해 동작.
    - 평가 결과가 `null`이 아니더라도, `lookupKey`에 대한 대상 `ConnectionFactory`을 찾을 수 없고, `AbstractRoutingConnectionFactory`가 `lenientFallback = true`로 설정되어 있을 때도 마찬가지.
    - `AbstractRoutingConnectionFactory`는 이 때 폴백으로 `determineCurrentLookupKey()`를 기반으로 `routing` 구현체를 사용.
    - 하지만, `lenientFallback = false`이면 `IllegalStateException`이 발생.
- 1.4부터, 라우팅 커넥션 팩토리를 리스너 컨테이너에서도 설정할 수 있음.
- 여기서는 큐 이름 목록이 lookup key로 사용됨. 예컨대, `setQueueNames("thing1", "thing2")`로 설정되어 있다면, lookup key는 `[thing1,thing]`이 됨.
- 1.6.9부터는 `setLookupKeyQualifier`를 이용해, 리스너 컨테이너에 lookup key에 대한 qualifier 지정 가능.

## Publisher Confirms and Returns

- `CachingConnectionFactory`의 `publisherConfirmType` 속성을 `ConfirmType.CORRELATED`로 설정하고 `publisherReturns`를 `true`로 설정하면,
- 확인되고<sup>confirmed</sup>(correlation과 함께) 반환된<sup>returned</sup> 메시지를 지원함.
- 이 옵션들이 설정되면, 팩토리에 의해 `Channel` 인스턴스들이 생성되고, `PublisherCallbackChannel`로 래핑됨(콜백을 사용하기 위한).
- 이 채널이 얻어지면, 클라이언트는 이 `Channel`과 함께 `PublisherCallbackChannel.Listener`를 등록할 수 있음.
- `PublisherCallbackChannel` 구현체는 확인 또는 반환을 적절한 리스너로 라우팅하는 로직을 가짐.

## Connection and Channel Listeners

- 커넥션 팩토리는 `ConnectionListener`와 `ChannelListener` 구현체 등록을 지원.
- 커넥션과 채널의 이벤트들을 통지받는 것.

```java
@FunctionalInterface
public interface ConnectionListener {

    void onCreate(Connection connection);

    default void onClose(Connection connection) {
    }

    default void onShutDown(ShutdownSignalException signal) {
    }

}

@FunctionalInterface
public interface ChannelListener {

    void onCreate(Channel channel, boolean transactional);

    default void onShutDown(ShutdownSignalException signal) {
    }

}
```

## Runtime Cache Properties

Table 1. Cache properties for CacheMode.CHANNEL

| Property                      | Meaning                                                      |
| :---------------------------- | :----------------------------------------------------------- |
| ` connectionName`             | The name of the connection generated by the `ConnectionNameStrategy`. |
| ` channelCacheSize`           | The currently configured maximum channels that are allowed to be idle. |
| ` localPort`                  | The local port for the connection (if available). This can be used to correlate with connections and channels on the RabbitMQ Admin UI. |
| ` idleChannelsTx`             | The number of transactional channels that are currently idle (cached). |
| ` idleChannelsNotTx`          | The number of non-transactional channels that are currently idle (cached). |
| ` idleChannelsTxHighWater`    | The maximum number of transactional channels that have been concurrently idle (cached). |
| ` idleChannelsNotTxHighWater` | The maximum number of non-transactional channels have been concurrently idle (cached). |

Table 2. Cache properties for CacheMode.CONNECTION

| Property                       | Meaning                                                      |
| :----------------------------- | :----------------------------------------------------------- |
| ` connectionName:`             | The name of the connection generated by the `ConnectionNameStrategy`. |
| ` openConnections`             | The number of connection objects representing connections to brokers. |
| ` channelCacheSize`            | The currently configured maximum channels that are allowed to be idle. |
| ` connectionCacheSize`         | The currently configured maximum connections that are allowed to be idle. |
| ` idleConnections`             | The number of connections that are currently idle.           |
| ` idleConnectionsHighWater`    | The maximum number of connections that have been concurrently idle. |
| ` idleChannelsTx:`             | The number of transactional channels that are currently idle (cached) for this connection. You can use the `localPort` part of the property name to correlate with connections and channels on the RabbitMQ Admin UI. |
| ` idleChannelsNotTx:`          | The number of non-transactional channels that are currently idle (cached) for this connection. The `localPort` part of the property name can be used to correlate with connections and channels on the RabbitMQ Admin UI. |
| ` idleChannelsTxHighWater:`    | The maximum number of transactional channels that have been concurrently idle (cached). The localPort part of the property name can be used to correlate with connections and channels on the RabbitMQ Admin UI. |
| ` idleChannelsNotTxHighWater:` | The maximum number of non-transactional channels have been concurrently idle (cached). You can use the `localPort` part of the property name to correlate with connections and channels on the RabbitMQ Admin UI. |

