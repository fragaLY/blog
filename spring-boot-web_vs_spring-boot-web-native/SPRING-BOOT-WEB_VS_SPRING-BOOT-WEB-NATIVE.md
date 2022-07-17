------------------------------------------------------------------------------------------------------------------------

![](./static/main.jpeg)

<h6>CHAPTER 1: YOU CAN'T BE DOING THIS, VADZIM. YOU CAN'T KEEP TRYING TO TUNE PERFORMANCE ALL THE TIME.</h6>

Hello, world,

Today I would like to compare the performance of [Spring Boot Web](https://spring.io/projects/spring-boot) and [Spring Boot Web Native](https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/).

I read tons of articles regarding them and found some contradictory results. So, I decided to check it on my own.

I hope you, my reader, are familiar with Spring Boot but I will provide you a quote from official documentation on what is it:
> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".
We take an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss. Most Spring Boot applications need minimal Spring configuration.
If you’re looking for information about a specific version, or instructions about how to upgrade from an earlier release, check out the project release notes section on our wiki.
Features: Create stand-alone Spring applications; Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files);
Provide opinionated 'starter' dependencies to simplify your build configuration; Automatically configure Spring and 3rd party libraries whenever possible;
Provide production-ready features such as metrics, health checks, and externalized configuration; Absolutely no code generation and no requirement for XML configuration.

Now, let's check what is Spring Native:
> Spring Native provides support for compiling Spring applications to native executables using the GraalVM native-image compiler.
Compared to the Java Virtual Machine, native images can enable cheaper and more sustainable hosting for many types of workloads. These include microservices, function workloads, well suited to containers, and Kubernetes
Using native image provides key advantages, such as instant startup, instant peak performance, and reduced memory consumption.
There are also some drawbacks and trade-offs that the GraalVM native project expect to improve on over time. Building a native image is a heavy process that is slower than a regular application. A native image has fewer runtime optimizations after warmup. Finally, it is less mature than the JVM with some different behaviors.
The key differences between a regular JVM and this native image platform are:
A static analysis of your application from the main entry point is performed at build time; The unused parts are removed at build time;
Configuration is required for reflection, resources, and dynamic proxies; Classpath is fixed at build time;
No class lazy loading: everything shipped in the executables will be loaded in memory on startup; Some code will run at build time;
There are some limitations around some aspects of Java applications that are not fully supported;
The goal of this project is to incubate the support for Spring Native, an alternative to Spring JVM, and provide a native deployment option designed to be packaged in lightweight containers. In practice, the target is to support your Spring applications, almost unmodified, on this new platform.

