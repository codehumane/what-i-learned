
# AmqpTemplate

https://docs.spring.io/spring-amqp/reference/html/#amqp-template

- 다른 스프링 프레임워크들처럼, 중심 역할을 수행하는 "template"을 제공.
- 주요 연산들을 정의한 인터페이스는 `AmqpTemplate`.
- 주요 연산이라 함은 메시지들을 보내고 받는 일반적 행위들.
- 이름이 AMQP인 만큼 특정 구현체가 아닌 AMQP 프로토콜에 대응.
- 이 구현체 중 하나가(그리고 현재까지 하나뿐인) `RabbitTemplate`.

## Adding Retry Capabilities

- 브로커 연결 문제 등을 다루는데 `RetryTemplate`이 도움.
- 이 역시 [spring-retry](https://github.com/spring-projects/spring-retry)에 기반.

```java
@Bean
public RabbitTemplate rabbitTemplate() {
    var template = new RabbitTemplate(connectionFactory());
    var retryTemplate = new RetryTemplate();

    var backOffPolicy = new ExponentialBackOffPolicy();
    backOffPolicy.setInitialInterval(500);
    backOffPolicy.setMultiplier(10.0);
    backOffPolicy.setMaxInterval(10000);

    retryTemplate.setBackOffPolicy(backOffPolicy);
    template.setRetryTemplate(retryTemplate);
    return template;
}
```

- 1.4 버전부터는 `retryTemplate` 프로티에 더해 `recoveryCallback`도 지원.
- `RetryTemplate.execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback)`.

```java
retryTemplate.execute(
    new RetryCallback<Object, Exception>() {

        @Override
        public Object doWithRetry(RetryContext context) throws Exception {
            context.setAttribute("message", message);
            return rabbitTemplate.convertAndSend(exchange, routingKey, message);
        }
    },
    new RecoveryCallback<Object>() {

        @Override
        public Object recover(RetryContext context) throws Exception {
            Object message = context.getAttribute("message");
            Throwable t = context.getLastThrowable();
            // ...
            return null;
        }
    }
)
```

## Publishing is Asynchronous - How to Detect Successes and Failures

- 메시지 퍼블리싱은 비동기 메커니즘.
- 기본적으로 라우팅되지 못한 메시지는 RabbitMQ에 의해 drop 됨.
- 성공적인 퍼블리싱을 위해 비동기 confirm을 받을 수 있음.
- 아래의 2가지 실패 시나리오를 생각해보자.

```
1. 익스체인지로 퍼블리싱 했는데 매칭되는 목적지 큐가 하나도 없음.
2. 존재하지 않는 익스체인지로 퍼블리싱.
```

- 첫 번째 케이스는 퍼블리셔의 반환으로 대응이 가능(뒤에서 자세히 다룸).
- 두 번째 케이스는 메시지가 drop되고 아무런 반환도 되지 않음.
- 관련된 채널은 예외와 함께 닫힘.
- 기본적으로 이 예외는 로그에 남지만,
- `ChannelListener`를 `CachingConnectionFactory`에 등록해서 이 이벤트를 통지 받을 수 있음.

```
this.connectionFactory.addConnectionListener(new ConnectionListener() {

    @Override
    public void onCreate(Connection connection) {
    }

    @Override
    public void onShutDown(ShutdownSignalException signal) {
        ...
    }

})
```

- signal의 `reason` 프로퍼티에서 문제를 좀 더 확인할 수 있음.
- 보내는 측 스레드에서 이 예외를 감지하고 싶다면,
- `RabbitTemplate`에 `setChannelTransacted(true)`를 하고,
- `txCommit()`에서 예외를 받음.
- 하지만 트랜잭션은 성능을 크게 저하시킴.
- 따라서 트랜잭션 활성화는 신중히 고려할 것.

## Correlated Publisher Confirms and Returns

`RabbitTemplate`은  publisher confirms와 returns를 지원.

일단, 메시지 반환을 위한 내용들.

- 일단, template의 `mendatory` 속성이 `true`로 설정되어 있거나,
- `mandatory-expression`이 `true`로 평가되어야 함.
- [Publisher Confirms and Returns](https://docs.spring.io/spring-amqp/reference/html/#cf-pub-conf-ret) 참고.
- 반환 메시지는 `RabbitTemplate.ReturnsCallback`을 등록한 클라이언트에 전달됨.
- 등록은 `setReturnsCallback(ReturnsCallback callback)`을 통함.
- 콜백은 아래 메서드를 구현해야 함.

```java
void returnedMessage(ReturnedMessage returned);
```

`ReturnedMessage`는 아래 속성들을 가짐.

- `message` - 반환된 메시지 자체.
- `replyCode` - 반환된 이유를 나타내는 코드.
- `replyText` - 반환된 이유의 텍스트. 예를 들면, `NO_ROUTE`.
- `exchange` - 메시지가 보내진 exchange.
- `routingKey` - 사용된 라우팅 키.

다음으로, 퍼블리서 확인 이야기.

- `CachingConnectionFactory`의 `publisherConfirm` 속성이 `ConfirmType.CORRELATED`로 설정 되어 있어야 함.
- 확인은 `RabbitTemplate.ConfirmCallback`을 등록한 클라이언트에게 전달됨.
- 등록은 `setConfirmCallback(ConfirmCallback callback)`을 사용.
- 콜백은 아래 메서드를 구현해야 함.

```java
void confirm(CorrelationData correlationData, boolean ack, String cause);
```

- `CorrelationData`는 원본 메시지를 클라이언트가 보낼 때 제공되는 객체.
- `ack`는 `ack`를 받으면 참, `nack`를 받으면 거짓.
- `nack`의 경우 cause가 포함될 수 있음(생성됐다면).
- 예컨대, 존재하지 않는 exchange로 메시지를 보내면 브로커는 채널을 닫음.
- 이 경우 닫힘의 이유가 cause에 담김.

마지막으로, `CorrelationData`의 최근 변경들 이야기.

- 2.1부터 `CorrelationData` 객체는 `ListenableFuture`를 포함.
- `ConfirmCallback` 말고 이를 통해 결과를 얻을 수 있음.
- 아래는 그 예시.

```java
CorrelationData cd1 = new CorrelationData();
this.templateWithConfirmsEnabled.convertAndSend("exchange", queue.getName(), "foo", cd1);
assertTrue(cd1.getFuture().get(10, TimeUnit.SECONDS).isAck());
```

- `ListenableFuture<Confirm>`이므로 결과를 얻어내는 데 `get()`이나 비동기 콜백을 이용할 수 있음.
- `Confirm` 객체는 `ack`와 `reason`(nack인 경우)을 가지는 간단한 빈.
- 브로커가 만든 nack에 대해서는 이유가 주어지지 않음.
- 프레임워크에서 만든 nack에 대해서 만들어짐.
- 예컨대, 아직 처리 중인 `ack`에 대해 커넥션이 닫힌 경우.

한편, 확인과 반환 모두 활성화되어 있다면, `CorrelationData`는 반환 메시지와 함께 만들어짐. 단, `CorrelationData`가 고유한 `id`를 가지는 경우에 한함.

## Scoped Operations

- 일반적으로, 템플릿을 쓸 때,
- `Channel`을 캐시에서 체크아웃 (또는 생성) 하고,
- 연산을 위해 사용되고 재사용을 위해 캐시에 반환됨.
- 멀티 스레드 환경에서, 다음 연산에 같은 채널이 사용된다는 보장은 불가.
- 하지만, 몇 개의 연산이 모두 같은 채널에서 수행되는 것이 필요할 때가 있음.
- (어떤 상황인지 추정은 안 됨)
- 2.0부터는 `invoke` 메서드가 제공되고, 여기에 `OperationsCallback`을 쓸 수 있음.
- 인자로 주어진 `RabbitOperations`을 사용한 콜백 내의 모든 연산들은 동일한 전용 `Channel` 사용하게 됨.
- 사용이 끝나면 캐시로 반환되지 않고 닫힘.
- 만약 채널이 `PublisherCallbackChannel`이라면, 모든 확인이 수신된 이후에 다시 캐시로 반환.

```java
@FunctionalInterface
public interface OperationsCallback<T> {
    T doInRabbit(RabbitOperations operations);
}
```

- (다행이 어떤 상황에 필요한지 설명 이어짐)
- 이것이 필요한 한 가지 예는, `Channel`의 `waitForConfirms()` 메서드를 쓰길 원할 때.
- 이 메서드는 예전에는 Spring API로 노출되지 않았음. 채널이 일반적으로 캐시되고 공유되기 때문.
- 이제 `RabbitTemplate`은 `waitForConfirms(long timeout)`와 `waitForConfirmsOrDie(long timeout`을 제공함.
- 이는 `OperationsCallback` 범위 내에서 사용되는 채널로 위임.
- 범위 바깥에서는 이 메서드 사용 불가.

```java
Collection<?> messages = getMessagesToSend()
Boolean result = this.template.invoke(t -> {
    // convertAndSend는 메시지 단위
    messages.forEach(m -> t.convertAndSend(ROUTE, m));
    // wait은 전용 channel 단위
    t.waitForConfirmsOrDie(10_000);
    return true;
});
```

- confirm을 이런 식으로 사용한다면, 그리고 returns도 비활성화 되어 있다면,
- correlating confirm를 위한 많은 인프라 설정이 필요 없어짐.
- 2.2 버전부터는 커넥션 팩토리가 `publisherConfirmType`을 지원하며,
- 이를 `ConfirmType.SIMPLE`로 설정한다면 인프라를 피하고 confirm 절차는 좀 더 효율적.
- 그리고 `RabbitTemplate`은 전송할 메시지의 `MessageProperties`에 `publisherSequenceNumber` 설정 가능.
- confirm을 확인하고 싶다면, 아래와 같이 오버로딩 된 `invoke` 메서드를 이용하면 됨.

```java
public <T> T invoke(
    OperationsCallback<T> action,
    ConfirmCallback acks,
    ConfirmCallback nacks
);
```

## Strict Message Ordering in a Multi-Threaded Environment

- Scoped Operations에서 다룬 내용은 연산들이 모두 같은 스레드에서 수행될 때에만 적용됨.
- 하지만 아래의 경우를 생각해보자.

```
1. thread-1이 큐에 메시지를 보내고, thread-2에게 작업을 위임.
2. thread-2가 동일한 큐로 메시지를 보냄.
```

- RabbitMQ의 비동기 특성 때문에, 그리고 캐시된 채널의 사용 때문에,
- 동일한 채널이 사용되고 따라서 큐에 메시지가 의도한 순서대로 도착했음을 보장할 수 없음.
- 이를 해결하는 한 가지 방법은 채널 캐시 크기를 1로 가져가는 것(`channelCheckoutTimeout`과 함께).

```java
@SpringBootApplication
public class Application {
    
    private static final Logger log = LoggerFactory.getLogger(Applicatino.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    TaskExecutor exec() {
        ThreadPoolTaskExecutor exec = new ThreadPoolTaskExecutor();
        exec.setCorePoolSize(10);
        return exec;
    }

    @Bean
    CachingConnectionFactory ccf() {
        CachingConnectionFactory ccf = new CachingConnectionFactory("localhost");
        CachingConnectionFactory publisherCF = (CachingConnectionFactory) ccf.getPublisherConnectionFactory();
        publisherCF.setChannelCacheSize(1);
        publisherCF.setChannelCheckoutTimeout(1000L);
        return ccf;
    }

    @RabbitListener(queues = "queue")
    void listen(String in) {
        log.info(in);
    }

    @Bean
    Queue queue() {
        return new Queue("queue");
    }

    @Bean
    public ApplicationRunner runner(Service service, TaskExecutor exec) {
        return args -> {
            exec.execute(() -> service.mainService("test"));
        }
    }
}

@Component
class Service {
    private static final Logger LOG = LoggerFactory.getLogger(Service.class);
    private final RabbitTemplate template;
    private final TaskExecutor exec;

    Service(RabbitTemplate template, TaskExecutor exec) {
        template.setUsePublisherConnection(true);
        this.template = template;
        this.exec = exec;
    }

    void mainService(String toSend) {
        LOG.info("Publishing from main service");
        this.template.convertAndSend("queue", toSend);
        this.exec.execute(() -> secondaryService(toSend.toUpperCase()));
    }

    void secopndaryService(String toSend) {
        LOG.info("Publishing from secondary service");
        this.template.convertAndSend("queue", toSend);
    }
}
```

- 위에서 퍼블리싱이 2개의 서로 다른 스레드에서 이뤄지긴 하지만,
- 채널이 1개로 설정되어 있으므로 2개는 같은 채널을 사용하게 됨.
- 2.3.7 이후 버전에서는 `ThreadChannelConnectionFactory`가 제공되며,
- `prepareContextSwitch`와 `switchContext`를 이용해서 순서를 보장하는 처리가 가능.

```java
@SpringBootApplication
public class Application {
    
    // ... 위 코드와 동일

    @Bean
    ThreadChannelConnectionFactory tccf() {
        ConnectionFactory rabbitConnectionFactory = new ConnectionFactory();
        rabbitConnectionFactory.setHost("localhost");
        return new ThreadChannelConnectionFactory(rabbitConnectionFactory);
    }

    // ... 위 코드와 동일

@Component
class Service {
    
    // ... 위 코드와 동일

    void mainService(String toSend) {
        LOG.info("Publishing from main service");
        this.template.convertAndSend("queue", toSend);
        Object context = this.connFactory.prepareSwitchContext();
        this.exec.execute(() -> secondaryService(toSend.toUpperCase(), context));
    }

    void secondaryService(String toSend, Object threadContext) {
        LOG.info("Publishing from secondary service");
        this.connFactory.switchContext(threadContext);
        this.template.convertAndSend("queue", toSend);
        this.connFactory.closeThreadChannel();
    }
}
```
