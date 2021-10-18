------------------------------------------------------------------------------------------------------------------------

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
![](./static/hapi-fhir-jpa-architecture.png)

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

The basic configuration for HFJSS had been tuned a bit, mostly tuning affected pools and workers, some searching, validation, and reindexing settings.
If you are not a lot familiar with hikari pool tuning, there is a [great article](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing) about pool sizing. Long story short,
The `application.yaml` file with basic settings that could be useful for you has the next structure:

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

- [ ] Clarify the performance tests we've used.
------------------------------------------------------------------------------------------------------------------------
<h6>CHAPTER 4: WHERE'S MY RESOURCES, LEBOWSKI?</h6>

Moreover, it would be nice to provide you with some information regarding the datasets we've used.
The `Resources` we've used are the next (all the multipliers for resources that related to `Patient` were gained from researching the institutions and organization):

| Resource          | Multiplier | 1 step   | 2 step     | 3 step     | 4 step     | 5 step      |
| :---------------- | :----------| :--------| :--------  | :--------  | :--------  | :--------   |
| Patient           | x1         | 50_000   | 100_000    | 250_000    | 500_000    | 1_000_000   |
| Observation       | x100       | 5_000_000| 10_000_000 | 25_000_000 | 50_000_000 | 100_000_000 |
| Diagnostic Report | x10        | 500_000  | 1_000_000  | 2_500_000  | 5_000_000  | 10_000_000  |
| Condition         | x10        | 500_000  | 1_000_000  | 2_500_000  | 5_000_000  | 10_000_000  |
| Procedure         | x3         | 150_000  | 300_000    | 750_000    | 1_500_000  | 3_000_000   |
| Episode Of Care   | x2         | 100_000  | 200_000    | 500_000    | 1_000_000  | 2_000_000   |
| Composition       | x2         | 100_000  | 200_000    | 500_000    | 1_000_000  | 2_000_000   |
| DB dump size (GiB)| `N/A`      | 6,5      | 15         | 43         | 100        | 230         |

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 5: WHERE'S MY PERFORMANCE, LEBOWSKI?</h6>

At the very beginning of this chapter, I would like to share with you some special secrets.
The hapi fhir server is a healthcare one and the business is not going to deal with a huge RPS.
But the metrics that I will share with you soon could help us to understand the possible scaling strategy better, follow fault-tolerance best practices, and provide the vision of resources consumption.
The last but not least, all these metrics can help us to predict some cost management.

Let's assume that the average institution has the biggest load from 8:00 till 17:00.
As a result, we will have ~40 business hours per week. The next metrics will be presented for a typical business day (transactions per working day, TPWD in future) and RPS as well.
And now, show me the numbers, dude!

<h2>Tomcat</h6>

- G1

| Step | RPS | TPWD | RAM  | CPU  | JVM Metrics  |
| :--- | :---| :----| :----| :----| :------------|
| 1    |     |      |      |      |              |
| 2    |     |      |      |      |              |
| 3    |     |      |      |      |              |
| 4    |     |      |      |      |              |
| 5    |     |      |      |      |              |

- ZGC

| Step | RPS | TPWD | RAM  | CPU  | JVM Metrics  |
| :--- | :---| :----| :----| :----| :------------|
| 1    |     |      |      |      |              |
| 2    |     |      |      |      |              |
| 3    |     |      |      |      |              |
| 4    |     |      |      |      |              |
| 5    |     |      |      |      |              |

<h2>Jetty</h6>

- G1

| Step | RPS | TPWD | RAM  | CPU  | JVM Metrics  |
| :--- | :---| :----| :----| :----| :------------|
| 1    |     |      |      |      |              |
| 2    |     |      |      |      |              |
| 3    |     |      |      |      |              |
| 4    |     |      |      |      |              |
| 5    |     |      |      |      |              |

- ZGC

| Step | RPS | TPWD | RAM  | CPU  | JVM Metrics  |
| :--- | :---| :----| :----| :----| :------------|
| 1    |     |      |      |      |              |
| 2    |     |      |      |      |              |
| 3    |     |      |      |      |              |
| 4    |     |      |      |      |              |
| 5    |     |      |      |      |              |

<h2>Undertow</h6>

- G1

| Step | RPS | TPWD | RAM  | CPU  | JVM Metrics  |
| :--- | :---| :----| :----| :----| :------------|
| 1    |     |      |      |      |              |
| 2    |     |      |      |      |              |
| 3    |     |      |      |      |              |
| 4    |     |      |      |      |              |
| 5    |     |      |      |      |              |

- ZGC

| Step | RPS | TPWD | RAM  | CPU  | JVM Metrics  |
| :--- | :---| :----| :----| :----| :------------|
| 1    |     |      |      |      |              |
| 2    |     |      |      |      |              |
| 3    |     |      |      |      |              |
| 4    |     |      |      |      |              |
| 5    |     |      |      |      |              |
------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 6: WHERE'S MY BOOT-UP TIME, LEBOWSKI?</h6>

All the steps that we can find in the table below were described in chapter 4.
Now we will take a look at boot-up metrics on different embedded web servers.

| Step | Tomcat   | Jetty    | Undertow |
| :--- | :------- | :------- | :------- | 
| 1    | 48.974s  |          |          |
| 2    |          |          |          |
| 3    |          |          |          |
| 4    |          |          |          |
| 5    |          |          |          |

Moreover, to decrease the boot-up time we added [spring-indexer](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-scanning-index).
While classpath scanning is very fast, it is possible to improve the startup performance of large applications by creating a static list of candidates at compilation time.
In this mode, all modules that are targets of component scanning must use this mechanism.
It creates `META-INF/spring.components` were all the beans described.

Actually, it increases the build time in few seconds, but every cloud has a silver lining.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 7: I CAN'T BE WORRYING ABOUT THAT SH1T. LIFE GOES ON, MAN.</h6>


Thank you.

If you have any question, feel free to contact me direct in linkedin or via email.

Have a nice day, dude!

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