I will not dive deeper into AOT and [GraalVM](https://www.graalvm.org/). This is out of the scope of this article.

So, what I am going to check? I would like to check the performance of applications using [Tomcat](https://tomcat.apache.org/), [Jetty](https://www.eclipse.org/jetty/), [Undertow](https://undertow.io/), [JIB](https://github.com/GoogleContainerTools/jib)-built image, and Native Application using both [native build tools](https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/#getting-started-native-build-tools) and [build packs](https://docs.spring.io/spring-native/docs/current/reference/htmlsingle/#getting-started-buildpacks).

Let's move forward.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 2: MAMA SAYS STUPID IS AS STUPID DOES.</h6>

I've created an application that handles the transportation service business.

Let's assume that the average user has the biggest interest to book the transfer from 8:00 till 00:00.
As a result, we will have 16 per day and 112 hours of load per week.

Furthermore, I would like to share with you some presents:
* I disabled all application levels caches to get clear performance metrics of the server as it is;
* The database and application are running in the same network to avoid net lags and penalties;
* The fetching strategy is `EAGER`. First, to do a higher load and non-optimized queries to the database. Second, not to spend time to set up entity graphs.
* The average time to boot up the service will not depend on the database structure due to the fact that I am not using any database migration tools such as [Flyway](https://flywaydb.org/) or [Liquibase](https://www.liquibase.org/);
* All database structures had been already prepared and filled with faked data;
* As a load testing tool I chose [Gatling](https://gatling.io/);
* The versions of all languages, frameworks, libraries, and databases are the latest LTS at the moment of testing.

So, let's create the database definition for my application. The script is pretty easy to read and understand if you are familiar with SQL.

![](./static/database-structure.png)

```sql
-- SCHEMA DEFINITION
CREATE SCHEMA IF NOT EXISTS A2B;
SET
SEARCH_PATH TO A2B;

-- USERS TABLE DEFINITION
CREATE TABLE users
(
    id         bigserial   NOT NULL,
    first_name varchar(50) NOT NULL,
    last_name  varchar(50) NOT NULL,
    email      varchar(50) NOT NULL,
    phone      varchar(50) NOT NULL,

    CONSTRAINT "pk_domain_user.user" PRIMARY KEY (id)
);

CREATE UNIQUE INDEX "domain_user.email_index" ON users (email);
CREATE UNIQUE INDEX "domain_user.phone_index" ON users (email);

-- COUNTRIES TABLE DEFINITION
CREATE TABLE countries
(
    id   bigserial   NOT NULL,
    name varchar(50) NOT NULL,
    code varchar(10) NOT NULL,

    CONSTRAINT "pk_domain_country.country" PRIMARY KEY (id)
);

-- CITIES TABLE DEFINITION
CREATE TABLE cities
(
    id         bigserial    NOT NULL,
    country_id bigint       NOT NULL,
    name       varchar(100) NOT NULL,
    code       varchar(10)  NOT NULL,

    CONSTRAINT "pk_domain_city.city" PRIMARY KEY (id),
    FOREIGN KEY (country_id) REFERENCES countries (id) ON DELETE CASCADE
);

-- LOCATIONS TABLE DEFINITION
CREATE TABLE locations
(
    id       bigserial NOT NULL,
    city_id  bigint    NOT NULL,
    location jsonb     NOT NULL,

    CONSTRAINT "pk_domain_location.location" PRIMARY KEY (id),
    FOREIGN KEY (city_id) REFERENCES cities (id) ON DELETE CASCADE
);

-- TRANSFERS TABLE DEFINITION
CREATE TABLE transfers
(
    id          bigserial NOT NULL,
    origin      bigint    NOT NULL,
    destination bigint    NOT NULL,
    capacity    smallint  NOT NULL,
    date        date      NOT NULL,
    duration    tsrange   NOT NULL,
    price       money     NOT NULL,
    description text      NOT NULL,

    CONSTRAINT "pk_domain_transfer.transfer" PRIMARY KEY (id),
    FOREIGN KEY (origin) REFERENCES locations (id) ON DELETE CASCADE,
    FOREIGN KEY (destination) REFERENCES locations (id) ON DELETE CASCADE
);

-- USERS_TRANSFERS TABLE DEFINITION
CREATE TABLE users_transfers
(
    user_id     bigint      NOT NULL,
    transfer_id bigint      NOT NULL,
    state       varchar(10) NOT NULL,
    description text        NOT NULL,

    CONSTRAINT "pk_domain_users_transfers.users_transfers" PRIMARY KEY (user_id, transfer_id),
    FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE CASCADE,
    FOREIGN KEY (transfer_id) REFERENCES transfers (id) ON DELETE CASCADE
);
```

To kick off the creation I've prepared the docker-compose file:

```docker
version: '3.8'
services:

  postgres:
    image: postgres:14.2-alpine
    container_name: postgres
    hostname: postgres
    restart: on-failure
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=a2b
    ports:
      - "5433:5432"
    volumes:
      - ./database_definition.sql:/docker-entrypoint-initdb.d/database_definition.sql

  pgadmin:
    image: dpage/pgadmin4:6.9
    container_name: pgadmin4
    restart: on-failure
    environment:
      PGADMIN_DEFAULT_EMAIL: user@user.com
      PGADMIN_DEFAULT_PASSWORD: "password"
    ports:
      - "5050:80"
    depends_on:
      - postgres

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - "./prometheus.yml:/etc/prometheus/prometheus.yml"
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"

```

To connect via pgadmin follow [pgadmin](localhost:5050) enter "user@user.com" and "password".
Create server and attach to host `docker inspect $(docker ps -aqf "name=postgres") | grep IPAddress`, port "5433", db "postgres" with "user" and "password" credentials.

Now, let's imagine the number of entities in use. I decided to launch the application for one country with six cities. I love my country, it's about Belarus.
So, in every city, we will have 20 points to be able to arrive or depart. I assume that every point has about 100 transfers.
The target audience is 200_000 users. And every one of them has at least 2 transfers.

|       UNIT      | AMOUNT |
|:----------------|:-------|
| Countries       | 1      |
| Cities          | 6      |
| Locations       | 120    |
| Transfers       | 12000  |
| Users           | 200000 |
| Users_Transfers | 500000 |

All this files you could find [there in my repository](https://github.com/fragaLY/performance-researches/tree/master/env/database).

For a generation of the dataset, I've created the other module [datagen](https://github.com/fragaLY/performance-researches/tree/master/datagen).
I will not dive deeper into data generation process. You could easily check out how it works in sources. After that, I got DB size, and DB dump, and created a separate docker image with the predefined dataset.
This docker image is located in every tested module directory. I've already described how to create a custom db image in my [previous post](../postgres-custom-image/POSTGRES-CUSTOM-IMAGE.md).

The DB metrics:

* Database size: 153 MB
* Database dump size: 130.8 MB
* Docker image with predefined data: 160.01 MB

OK, now we have the dataset. The next step is the creation of test scenarios and preparing everything for running a performance test.

------------------------------------------------------------------------------------------------------------------------
<h6>CHAPTER 3: RUN, GATLING, RUN!</h6>

PERFORMANCE DESCRIPTION.
TEST SCENARIOS DESCRIPTION.
GATLING TESTS DESCRIPTION.

------------------------------------------------------------------------------------------------------------------------
<h6>CHAPTER 4: MY MAMA ALWAYS SAID LIFE WAS LIKE A BOX OF BEAN DEFINITIONS. YOU NEVER KNOW WHAT YOU'RE GONNA GET.</h6>

SPRING BOOT WEB SETTINGS AND CONFIGURATIONS.
PERFORMANCE TESTS RESULTS.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 5: MY MAMA ALWAYS SAID YOU'VE GOT TO PUT THE PAST BEHIND YOU BEFORE YOU CAN MOVE ON.</h6>

SPRING BOOT WEB NATIVE SETTINGS AND CONFIGURATIONS.
PERFORMANCE TESTS RESULTS.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 6: FROM THAT DAY ON, WE WAS ALWAYS TOGETHER. SPRING AND NATIVE WAS LIKE PEAS AND CARROTS.</h6>

COMPARING THE PERFORMANCE OF BOTH SOLUTIONS.

WHEN TO USE THE FIRST ONE.
WHEN TO USE THE SECOND ONE.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 7: MAMA ALWAYS SAID YOU CAN TELL A LOT ABOUT APPLICATIONS BY THEIR PERFORMANCE. WHERE THEY'VE BEEN, WHERE THEY'RE GOING.</h6>

SUM UP.

------------------------------------------------------------------------------------------------------------------------

<h6>BONUS: LIEUTENANT DAN GOT ME INVESTED IN SOME KIND OF SPRING COMPANY.</h6>

NEXT PRESENTATIONS ADD.
HAVE A NICE DAY.

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
