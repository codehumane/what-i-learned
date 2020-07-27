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

