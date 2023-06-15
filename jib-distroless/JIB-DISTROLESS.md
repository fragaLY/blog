------------------------------------------------------------------------------------------------------------------------

![](./static/main.jpeg)

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 1: WHO ARE YOU?</h6>

:mechanical_arm: Hello world,

Today a bit information regarding the [Jib](https://github.com/GoogleContainerTools/jib) and [Distroless](https://github.com/GoogleContainerTools/distroless).

> Jib builds optimized Docker and OCI images for your Java applications without a Docker daemon - and without deep mastery of Docker best-practices. It is available as plugins for Maven and Gradle and as a Java library.

> "Distroless" images contain only your application and its runtime dependencies. They do not contain package managers, shells or any other programs you would expect to find in a standard Linux distribution.

I worked before a lot with Jib and Distroless and I've tried to improve my images size, delivery time, and decrease possible vulnerability issues.
In fact, it could help to save your money due to the network traffic and storage usage.

Let's check it.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 2: IF YOU WERE ANY THINNER, YOU WOULDN'T EXIST.</h6>

What we had before investigation:

|SIZE |VULNERABILITIES           |
|:----|:-------------------------|
|760MB|515(34 critical, 126 high)|

![](./static/dive_base.jpeg)

![](./static/grype_base.jpeg)

In scope of investigation, we added [distroless](https://console.cloud.google.com/gcr/images/distroless/global/java17-debian11) as a base image and started to use a Jib and we got:

|SIZE |VULNERABILITIES           |
|:----|:-------------------------|
|      |        |

![](./static/dive_distroless.jpeg)

![](./static/grype_distroless.jpeg)

I know what you are thinking. The size is still pretty big. Use Native, Vadzim. Migrate to Go, Rust...
This is kinda a miracle in the business world. I believe, you understand me and migration costs and risks.

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

Of course in the chapter naming is a typical quote and joke. Remember, the asynchronous messaging is not a silver bullet, sometimes it doesn't fit your needs.

:gun: Sometimes you ought to follow the next quote:
>"HEAVY IS GOOD, HEAVY IS RELIABLE. IF IT DOESN'T WORK YOU CAN ALWAYS HIT THEM WITH IT."

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
