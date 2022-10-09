------------------------------------------------------------------------------------------------------------------------

![](./static/main.jpeg)

<h6>CHAPTER 1: THERE'S ONLY ONE RULE IN THE JUNGLE: WHEN THE LION'S HUNGRY, HE EATS!</h6>

Hello, world,

The next step in my journey throw the jungles of frameworks is [Quarkus](https://quarkus.io/).
SUPERSONIC? SUBATOMIC?

Ok, Ok. Let's test it.

Before all, I would like to tell you that I would like to test the reactive approach
due to the fact, that in previous research this solution brings us a pretty good system speed up.

So, I've took a [Quarkus Reactive + Reactive Postgres Client ](https://quarkus.io/guides/reactive-sql-clients) with it's [architecture](https://quarkus.io/guides/quarkus-reactive-architecture).

I hope you are familiar with my previous performance results regarding the Spring Web (as Native), and Spring Reactive (as Native).
If not, please [take a look on it.](../spring-boot-reactive_vs_spring-boot-reactive-native/SPRING-BOOT-REACTIVE_VS_SPRING-BOOT-REACTIVE-NATIVE.md)

This article is not about Quarkus internal architecture and design, its paradigms, and the solutions that Quarkus Team brings to life.
This article is about performance.
I will not dive deeper into the Quarkus Reactive stack and the business description of my application you could always read it on your own in my previous research and in the documentation.

And today I will check the performance of a native executable (including in the docker solution) and a default one (including inD solution as well).

The lion is hungry. And I am going to eat.

------------------------------------------------------------------------------------------------------------------------
<h6>CHAPTER 2: BRILLIANCE SHOULD BE ACKNOWLEDGED.</h6>

So, we are going to create our application based on Quarkus.
Hopefully, the developers who are familiar with Spring Framework could easily migrate to Quarkus.
Quarkus brings a number of pros and cons. And it depends on you what is more appropriate for your business.
I will just provide you the link to the [sources](https://github.com/fragaLY/performance-researches/tree/master/quarkus-reactive).

The languages, frameworks, and tools I used.

|JDK|GC|Gradle|Quarkus     |
|:--|:-|:-----|:-----------|
|17 |G1|7.5.1 |2.13.1.Final|

I will highlight some of the configurations here.

### Gradle Build Script

```groovy

plugins {
    id 'java'
    id 'io.quarkus' version '2.13.1.Final'
}

repositories {
    mavenCentral()
    mavenLocal()
}

dependencies {
    //region quarkus
    implementation enforcedPlatform("${quarkusPlatformGroupId}:${quarkusPlatformArtifactId}:${quarkusPlatformVersion}")
    implementation("io.quarkus:quarkus-resteasy-reactive-jackson")
    implementation("io.quarkus:quarkus-reactive-pg-client")
    implementation("io.quarkus:quarkus-config-yaml")
    implementation("io.quarkus:quarkus-logging-json")
    implementation("io.quarkus:quarkus-arc")
    //endregion
    //region lombok
    annotationProcessor("org.projectlombok:lombok:${lombokVersion}")
    implementation("org.projectlombok:lombok:${lombokVersion}")
    //endregion
}

group 'by.vk.quarkusweb'
version '1.0-SNAPSHOT'

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

compileJava {
    options.encoding = 'UTF-8'
    options.compilerArgs << '-parameters'
}

```

And application.yml, for sure.

```yaml

quarkus:
  banner:
    enabled: false
  container-image:
    build: true # set false for non image building procedure
  datasource:
    db-kind: postgresql
    username: postgres
    password: postgres
    reactive:
      url: postgresql://localhost:5433/a2b?currentSchema=a2b

```

As I already mentioned, I decided to check all possible types of launching the application, such as jar, jar in docker, native executable, and in docker native solutions as well.

In Quarkus you have [several ways](https://quarkus.io/guides/container-image) of building a docker image:
- JIB (both for UBI and Distroless images);
- Docker;
- S2I;
- Buildpack.

And a bit more of building the native executable, especially as a part of the docker image.
And I am going to check all of these cases.

So, in total I will have results for:
  - Typical fast and uber jars and 4 different images that had been built with JIB, Docker, S2I, and Buildpack;
  - Fully static native executable (non-ready for production),
  Linux executable without GraalVM,
  using the container-image extensions,
  manually using the micro base image,
  manually using the minimal base image,
  using a multi-stage Docker build,
  using a Distroless base image,
  using a scratch base image,
  using buildpack;
  - For the best-rated result of the native executable I will provide the results of compression using [UPX](https://quarkus.io/guides/upx) both for max and min compression.

Hm, 17 approaches for only one framework. Nice one.

Just do it.

------------------------------------------------------------------------------------------------------------------------
<h6>CHAPTER 3: A GENTLEMAN'S QUOTE IS A GENTLEMAN'S WORD.</h6>

There are results of non-native solutions.

* FAST JAR

Global information:

![](./static/reactive/jar/fast/global.png)

Requests:

![](./static/reactive/jar/fast/requests.png)

Requests per second:

![](./static/reactive/jar/fast/requests_per_second.png)

Responses per second:

![](./static/reactive/jar/fast/responses_per_second.png)

Response time for first minute:

![](./static/reactive/jar/fast/response_time_1.png)

Response time for all time:

![](./static/reactive/jar/fast/response_time_all.png)

You could download the [Jar Performance Tests Results](./static/reactive/jar/fast/reactive-fast-jar.zip) and check it on your own.

* UBER JAR

Global information:

![](./static/reactive/jar/uber/global.png)

Requests:

![](./static/reactive/jar/uber/requests.png)

Requests per second:

![](./static/reactive/jar/uber/requests_per_second.png)

Responses per second:

![](./static/reactive/jar/uber/responses_per_second.png)

Response time for first minute:

![](./static/reactive/jar/uber/response_time_1.png)

Response time for all time:

![](./static/reactive/jar/uber/response_time_all.png)

You could download the [Jar Performance Tests Results](./static/reactive/jar/uber/reactive-uber-jar.zip) and check it on your own.

* JIB with default UBI base image

Global information:

![](./static/reactive/jib/ubi/global.png)

Requests:

![](./static/reactive/jib/ubi/requests.png)

Requests per second:

![](./static/reactive/jib/ubi/requests_per_second.png)

Responses per second:

![](./static/reactive/jib/ubi/responses_per_second.png)

Response time for first minute:

![](./static/reactive/jib/ubi/response_time_1.png)

Response time for all time:

![](./static/reactive/jib/ubi/response_time_all.png)

Docker image investigation:

![](./static/reactive/jib/ubi/dive_docker_image.png)

You could download the [Jar Performance Tests Results](./static/reactive/jib/ubi/reactive-jib-ubi.zip) and check it on your own.

* JIB with Distroless base image

Global information:

![](./static/reactive/jib/distroless/global.png)

Requests:

![](./static/reactive/jib/distroless/requests.png)

Requests per second:

![](./static/reactive/jib/distroless/requests_per_second.png)

Responses per second:

![](./static/reactive/jib/distroless/responses_per_second.png)

Response time for first minute:

![](./static/reactive/jib/distroless/response_time_1.png)

Response time for all time:

![](./static/reactive/jib/distroless/response_time_all.png)

Docker image investigation:

![](./static/reactive/jib/distroless/dive_docker_image.png)

You could download the [Jar Performance Tests Results](./static/reactive/jib/distroless/reactive-jib-distroless.zip) and check it on your own.

* Docker

Global information:

![](./static/reactive/docker/global.png)

Requests:

![](./static/reactive/docker/requests.png)

Requests per second:

![](./static/reactive/docker/requests_per_second.png)

Responses per second:

![](./static/reactive/docker/responses_per_second.png)

Response time for first minute:

![](./static/reactive/docker/response_time_1.png)

Response time for all time:

![](./static/reactive/docker/response_time_all.png)

Docker image investigation:

![](./static/reactive/docker/dive_docker_image.png)

You could download the [Jar Performance Tests Results](./static/reactive/docker/reactive-docker.zip) and check it on your own.

* S2I

Global information:

![](./static/reactive/S2I/global.png)

Requests:

![](./static/reactive/S2I/requests.png)

Requests per second:

![](./static/reactive/S2I/requests_per_second.png)

Responses per second:

![](./static/reactive/S2I/responses_per_second.png)

Response time for first minute:

![](./static/reactive/S2I/response_time_1.png)

Response time for all time:

![](./static/reactive/S2I/response_time_all.png)

Docker image investigation:

![](./static/reactive/S2I/dive_docker_image.png)

You could download the [Jar Performance Tests Results](./static/reactive/S2I/reactive-s2i.zip) and check it on your own.

* Buildpack

Global information:

![](./static/reactive/buildpack/global.png)

Requests:

![](./static/reactive/buildpack/requests.png)

Requests per second:

![](./static/reactive/buildpack/requests_per_second.png)

Responses per second:

![](./static/reactive/buildpack/responses_per_second.png)

Response time for first minute:

![](./static/reactive/buildpack/response_time_1.png)

Response time for all time:

![](./static/reactive/buildpack/response_time_all.png)

Docker image investigation:

![](./static/reactive/buildpack/dive_docker_image.png)

You could download the [Jar Performance Tests Results](./static/reactive/buildpack/reactive-s2i.zip) and check it on your own.

Let's gather all the information:

|TYPE         |BUILD TIME (s)|ARTIFACT SIZE (MB)|BOOT UP (s)|ACTIVE USERS|RPS    |RESPONSE TIME (95th pct) (ms)|SATURATION POINT|JVM HEAP (MB)|JVM NON-HEAP (MB)|JVM CPU (%)|THREADS (MAX)|POSTGRES CPU (%)|
|:------------|:-------------|:-----------------|:----------|:-----------|:------|:----------------------------|:---------------|:------------|:----------------|:----------|:------------|:---------------|
|FAST JAR     |4             |N/A               |0,987      |10246       |755.434|13686                        |1971            |999          |55               |9          |25           |99              |
|UBER JAR     |8             |17,7              |1,884      |10258       |753.933|14111                        |2149            |934          |55               |5          |23           |99              |
|JIB/ubi8     |16            |384               |1.151      |10244       |593.275|20170                        |1305            |999          |55               |8          |26           |70              |
|JIB/distroless|14           |249               |1,088      |10202       |428.563|33060                        |1339            |915          |55               |15         |26           |93              |
|DOCKER       |              |                  |           |            |       |                             |                |             |                 |           |             |                |
|S2I          |              |                  |           |            |       |                             |                |             |                 |           |             |                |
|BUILDPACK    |              |                  |           |            |       |                             |                |             |                 |           |             |                |

Move on.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 4: THE YOUNG SUCCEED THE OLD!</h6>

Now it's a time to compare previous solutions with native ones.

TBD

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 5: IF YOU WISH TO BE THE KING OF THE JUNGLE, IT'S NOT ENOUGH TO ACT LIKE A KING. YOU MUST BE THE KING.</h6>

Let's compare all the results including the Spring Web, Spring Reactive and their native solutions as well.

|FRAMEWORK|APPLICATION TYPE|BUILD TYPE        |BUILD TIME (s)|ARTIFACT SIZE (MB)|BOOT UP (s)|ACTIVE USERS|TOTAL REQUESTS|OK     |KO(%)|RPS    |RESPONSE TIME (95th pct) (ms)|SATURATION POINT|RAM (MB)|CPU (%) |THREADS (MAX)|POSTGRES CPU (%)|
|:--------|:---------------|:-----------------|:-------------|:-----------------|:----------|:-----------|:-------------|:------|:----|:------|:----------------------------|:---------------|:-------|:-------|:------------|:---------------|
|SPRING   |WEB             |BUILD PACK        |751           |144,79            |1,585      |10201       |453012        |339759 |25   |374.566|47831                        |584             |310     |12,5    |64           |99              |
|         |                |NATIVE BUILD TOOLS|210           |116,20            |0,310      |8759        |480763        |342782 |29   |414.785|32175                        |1829            |263     |8       |52           |99              |
|         |                |UNDERTOW          |5             |49,70             |3,59       |10311       |523756        |396071 |24   |381.127|50977                        |1611            |658     |11      |33           |99              |
|         |                |UNDERTOW IN DOCKER|46            |280               |5,20       |10264       |430673        |289692 |33   |448.682|29998                        |916             |840     |15      |32           |99              |
|         |REACTIVE        |BUILD PACK        |1243          |98,5              |0,103      |10268       |691487        |573983 |17   |615.75	|17891                        |1904            |685     |30      |14           |70              |
|         |                |NATIVE BUILD TOOLS|187           |71,7              |0,107      |10224       |1013549       |915094 |10   |934.147|12591                        |3038            |634     |32      |23           |70              |
|         |                |JAR               |3,1           |40,6              |2,55       |10326       |1168782       |1079847|8    |1091,3 |10406                        |4391            |1823    |8       |31           |70              |
|         |                |JAR IN DOCKER     |39            |271               |3,95       |10258       |699180        |581761 |17   |631.599|18955                        |2250            |883     |29      |31           |70              |
|         |                |                  |              |                  |           |            |              |       |     |       |                             |                |        |        |             |                |
|QUARKUS  |REACTIVE        |FAST JAR          |4             |N/A               |0,987      |10246       |828711        |718773 |13   |755.434|13686                        |1971            |1054    |9       |25           |99              |
|         |                |UBER JAR          |8             |17,7              |1,884      |10258       |826311        |716252 |13   |753.933|14111                        |2149            |989     |5       |23           |99              |
|         |                |JIB WITH UBI      |16            |384               |1.151      |10244       |661502        |120360 |18   |593.275|20170                        |1305            |1054    |8       |26           |70              |
|         |                |JIB WITH DISTROLESS|14           |249               |1.088      |10202       |473991        |343384 |28   |428.563|33060                        |1339            |970     |8       |26           |93              |

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 6: NOW STARTS THE ALPHA DANCE.</h6>

TBD

------------------------------------------------------------------------------------------------------------------------

<h6>BONUS: I'LL HAVE A PINT AND A PICKLED EGG.</h6>

This article is the 3rd in my performance journey.

Next, I will bring you details regarding the [Micronaut](https://micronaut.io/), [Vert.x](https://vertx.io/), [Helidon](https://helidon.io/), and [Ktor](https://ktor.io/).

So, will be in touch.

HAVE A NICE DAY.

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
