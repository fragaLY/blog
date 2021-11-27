------------------------------------------------------------------------------------------------------------------------

_Please be assured that you are familiar with the [license](https://www.gnu.org/licenses/agpl-3.0.en.html)._

------------------------------------------------------------------------------------------------------------------------

![](./static/main.jpeg)

<h6>CHAPTER 1: HEY, NICE MARMOT.</h6> 
Firstly, I would like to bring you some brief information about the HAPI FHIR.

<a href="https://hapifhir.io/">HAPI FHIR</a> is a complete implementation of the <a href="http://hl7.org/fhir/">HL7 FHIR</a> standard for healthcare interoperability in Java.
They are an open community developing software licensed under the business-friendly Apache Software License 2.0.
HAPI FHIR is a product of Smile CDR.

What is <a href="http://hl7.org/fhir/">HL7 FHIR</a>?

FHIR – Fast Healthcare Interoperability Resources – is a next generation standards framework created by HL7. FHIR combines the best features of HL7's v2 , HL7 v3  and CDA  product lines while leveraging the latest web standards and applying a tight focus on implementability.

FHIR solutions are built from a set of modular components called "Resources". These resources can easily be assembled into working systems that solve real world clinical and administrative problems at a fraction of the price of existing alternatives. FHIR is suitable for use in a wide variety of contexts – mobile phone apps, cloud communications, EHR-based data sharing, server communication in large institutional healthcare providers, and much more.

Resource example (it could be illustrated and stored as JSON as well)
![](./static/resource-example.png)

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 2: THE HAPI-FHIR-JPASERVER-STARTER ABIDES.</h6>

The HAPI FHIR team provides a set of [open source](https://github.com/hapifhir) solutions for HL7 FHIR.
One of them is [hapi-fhir-jpaserver-starter](https://github.com/hapifhir/hapi-fhir-jpaserver-starter).

To be honest, the HL7 and HAPI FHIR is not the easiest architecture and solution to understand.
You will need some patience, concentration, and time to feel strong in the healthcare domain.

So, what is the _hapi-fhir-jpaserver-starter_ (HFJS in future)?

This project is a complete starter project you can use to deploy a FHIR server using HAPI FHIR JPA.
This is a typical solution based on [Spring Boot](https://spring.io/projects/spring-boot).

It has one great killer feature: during the start of the application all the schemas and database structures are created as it should be done for HL7.
You can bring your custom changes easily using autoconfiguration, tune your database settings, such as indexes, resource validations both for requests and responses, types of supported resources.

To check up on what you can bring useful in your project, please visit the [demo server](http://hapi.fhir.org/baseR4/swagger-ui/).
It's absolutely not necessary to use all the resources, you can move to microservice architecture and split logic for different domain-based areas.

This project is a fully contained FHIR server, supporting all standard operations (read/create/delete/etc).
It bundles an embedded instance of the H2 Java Database so that the server can run without depending on any external database, but it can also be configured to use an installation of Oracle, Postgres, etc.

Long story short, this is the representation of [HAPI JPA Server architecture](https://hapifhir.io/hapi-fhir/docs/server_jpa/architecture.html):
![](static/1step/g1/tomcat/50k-tomcat-g1-hapi-fhir-jpa-architecture.png)

To dive deeper in the topic, you can follow the next links:
- [Database Schema](https://hapifhir.io/hapi-fhir/docs/server_jpa/schema.html)
- [Configuration](https://hapifhir.io/hapi-fhir/docs/server_jpa/configuration.html)

In spite of all the benefits that HFJS brings to us, there are a lot of trade-offs.
And in a world where every millisecond may costs you a huge profit, the performance, boot-up time, and resources consumption have a great impact.

------------------------------------------------------------------------------------------------------------------------
<h6>CHAPTER 3: IT REALLY TIED THE ROOM TOGETHER.</h6>
After a few months of investigations and data preparations for HFJS, we get the vision of possible resources consumption.
So, we stopped on the next setup, in my honest opinion, this is the minimum requirement for HAPI FHIR.
`OS Linux (amd64) OS ImageOracle Linux Server 8.2 Cores 4, RAM 16000Mi`. 

For compile and build processes we are using `JDK 11, Gradle 7.3.` We've took a useful for us parts of HFJSS, such as `ca.uhn.hapi.fhir:hapi-fhir-base, ca.uhn.hapi.fhir:hapi-fhir-jpaserver-base, ca.uhn.hapi.fhir:hapi-fhir-validation, ca.uhn.hapi.fhir:hapi-fhir-structures-r4, ca.uhn.hapi.fhir:hapi-fhir-validation-resources-r4`.
And we build our own theme park, with blackjack and hO_okers.
The versions of used libs:
```properties
hapi_fhir_version=5.5.2
logback_version=0.1.5
hikari_version=5.0.0
micrometer_version=1.7.5
spring_version=2.5.6
spring_cloud_version=2020.0.4
spring_dependency_management_version=1.0.11.RELEASE
```
The basic configuration for HFJSS had been tuned a bit, mostly tuning affected pools and workers, some searching, validation, and reindexing settings.
If you are not a lot familiar with hikari pool tuning, there is a [great article](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing) about pool sizing. Long story short,
The `application.yaml` file with basic settings that could be useful for you has the next structure.
It will be tuned soon.

``` yaml
server:
  port: 8080
  shutdown: graceful
  tomcat:
    connection-timeout: 20s
    threads:
      max: 24
      min-spare: 8
  error:
    whitelabel:
      enabled: false

spring:
  application:
    name: hapi-fhir-service
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
    driverClassName: org.postgresql.Driver
    hikari:
      minimumIdle: 8
      maximumPoolSize: 32
      connection-timeout: 35000
      pool-name: "hfs-hikari-pool"
      idle-timeout: 10000
      max-lifetime: 30000
      auto-commit: true
  data:
    jpa:
      repositories:
        bootstrap-mode: deferred
  jpa:
    open-in-view: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: false
        show_sql: false
        hbm2ddl.auto: update
        cache:
          use_query_cache: false
          use_second_level_cache: false
          use_structured_entries: false
          use_minimal_puts: false
        search:
          enabled: false
          backend:
            type: lucene
            analysis:
              configurer: ca.uhn.fhir.jpa.search.HapiLuceneAnalysisConfigurer
            directory:
              type: local-filesystem
              root: target/lucenefiles
            lucene_version: lucene_current
  batch:
    job:
      enabled: false
  zipkin:
    sender:
      type: KAFKA
  kafka:
    bootstrap-servers: ${KAFKA_BOOTSTRAP_SERVERS}
  thymeleaf:
    enabled: false

hapi:
  fhir:
    base_path: "/*"
    defer_indexing_for_codesystems_of_size: 101
    supported_resource_types:
      - Patient
      - Observation
      - Organization
      - Encounter
      - Condition
      - Composition
      - EpisodeOfCare
      - Medication
      - MedicationAdministration
      - MedicationRequest
      - Immunization
      - DiagnosticReport
      - Procedure
      - Bundle
      - Goal
      - CarePlan
      - Practitioner
      - Claim
      - ExplanationOfBenefit
      - ValueSet
      - CodeSystem
      - StructureDefinition
      - ImagingStudy
      - List
    allow_external_references: true
    client_id_strategy: ANY
    allow_cascading_deletes: false
    allow_contains_searches: false
    allow_multiple_delete: false
    delete_enable: false
    allow_override_default_search_params: false
    allow_placeholder_references: true
    auto_create_placeholder_reference_targets: true
    default_encoding: JSON
    default_pretty_print: true
    default_page_size: 20
    empi_enabled: false
    enable_index_missing_fields: true
    enforce_referential_integrity_on_delete: true
    enforce_referential_integrity_on_write: true
    etag_support_enabled: true
    expunge_enabled: false
    fhir_version: R4
    fhir_path_interceptor_enabled: false
    filter_search_enabled: true
    graphql_enabled: false
    binary_storage_enabled: false
    last-n-enabled: false
    mark-resources-for-reindexing-upon-search-parameter-change: false
    max_binary_size: 104857600
    max_page_size: 200
    bulk_export_enabled: false
    retain_cached_searches_mins: 60
    reuse_cached_search_results_millis: 60000
    partitioning:
      enabled: false
      cross_partition_reference_mode: true
      multitenancy_enabled: true
      partitioning_include_in_search_hashes: true
    persistenceUnitName: "HAPI_PERSISTENCE_UNIT"
    cors:
      enabled: false
      allow_credentials: true
      allowed_origin:
        - '*'
    expunge-thread-count: 2
    reindex-thread-count: 2
    search-total-mode: ESTIMATED
    search-coord-core-pool-size: 20
    search-coord-max-pool-size: 100
    search-coord-queue-capacity: 20
    status-based-reindexing-disabled: true
    normalized_quantity_search_level: NORMALIZED_QUANTITY_SEARCH_SUPPORTED
    enable_index_contained_resource: false
    validation:
      enabled: true
      requests_enabled: true
      responses_enabled: false
    logger:
      enabled: true
      name: "fhirtest.access"
      error_format: "[HAPI FHIR ERROR] - ${requestVerb} ${requestUrl}"
      format: "Path[${servletPath}] Source[${requestHeader.x-forwarded-for}] Operation[${operationType} ${operationName} ${idOrResourceName}] UA[${requestHeader.user-agent}] Params[${requestParameters}] ResponseEncoding[${responseEncodingNoDefault}] Operation[${operationType} ${operationName} ${idOrResourceName}] UA[${requestHeader.user-agent}] Params[${requestParameters}] ResponseEncoding[${responseEncodingNoDefault}]"
      log_exceptions: true
    elasticsearch:
      enabled: false

```

One important thing is that we disabled hibernate caching to get clear performance metrics of the server as it is.

The average time to boot up the service depends on if it's the first start because the database structure could be created or not.
For sure, on production, we will use warm-up, and mostly we will have the situation without creating a database structure.
We will cover this and other topics in next chapters.

To my mind, I ought to share with you some use cases that we will cover with performance tests. You can find that most of all there are READ operations, and that is correct.
The user will do these operations most of all, and we should follow his/her behavior.

Outpatient practice (ambulatory):

| #  | Scenario step          | Assumed FHIR data operation |
| :---------------------: | :--------------------- | :-------------------------- |
| 1 | Login              |             |
| 2 | As a doctor, I want to open my patient’s medical card with general information page with diagnoses | Get  Patient resource, get several (10) Condition resources for this patient |
| 3 | As a doctor, I want to look through titles and dates of all available to me medical records (sections) for my patient for all the time | |
| 4 | As a doctor, I want to open some of them and read (e.g. General blood analysis 12.09.2003, Renal ultrasound 18.02.2014, General blood analysis 22.06.2020). Alternative flow: As a doctor, I want to scroll and look through all sections in my patient's card | Simultaneously get 10 DiagnosticReports by IDs, including all references recursively (_include:recurse=*) |
| 5 | As a doctor, I want to create a new medical record (section) of today’s visit and save it | Create DiagnosticReport with subject = this patient and 10 Observations in a result array |
| 6 | As a doctor, I want to close current patient and see a list of patients again as described in (2) | See (2) |
| 7 | Repeat (2) - (7) every 15-30 minutes for each patient | |
| 8 | Logout |  |

Inpatient practice (hospital):

| #  | Scenario step          | Assumed FHIR data operation |
| :---------------------: | :--------------------- | :-------------------------- |
| 1 | Login              |             |
| 2 | As a doctor, I want to see my list of patients, that are in my responsibility today: patients that are now in my hospital in my department and assigned to me | Find all Patients, that have EpisodeOfCare with period including [some date] AND [some] managingOrganization |
| 3 | As a doctor, I want to open my patient’s medical card with general information page | Get Patient resource, get several (5-20) resources for this patient, like AllergyIntolerance, Condition, RelatedPerson, Immunization |
| 4 | As a doctor, I want to look through titles and dates of all available to me medical records (sections) for my patient for all the time | Select all Compositions with subject = this patient (in average, 50, can be 5-200) |
| 5 | As a doctor, I want to open some of them and read (e.g. General blood analysis 12.09.2003, Renal ultrasound 18.02.2014, General blood analysis 22.12.2021) | For each of 3 opened medical records: Get Composition, in it’s section for all entryes select DiagnosticReport, in it’s result for all references select all (1-25) Observations |
| 6 | As a doctor, I want to look through dates and diagnoses of all hospitalizations of my patient | Find all EpisodeOfCare that was for this Patient |
| 7 | As a doctor, I want to open some hospitalizations (medical forms) and read all sections in them (e.g. Acute appendicitis 21.04.2017-25.04.2017, Pneumonia 13.09.2019-28.09.2019) | For each hospitalization (EpisodeOfCare) – find  Composition that in event in detail contains a reference to current EpisodeOfCare. For that Composition for all sections (5-20) in it’s entryes select all (1-25) Observations |
| 8 | As a doctor, I want to open a medical form for current hospitalization and read all sections | The same as (7) |
| 9 | As a doctor, I want to create a new section in this medical form with a daily medical record of my examination | Create Composition with subject = this patient and 10-30 Observations as entryes in it’s section |
| 10 | As a doctor, I want to close current patient and see a list of patients again as described in (2) | See (2) |
| 11 | Repeat (2) - (7) every 15-30 minutes for each patient |  |
| 12 | Logout |  |

------------------------------------------------------------------------------------------------------------------------
<h6>CHAPTER 4: WHERE'S MY RESOURCES, LEBOWSKI?</h6>

Moreover, it would be nice to provide you with some information regarding the datasets we've used.
The `Resources` we've used are the next (all the multipliers for resources that related to `Patient` were gained from researching the institutions and organization):

| Resource          | Multiplier | 1 step   | 2 step     | 3 step     | 4 step     |
| :---------------- | :----------| :--------| :--------  | :--------  | :--------  |
| Patient           | x1         | 50_000   | 100_000    | 250_000    | 500_000    |
| Observation       | x100       | 5_000_000| 10_000_000 | 25_000_000 | 50_000_000 |
| Diagnostic Report | x10        | 500_000  | 1_000_000  | 2_500_000  | 5_000_000  |
| Condition         | x10        | 500_000  | 1_000_000  | 2_500_000  | 5_000_000  |
| Procedure         | x3         | 150_000  | 300_000    | 750_000    | 1_500_000  |
| Episode Of Care   | x2         | 100_000  | 200_000    | 500_000    | 1_000_000  |
| Composition       | x2         | 100_000  | 200_000    | 500_000    | 1_000_000  |
| DB dump size (GiB)| `N/A`      | 6,5      | 15         | 24         | 38         |

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 5: WHERE'S MY PERFORMANCE, LEBOWSKI?</h6>

At the very beginning of this chapter, I would like to share with you some special secrets.
The hapi fhir server is a healthcare one and the business is not going to deal with a huge RPS.
But the metrics that I will share with you soon could help us to understand the possible scaling strategy better, follow fault-tolerance best practices, and provide the vision of resources consumption.
The last but not least, all these metrics can help us to predict some cost management.

Let's assume that the average institution has the biggest load from 8:00 till 17:00.
As a result, we will have ~40 business hours per week. The next metrics will be presented for a typical business day (transactions per working day, TPWD in future) and RPS as well.
And now, show me the numbers, dude!

<h2>Tomcat</h2>

|STEP|TRANSACTIONS PER WORKING DAY|TRANSACTIONS PER SECOND|RESPONSE TIMES OVER TIME|HITS PER SECOND|NODE CPU|NODE RAM (GiB)|JVM RATE (MAX ops/s)|JVM HEAP (GiB)|JVM NON-HEAP (Mib)|JVM CPU (%)|THREADS (MAX)                                   |HIKARI POOL (MAX)|HIKARI CONNECTIONS TIME (MAX ms)|POSTGRES STATEMENTS CALLS (ops/s)|
|:---|:---------------------------|:----------------------|:-----------------------|:--------------|:-------|:-------------|:-------------------|:-------------|:-----------------|:----------|:-----------------------------------------------|:----------------|:-------------------------------|:--------------------------------|
| 1  |460800                      | 16                    |~100                    | ~35           |~0.400  |~5            |~31                 |~3,20         |~275              |~20        |~94 (25 runnable, 40 waiting, 45 timed-waiting) |9 active, 15 idle|38 usage, 62 creation           |max: 475, avg: 185               |
| 2  |403200                      | 14                    |~100                    | ~35           |~0.360  |~3,36         |~31                 |~2,23         |~256              |~14        |~94 (22 runnable, 40 waiting, 45 timed-waiting) |5 active, 16 idle|35 usage, 45 creation           |max: 499, avg: 248               |
| 3  |604800                      | 21                    |~200                    | ~54           |~0.600  |~2,48         |~26                 |~2,14         |~257              |~14        |~104 (22 runnable, 54 waiting, 40 timed-waiting)|7 active, 19 idle|75 usage, 38 creation           |max: 499, avg: 248               |
| 4  |403200                      | 14                    |~80                     | ~40           |~0.360  |~6,65         |~31                 |~4,46         |~253              |~13        |~104 (23 runnable, 40 waiting, 41 timed-waiting)|7 active, 16 idle|78 usage, 36 creation           |max: 619, avg: 283               |

<h2>Jetty</h2>

|STEP|TRANSACTIONS PER WORKING DAY|TRANSACTIONS PER SECOND|RESPONSE TIMES OVER TIME|HITS PER SECOND|NODE CPU|NODE RAM (GiB)|JVM RATE (MAX ops/s)|JVM HEAP (GiB)|JVM NON-HEAP (Mib)|JVM CPU (%)|THREADS (MAX)                                   |HIKARI POOL (MAX) |HIKARI CONNECTIONS TIME (MAX ms)|POSTGRES STATEMENTS CALLS (ops/s)|
|:---|:---------------------------|:----------------------|:-----------------------|:--------------|:-------|:-------------|:-------------------|:-------------|:-----------------|:----------|:-----------------------------------------------|:-----------------|:-------------------------------|:--------------------------------|
| 1  |460800                      |16                     |~100                    |~40            |~0,300  |~6            |~32                 |~4,52         |~258              |~11        |~95 (23 runnable, 23 waiting, 54 timed-waiting) |8 active, 16 idle |22 usage, 53 creation           |max: 498, avg: 237               |
| 2  |460800                      |16                     |~80                     |~40            |~0,41   |~6,01         |~31                 |~4,01         |~256              |~17        |~98 (21 runnable, 23 waiting, 56 timed-waiting) |7 active, 19 idle |18 usage, 125 creation          |max: 486, avg: 310               |
| 3  |460800                      |16                     |~80                     |~40            |~0,400  |~5,69         |~32                 |~3,34         |~261              |~19        |~88 (24 runnable, 23 waiting, 48 timed-waiting) |6 active, 15 idle |18 usage, 125 creation          |max: 515, avg: 262               |
| 4  |460800                      |16                     |~200                    |~40            |~0,350  |~3,96         |~31                 |~3,09         |~287              |~13        |~98 (25 runnable, 24 waiting, 58 timed-waiting) |13 active, 16 idle|36 usage, 41 creation           |max: 508, avg: 235               |

<h2>Undertow</h2>

|STEP|TRANSACTIONS PER WORKING DAY|TRANSACTIONS PER SECOND|RESPONSE TIMES OVER TIME|HITS PER SECOND|NODE CPU|NODE RAM (GiB)|JVM RATE (MAX ops/s)|JVM HEAP (GiB)|JVM NON-HEAP (Mib)|JVM CPU (%)|THREADS (MAX)                                   |HIKARI POOL (MAX)|HIKARI CONNECTIONS TIME (MAX ms)|POSTGRES STATEMENTS CALLS (ops/s)|
|:---|:---------------------------|:----------------------|:-----------------------|:--------------|:-------|:-------------|:-------------------|:-------------|:-----------------|:----------|:-----------------------------------------------|:----------------|:-------------------------------|:--------------------------------|
| 1  |576000                      |18                     |~50                     |~43            |~0.250  |~3,5          |~31                 |~3,83         |~256              |~10        |~89 (27 runnable, 38 waiting, 42 timed-waiting) |7 active, 15 idle|13 usage,  20 creation          |max: 684, avg: 350               |
| 2  |489600                      |17                     |~70                     |~50            |~0,360  |~3,46         |~31                 |~1,88         |~256              |~12        |~86 (24 runnable, 26 waiting, 39 timed-waiting) |8 active, 19 idle|21 usage,  52 creation          |max: 502, avg: 250               |
| 3  |460800                      |16                     |~70                     |~45            |~0,250  |~6,51         |~31                 |~4,34         |~257              |~15        |~86 (27 runnable, 10 waiting, 17 timed-waiting) |6 active, 18 idle|103 usage, 55 creation          |max: 515, avg: 192               |
| 4  |460800                      |16                     |~80                     |~40            |~0,380  |~4,65         |~31                 |~2,57         |~251              |~28        |~87 (27 runnable, 26 waiting, 40 timed-waiting) |7 active, 16 idle|69 usage,  51 creation          |max: 523, avg: 307               |

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 6: WHERE'S MY BOOT-UP TIME, LEBOWSKI?</h6>

Now we will take a look at boot-up metrics on different embedded web servers.

Due to the fact, that we are using fewer libraries than the default HFJS starter it decreases the boot time up to 2 times.

| Tomcat   | Jetty    | Undertow |
| :------: | :------: | :------: |
| ~ 55,5s  |   ~46,5s |  ~ 45,5s |

Moreover, to decrease the boot-up time we've tried to add [spring-indexer](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-scanning-index).
Nevertheless, the spring-indexer is not compatible with hapi-fhir.
In spite of this, I would like to share the benefits it could bring to your projects.
While classpath scanning is very fast, it is possible to improve the startup performance of large applications by creating a static list of candidates at compilation time.
In this mode, all modules that are targets of component scanning must use this mechanism.
It creates `META-INF/spring.components` were all the beans described.

Actually, it increases the build time in few seconds, but every cloud has a silver lining.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 7: I CAN'T BE WORRYING ABOUT THAT SH1T. LIFE GOES ON, MAN.</h6>

After all the tests we've already described we are going to use Undertow as the embedded server.
Now we are going to compare the performance using JDK 11 and JDK 17.
The result are below.

|JDK  |TRANSACTIONS PER WORKING DAY|TRANSACTIONS PER SECOND|RESPONSE TIMES OVER TIME|HITS PER SECOND|NODE CPU|NODE RAM (GiB)|JVM RATE (MAX ops/s)|JVM HEAP (GiB)|JVM NON-HEAP (Mib)|JVM CPU (%)|THREADS (MAX)                                   |HIKARI POOL (MAX)|HIKARI CONNECTIONS TIME (MAX ms)|
|:----|:---------------------------|:----------------------|:-----------------------|:--------------|:-------|:-------------|:-------------------|:-------------|:-----------------|:----------|:-----------------------------------------------|:----------------|:-------------------------------|
|11   |460800                      |16                     |~80                     |~40            |~0,380  |~4,65         |~31                 |~2,57         |~251              |~28        |~87 (27 runnable, 26 waiting, 40 timed-waiting) |7 active, 16 idle|69 usage,  51 creation          |
|17   |576000                      |20                     |~100                    |~54            |~0,320  |~3,83         |~31                 |~2,24         |~242              |~17        |~86 (28 runnable, 26 waiting, 41 timed-waiting) |6 active, 16 idle|110 usage, 77 creation          |

After comparing the results of JDK 11 and JDK17 we are decided to choose the latest one LTS version (JDK 17).

Let's check the performance for different kinds of GC.

|GC TYPE   |TRANSACTIONS PER WORKING DAY|TRANSACTIONS PER SECOND|RESPONSE TIMES OVER TIME|HITS PER SECOND|NODE CPU|NODE RAM (GiB)|JVM RATE (MAX ops/s)|JVM HEAP (GiB)|JVM NON-HEAP (Mib)|JVM CPU (%)|THREADS (MAX)                                   |HIKARI POOL (MAX)|HIKARI CONNECTIONS TIME (MAX ms)|POSTGRES STATEMENTS CALLS (ops/s)|
|:---------|:---------------------------|:----------------------|:-----------------------|:--------------|:-------|:-------------|:-------------------|:-------------|:-----------------|:----------|:-----------------------------------------------|:----------------|:-------------------------------|:--------------------------------|
|G1        |576000                      |20                     |~100                    |~54            |~0,320  |~3,83         |~31                 |~2,24         |~242              |~17        |~86 (28 runnable, 26 waiting, 41 timed-waiting) |6 active, 16 idle|110 usage, 77 creation          |max: 488, avg: 243               |
|ZGC       |460800                      |16                     |~80                     |~40            |~0,350  |~3,09         |~32                 |~3,08         |~219              |~15        |~86 (25 runnable, 27 waiting, 43 timed-waiting) |5 active, 16 idle|80 usage,  53 creation          |max: 497, avg: 248               |

------------------------------------------------------------------------------------------------------------------------

<h6>BONUS: HA HEY, THIS IS A PRIVATE RESIDENCE MAN.</h6>

  The final fight is migration from Postgres 13.3 to [Postgres 14.1](https://www.postgresql.org/docs/14/release-14-1.html) due to having a lot of performance tunings, such as:
- Numerous performance improvements have been made for parallel queries, heavily-concurrent workloads, partitioned tables, logical replication, and vacuuming;
- B-tree index updates are managed more efficiently, reducing index bloat;
- VACUUM automatically becomes more aggressive, and skips inessential cleanup, if the database starts to approach a transaction ID wraparound condition;
- Ensure that parallel VACUUM doesn't miss any indexes;
- Fix REINDEX CONCURRENTLY to preserve operator class parameters that were attached to the target index;
- Avoid O(N^2) behavior in some list-manipulation operations;
- Add more defensive checks around B-tree posting list splits;
- Fix memory leak in HMAC hash calculations.

The test is going to be started on the best JVM setup that we've already found.
The dataset same for step 4. The size of DB is 443TB. Pretty big.

|POSTGRES  |TRANSACTIONS PER WORKING DAY|TRANSACTIONS PER SECOND|RESPONSE TIMES OVER TIME|HITS PER SECOND|NODE CPU|NODE RAM (GiB)|JVM RATE (MAX ops/s)|JVM HEAP (GiB)|JVM NON-HEAP (Mib)|JVM CPU (%)|THREADS (MAX)                                   |HIKARI POOL (MAX)|HIKARI CONNECTIONS TIME (MAX ms)|POSTGRES STATEMENTS CALLS (ops/s)|POSTGRES TOTAL DURATION OF QUERIES (ms)|
|:---------|:---------------------------|:----------------------|:-----------------------|:--------------|:-------|:-------------|:-------------------|:-------------|:-----------------|:----------|:-----------------------------------------------|:----------------|:-------------------------------|:--------------------------------|:--------------------------------------|
|13.3      |432000                      |15                     |~80                     |~36            |~0,400  |~8,5          |~32                 |~3,96         |~242              |~12        |~98 (21 runnable, 23 waiting, 57 timed-waiting) |6 active, 23 idle|31 usage, 165 creation          |max: 537, avg: 245               |max: 60,58, avg: 18,86                 |
|14.1      |432000                      |15                     |~80                     |~36            |~0,400  |~8,5          |~32                 |~3,96         |~242              |~12        |~98 (21 runnable, 23 waiting, 57 timed-waiting) |6 active, 23 idle|31 usage, 165 creation          |max: 537, avg: 245               |max: 60,58, avg: 18,86                 |

- Most comfortable setup for application

|JDK|GC|SERVER  |GRADLE|POSTGRES|POSRGRES VM RAM|POSRGRES VM CORE |POSTGRES SSD|JVM RAM|JVM CORE|
|:--|:-|:-------|:-----|:-------|:--------------|:----------------|:-----------|:------|:-------|
|17 |G1|Undertow|7.3   |?       |4 GiB          |2                |500 GB      |3 GiB  |2 core  |

``` yaml

server:
  port: 8080
  shutdown: graceful
  undertow:
    threads:
      worker: 24
      io: 3
  error:
    whitelabel:
      enabled: false

spring:
  application:
    name: misp-hapi-fhir-service
  main:
    banner-mode: off
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
    driverClassName: org.postgresql.Driver
    hikari:
      minimumIdle: 4
      maximumPoolSize: 10
      connection-timeout: 35000
      pool-name: "mhfs-hikari-pool"
      idle-timeout: 10000
      max-lifetime: 30000
      auto-commit: true
  data:
    jpa:
      repositories:
        bootstrap-mode: deferred
  jpa:
    open-in-view: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: false
        show_sql: false
        hbm2ddl.auto: update
        cache:
          use_query_cache: false
          use_second_level_cache: false
          use_structured_entries: false
          use_minimal_puts: false
        search:
          enabled: false
          backend:
            type: lucene
            analysis:
              configurer: ca.uhn.fhir.jpa.search.HapiLuceneAnalysisConfigurer
            directory:
              type: local-filesystem
              root: target/lucenefiles
            lucene_version: lucene_current
  batch:
    job:
      enabled: false
  zipkin:
    sender:
      type: KAFKA
  kafka:
    bootstrap-servers: ${KAFKA_BOOTSTRAP_SERVERS}
  thymeleaf:
    enabled: false

hapi:
  fhir:
    base_path: "/*"
    defer_indexing_for_codesystems_of_size: 101
    supported_resource_types:
      - Patient
      - Observation
      - Organization
      - Encounter
      - Condition
      - Composition
      - EpisodeOfCare
      - Medication
      - MedicationAdministration
      - MedicationRequest
      - Immunization
      - DiagnosticReport
      - Procedure
      - Bundle
      - Goal
      - CarePlan
      - Practitioner
      - Claim
      - ExplanationOfBenefit
      - ValueSet
      - CodeSystem
      - StructureDefinition
      - ImagingStudy
      - List
    allow_external_references: true
    client_id_strategy: ANY
    allow_cascading_deletes: false
    allow_contains_searches: false
    allow_multiple_delete: false
    delete_enable: false
    allow_override_default_search_params: false
    allow_placeholder_references: true
    auto_create_placeholder_reference_targets: true
    default_encoding: JSON
    default_pretty_print: true
    default_page_size: 20
    empi_enabled: false
    enable_index_missing_fields: true
    enforce_referential_integrity_on_delete: true
    enforce_referential_integrity_on_write: true
    etag_support_enabled: true
    expunge_enabled: false
    fhir_version: R4
    fhir_path_interceptor_enabled: false
    filter_search_enabled: true
    graphql_enabled: false
    binary_storage_enabled: false
    last-n-enabled: false
    mark-resources-for-reindexing-upon-search-parameter-change: false
    max_binary_size: 104857600
    max_page_size: 200
    bulk_export_enabled: false
    retain_cached_searches_mins: 60
    reuse_cached_search_results_millis: 60000
    partitioning:
      enabled: false
      cross_partition_reference_mode: true
      multitenancy_enabled: true
      partitioning_include_in_search_hashes: true
    persistenceUnitName: "HAPI_PERSISTENCE_UNIT"
    cors:
      enabled: false
      allow_credentials: true
      allowed_origin:
        - '*'
    expunge-thread-count: 2
    reindex-thread-count: 2
    search-total-mode: ESTIMATED
    search-coord-core-pool-size: 20
    search-coord-max-pool-size: 100
    search-coord-queue-capacity: 20
    status-based-reindexing-disabled: true
    normalized_quantity_search_level: NORMALIZED_QUANTITY_SEARCH_SUPPORTED
    enable_index_contained_resource: false
    validation:
      enabled: true
      requests_enabled: true
      responses_enabled: false
    logger:
      enabled: true
      name: "fhirtest.access"
      error_format: "[HAPI FHIR ERROR] - ${requestVerb} ${requestUrl}"
      format: "Path[${servletPath}] Source[${requestHeader.x-forwarded-for}] Operation[${operationType} ${operationName} ${idOrResourceName}] UA[${requestHeader.user-agent}] Params[${requestParameters}] ResponseEncoding[${responseEncodingNoDefault}] Operation[${operationType} ${operationName} ${idOrResourceName}] UA[${requestHeader.user-agent}] Params[${requestParameters}] ResponseEncoding[${responseEncodingNoDefault}]"
      log_exceptions: true
    elasticsearch:
      enabled: false

management:
  server:
    port: 8081
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
    db:
      enabled: true
  endpoint:
    health:
      enabled: true
      probes:
        enabled: true
      show-components: never
      show-details: never
      group:
        readiness:
          include: readinessState, db
    metrics.enabled: true
    prometheus.enabled: true
  endpoints.web.exposure.include: "*"
  metrics.export.prometheus.enabled: true

logging.level:
  ROOT: info
  com.epam.charity: info
  org.springframework: info

```

<h3>In comparing with the basic setup, we optimized our costs up to 5 times.<h3>

Thank you.

If you have any question, feel free to contact me direct in [linkedin](https://www.linkedin.com/in/vadzimkavalkou/).

Have a nice day, dude!

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
