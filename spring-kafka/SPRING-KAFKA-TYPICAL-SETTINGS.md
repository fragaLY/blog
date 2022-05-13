------------------------------------------------------------------------------------------------------------------------

![](./static/main.jpeg)

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 1: GOOD DAGS. D'YA LIKE DAGS?</h6>

:mechanical_arm: Hi, folks,

Asynchronous messaging is as beautiful as a painful one.
Apache Kafka Clients API solves a lot of problems and creates no less than it is.
For sure, if you didn't investigate the solution and chose the most appropriate semantic.
Just a kind reminder, there are three of them: at least once, at most once, exactly once.
I will not dive deeper into the difference between them.
I hope, you can understand it from the naming, such a beautiful naming, some developers are supposed to be so clear as it.

So, I am about the most common options you could face during the investigation and kick-off steps.

Hope this page will help you to gather some of the tons of information regarding Kafka setup using Spring Framework.

Trying to solve the dog's breakfast do not eat it.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 2: YOU SHOULD NEVER UNDERESTIMATE THE PREDICTABILITY OF CONSUMER.</h6>

<b>Consumer Settings</b>

|Setting|Values|Useful Links|
|:------|:-----|:-----------|
|group-id|'name by convention'|[group-id](https://docs.spring.io/spring-kafka/reference/html/#listener-group-id)|
|enable-auto-commit|'true/false'|[auto-commit](https://docs.spring.io/spring-kafka/reference/html/#committing-offsets)|
|auto-offset-reset|'true/false'|[auto-offset-reset](https://medium.com/lydtech-consulting/kafka-consumer-auto-offset-reset-d3962bad2665)|
|isolation.level|'read_committed/read_uncommited'|[isolation-level](https://www.waitingforcode.com/apache-kafka/isolation-level-apache-kafka-consumers/read)|
|retry-policy|'[1,...,2147483647]'|[retry-policy](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/retrytopic/RetryTopicConfigurer.html)|
|backoff-period|'i.e. 1000ms'|[backoff-period](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/retrytopic/RetryTopicConfigurer.html)|

[Common Listeners Settings](https://docs.spring.io/spring-kafka/reference/html/#container-props)

Sample of Spring Configuration:
``` yaml
spring:
  kafka:
    bootstrap-servers: http://localhost:9092, http://localhost:9093, http://localhost:9094
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.LongDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      group-id: notification-events-listener-group
      enable-auto-commit: false
      auto-offset-reset: earliest
      properties:
        isolation.level: read_committed
    template:
      default-topic: notification-event
    listener:
      ack-mode: manual

```

Sample of Consumer Configuration:

``` kotlin
@Configuration
@EnableKafka
class NotificationEventsConsumer {

    @Bean
    fun factory(
        errorHandler: KafkaErrorHandlingLogger,
        configurer: ConcurrentKafkaListenerContainerFactoryConfigurer,
        consumerFactory: ConsumerFactory<Any, Any>,
        retryTemplate: RetryTemplate
    ): ConcurrentKafkaListenerContainerFactory<Any, Any> {
        val factory = ConcurrentKafkaListenerContainerFactory<Any, Any>()
        configurer.configure(factory, consumerFactory)
        factory.setConcurrency(1)
        factory.containerProperties.ackMode = ContainerProperties.AckMode.MANUAL
        factory.setErrorHandler(errorHandler)
        factory.setRetryTemplate(retryTemplate)
        return factory
    }

    @Bean
    fun retry(backOffPolicy: FixedBackOffPolicy, retryPolicy: SimpleRetryPolicy): RetryTemplate {
        val template = RetryTemplate()
        template.setBackOffPolicy(backOffPolicy)
        template.setRetryPolicy(retryPolicy)
        return template
    }

    @Bean
    fun retryPolicy() = SimpleRetryPolicy(3)

    @Bean
    fun backOffPolicy(): FixedBackOffPolicy {
        val policy = FixedBackOffPolicy()
        policy.backOffPeriod = 1000L //default value
        return policy
    }
}

@Component
class KafkaErrorHandlingLogger : ErrorHandler {

    private val logger = LoggerFactory.getLogger(javaClass)

    override fun handle(exception: java.lang.Exception, data: ConsumerRecord<*, *>?) {
        logger.error("[CONSUMER] Error during processing the topic [${data?.topic()}] message[key = [${data?.key()}], value = [${data?.value()}]] from partition [${data?.partition()}] with offset [${data?.offset()}]: $exception")
    }
}

```
------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 3: YOU SHOULD NEVER UNDERESTIMATE THE PREDICTABILITY OF PRODUCER.</h6>

<b>Producer Settings</b>

|Setting|Values|Useful Links|
|:------|:-----|:-----------|
|retries|'[1,...,2147483647]'|[retries](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html)|
|transaction-id-prefix|'name by convention'|[transaction-id-prefix](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html)|
|enable.idempotence|'true/false'|[idempotence](https://www.conduktor.io/kafka/idempotent-kafka-producer)|
|compression.type|'none/gzip/zl4/snappy/zstd'|[compression](https://www.conduktor.io/kafka/kafka-message-compression)|
|max.in.flight.requests.per.connection|'[1,...,2147483647]'|[in-flight-requests-per-connection](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html)|

[Common Producers Settings](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html)

Sample of Spring Configuration:
``` yaml
spring:
  kafka:
    producer:
      bootstrap-servers: http://localhost:9092, http://localhost:9093, http://localhost:9094
      key-serializer: org.apache.kafka.common.serialization.LongSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      acks: all
      retries: 5
      transaction-id-prefix: kafka-producer
      properties:
        enable.idempotence: true
        compression.type: none
        max.in.flight.requests.per.connection: 1
        auto.create.topics.enable: false
    template:
      default-topic: notification-event
    properties:
      min.insync.replicas: 2
    listener:
      ack-mode: manual
      missing-topics-fatal: false

```

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 4: SO WHAT DO YOU DO? YOU GO TO SEE THE MAN THAT KNOWS ABOUT THESE SORT OF ZOOKEEPER-FREE.</h6>

For sure, if you are in touch with Kafka development you've already heard about 'Zookeeper-free'.

So, I would like to share with you a nice [article](https://www.confluent.io/blog/kafka-without-zookeeper-a-sneak-peek/) about it.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 5: ANYTHING TO DECLARE? YEAH. DON'T GO TO SYNCHRONOUS COMMUNICATION.</h6>

Moreover, there is a :gear:	[gist](https://gist.github.com/fragaLY/f0a9a235c3e924b90dc83de5ec964271) regarding this settings.

If you have any question, feel free to contact me direct in [linkedin](https://www.linkedin.com/in/vadzimkavalkou/).

Of course in the chapter naming is a typical quote and a joke. Remember, the asynchronous messaging is not a silver bullet, sometimes it doesn't fit your needs.

:gun: Sometimes you ought to follow the next quote:
>"HEAVY IS GOOD, HEAVY IS RELIABLE. IF IT DOESN'T WORK YOU CAN ALWAYS HIT THEM WITH IT."

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
