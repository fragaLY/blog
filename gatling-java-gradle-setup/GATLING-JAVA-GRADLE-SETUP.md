------------------------------------------------------------------------------------------------------------------------

![](./static/main.jpeg)

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 1: THERE'S NO SCHOOL LIKE OLD SCHOOL, AND I'M THE F#$K!9G HEADMASTER.</h6>

:mechanical_arm: Hi, folks,

Preparing for my number of tech talks regarding the performance of different frameworks, I faced the problem of configuring a single approach to test it.
The scenarios and the system will be presented later, it's out of the scope of this post.

Why did I decide to choose Gatling?
For me it was a bit easy, I am already familiar with Apache JMeter and I would like to gain a new skill.

Load Testing with Gatling allows you to emulate heavy traffic, get reports and find potential bugs. Capacity tests, soak tests or stress tests can be done without much effort, in Java, Kotlin, and Scala, all along your CI/CD process:
- On-demand load injectors
- Advanced reporting
- Public APIs and Grafana datasource
- Continuous integration
- Clustering/Distributed mode

As you can see, I could write scenarios in Java, Kotlin, and Scala.
As my primary skill is Java I decided to choose it.

What about Gradle, it has a great demand nowadays and is more flexible for me.

So we are going with Gatling, Java, and Gradle.
Fasten your seat belts.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 2: DADDY! NICE WHEELS!.</h6>

Now just technical stuff regarding the preset of Gatling using Gradle.

<b>Gradle Settings</b>

``` gradle
plugins {
    id "java"
    id "io.gatling.gradle" version "3.7.6.3"
}

group 'by.vk'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    //region gatling
    implementation("io.gatling:gatling-core:3.7.6")
    implementation("io.gatling:gatling-http:3.7.6")
    implementation("io.gatling:gatling-app:3.7.6")
    gatlingRuntimeOnly("io.gatling:gatling-charts:3.7.6")
    gatlingRuntimeOnly("io.gatling.highcharts:gatling-charts-highcharts:3.7.6")
    //endregion
}

gatling {
    logLevel = 'WARN'
    logHttp = 'ALL'
    systemProperties = ['file.encoding': 'UTF-8']
    jvmArgs = [
            '-server',
            '-Xss256k',
            '-Xms1024m',
            '-Xmx2G',
            '-XX:MaxMetaspaceSize=128m',
            '-XX:+HeapDumpOnOutOfMemoryError',
            '-XX:+UseG1GC',
            '-XX:+ParallelRefProcEnabled',
            '-XX:MaxInlineLevel=20',
            '-XX:MaxTrivialSize=12',
            '-XX:+UseStringDeduplication',
            '-XX:+ExitOnOutOfMemoryError',
            '-XX:+OptimizeStringConcat',
            '-XX:HeapDumpPath=/opt/tmp/heapdump.bin'
    ]
}

```

<b>Gatling configuration:</b>

``` conf
gatling {
  core {
    runDescription = "A2B Gatling tests"
    encoding = "utf-8"           # Encoding to use throughout Gatling for file and string manipulation
    elFileBodiesCacheMaxCapacity = 0      # Cache size for request body EL templates, set to 0 to disable
    rawFileBodiesCacheMaxCapacity = 0     # Cache size for request body Raw templates, set to 0 to disable
    #rawFileBodiesInMemoryMaxSize = 1000     # Below this limit, raw file bodies will be cached in memory
    pebbleFileBodiesCacheMaxCapacity = 0  # Cache size for request body Peeble templates, set to 0 to disable
    #feederAdaptiveLoadModeThreshold = 100   # File size threshold (in MB). Below load eagerly in memory, above use batch mode with default buffer size
    #shutdownTimeout = 10000                 # Milliseconds to wait for the actor system to shutdown
    extract {
      regex {
        cacheMaxCapacity = 0 # Cache size for the compiled regexes, set to 0 to disable caching
      }
      xpath {
        cacheMaxCapacity = 0 # Cache size for the compiled XPath queries,  set to 0 to disable caching
      }
      jsonPath {
        cacheMaxCapacity = 0 # Cache size for the compiled jsonPath queries, set to 0 to disable caching
      }
      css {
        cacheMaxCapacity = 0 # Cache size for the compiled CSS selectors queries,  set to 0 to disable caching
      }
    }
    directory {
      #simulations = user-files/simulations # Directory where simulation classes are located (for bundle packaging only)
      #resources = user-files/resources     # Directory where resources, such as feeder files and request bodies are located (for bundle packaging only)
      #reportsOnly = ""                     # If set, name of report folder to look for in order to generate its report
      #binaries = ""                        # If set, name of the folder where compiles classes are located: Defaults to GATLING_HOME/target.
      #results = results                    # Name of the folder where all reports folder are located
    }
  }
  socket {
    connectTimeout = 1000                 # Timeout in millis for establishing a TCP socket
    #tcpNoDelay = true
    soKeepAlive = true                    # if TCP keepalive configured at OS level should be used
    #soReuseAddress = false
  }
  netty {
    useNativeTransport = true              # if Netty native transport should be used instead of Java NIO
    #allocator = "pooled"                   # switch to unpooled for unpooled ByteBufAllocator
    #maxThreadLocalCharBufferSize = 200000  # Netty's default is 16k
  }
  ssl {
    useOpenSsl = false                    # if OpenSSL should be used instead of JSSE (only the latter can be debugged with -Djava.net.debug=ssl)
    #useOpenSslFinalizers = false         # if OpenSSL contexts should be freed with Finalizer or if using RefCounted is fine
    #handshakeTimeout = 10000             # TLS handshake timeout in millis
    #useInsecureTrustManager = true       # Use an insecure TrustManager that trusts all server certificates
    #enabledProtocols = []             # Array of enabled protocols for HTTPS, if empty use Netty's defaults
    #enabledCipherSuites = []          # Array of enabled cipher suites for HTTPS, if empty enable all available ciphers
    #sessionCacheSize = 0              # SSLSession cache size, set to 0 to use JDK's default
    #sessionTimeout = 0                # SSLSession timeout in seconds, set to 0 to use JDK's default (24h)
    #enableSni = true                     # When set to true, enable Server Name indication (SNI)
    keyStore {
      #type = ""      # Type of SSLContext's KeyManagers store
      #file = ""      # Location of SSLContext's KeyManagers store
      #password = ""  # Password for SSLContext's KeyManagers store
      #algorithm = "" # Algorithm used SSLContext's KeyManagers store
    }
    trustStore {
      #type = ""      # Type of SSLContext's TrustManagers store
      #file = ""      # Location of SSLContext's TrustManagers store
      #password = ""  # Password for SSLContext's TrustManagers store
      #algorithm = "" # Algorithm used by SSLContext's TrustManagers store
    }
  }
  charting {
    noReports = false       # When set to true, don't generate HTML reports
    maxPlotPerSeries = 1000 # Number of points per graph in Gatling reports
    #useGroupDurationMetric = false  # Switch group timings from cumulated response time to group duration.
    indicators {
      #lowerBound = 800      # Lower bound for the requests' response time to track in the reports and the console summary
      #higherBound = 1200    # Higher bound for the requests' response time to track in the reports and the console summary
      #percentile1 = 50      # Value for the 1st percentile to track in the reports, the console summary and Graphite
      #percentile2 = 75      # Value for the 2nd percentile to track in the reports, the console summary and Graphite
      #percentile3 = 95      # Value for the 3rd percentile to track in the reports, the console summary and Graphite
      #percentile4 = 99      # Value for the 4th percentile to track in the reports, the console summary and Graphite
    }
  }
  http {
    fetchedCssCacheMaxCapacity = 0          # Cache size for CSS parsed content, set to 0 to disable
    fetchedHtmlCacheMaxCapacity = 0         # Cache size for HTML parsed content, set to 0 to disable
    perUserCacheMaxCapacity = 0             # Per virtual user cache size, set to 0 to disable
    #warmUpUrl = "https://gatling.io"          # The URL to use to warm-up the HTTP stack (blank means disabled)
    #enableGA = true                           # Very light Google Analytics (Gatling and Java version), please support
    #pooledConnectionIdleTimeout = 60000       # Timeout in millis for a connection to stay idle in the pool
    #requestTimeout = 60000                    # Timeout in millis for performing an HTTP request
    #enableHostnameVerification = false        # When set to true, enable hostname verification: SSLEngine.setHttpsEndpointIdentificationAlgorithm("HTTPS")
    dns {
      #queryTimeout = 5000                     # Timeout in millis of each DNS query in millis
      #maxQueriesPerResolve = 6                # Maximum allowed number of DNS queries for a given name resolution
    }
  }
  jms {
    #replyTimeoutScanPeriod = 1000  # scan period for timedout reply messages
  }
  data {
    #writers = [console, file]      # The list of DataWriters to which Gatling write simulation data (currently supported : console, file, graphite)
    console {
      #light = false                # When set to true, displays a light version without detailed request stats
      #writePeriod = 5              # Write interval, in seconds
    }
    file {
      #bufferSize = 8192            # FileDataWriter's internal data buffer size, in bytes
    }
    leak {
      #noActivityTimeout = 30  # Period, in seconds, for which Gatling may have no activity before considering a leak may be happening
    }
    graphite {
      #light = false              # only send the all* stats
      #host = "localhost"         # The host where the Carbon server is located
      #port = 2003                # The port to which the Carbon server listens to (2003 is default for plaintext, 2004 is default for pickle)
      #protocol = "tcp"           # The protocol used to send data to Carbon (currently supported : "tcp", "udp")
      #rootPathPrefix = "gatling" # The common prefix of all metrics sent to Graphite
      #bufferSize = 8192          # Internal data buffer size, in bytes
      #writePeriod = 1            # Write period, in seconds
    }
  }
}

```

<b>Logback configuration</b>

``` xml
<configuration>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %highlight(%-5level) %cyan(%-40logger{36}) - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="io.gatling" level="INFO"/>
    <logger name="io.gatling.http.ahc" level="TRACE"/>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>

</configuration>

```

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 3: SO DO US ALL A FAVOR BEFORE YOU CATCH A COLD.</h6>

<b>Scenarios sample on Java</b>

``` java
package by.vk.scenario.a2b;

import io.gatling.javaapi.core.ScenarioBuilder;
import io.gatling.javaapi.core.Simulation;
import io.gatling.javaapi.http.HttpProtocolBuilder;
import io.netty.handler.codec.http.HttpResponseStatus;

import java.time.Duration;
import java.util.Collections;
import java.util.Iterator;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ThreadLocalRandom;
import java.util.function.Supplier;
import java.util.stream.Stream;

import static io.gatling.javaapi.core.CoreDsl.StringBody;
import static io.gatling.javaapi.core.CoreDsl.jsonPath;
import static io.gatling.javaapi.core.CoreDsl.rampUsersPerSec;
import static io.gatling.javaapi.core.CoreDsl.scenario;
import static io.gatling.javaapi.http.HttpDsl.http;
import static io.gatling.javaapi.http.HttpDsl.status;

public class A2BSimulation extends Simulation {

    private static final Iterator<Map<String, Object>> FEEDER = Stream.generate((Supplier<Map<String, Object>>) () -> Collections.singletonMap("userId", ThreadLocalRandom.current().nextLong(1, 200_001))).iterator();

    final HttpProtocolBuilder protocol = http
            .warmUp("https://www.google.com")
            .baseUrl("http://localhost:8080/api/v1")
            .acceptHeader("application/json")
            .acceptLanguageHeader("en-US,en;q=0.5")
            .acceptEncodingHeader("gzip, deflate")
            .userAgentHeader("Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.64 Safari/537.36");

    final ScenarioBuilder scenario = scenario("A2B simulation")
            .feed(FEEDER)
            .exec(http("[GET] The list of available countries is presented.")
                    .get("/countries")
                    .check(status().is(HttpResponseStatus.OK.code()))
                    .check(jsonPath("$[:].countryId").findRandom().saveAs("countryId"))
            )
            .exitHereIfFailed()
            .pause(1, 3)
            .exec(http("[GET] The list of available cities is presented.")
                    .get("/countries/#{countryId}/cities")
                    .check(status().is(HttpResponseStatus.OK.code()))
                    .check(jsonPath("$[:].cityId").findRandom().saveAs("cityOriginId"))
            )
            .exitHereIfFailed()
            .pause(1, 3)
            .exec(http("[GET] The list of available origins is presented.")
                    .get("/countries/#{countryId}/cities/#{cityOriginId}/locations")
                    .check(status().is(HttpResponseStatus.OK.code()))
                    .check(jsonPath("$[:].locationId").findRandom().saveAs("originId"))
            )
            .pause(3, 5)
            .exec(http("[GET] The list of available cities is presented.")
                    .get("/countries/#{countryId}/cities")
                    .check(status().is(HttpResponseStatus.OK.code()))
                    .check(jsonPath("$[:].cityId").findRandom().saveAs("cityDestinationId"))
            )
            .exitHereIfFailed()
            .pause(1, 3)
            .exec(http("[GET] The list of available destinations is presented.")
                    .get("/countries/#{countryId}/cities/#{cityDestinationId}/locations")
                    .check(status().is(HttpResponseStatus.OK.code()))
                    .check(jsonPath("$[:].locationId").findRandom().saveAs("destinationId"))
            )
            .exitHereIfFailed()
            .pause(3, 5)
            .exec(http("[GET] The list of available transfers by selected origin, destination, and date is presented.")
                    .get("/transfers")
                    .queryParam("originId", "#{originId}")
                    .queryParam("destinationId", "#{destinationId}")
                    .queryParam("date", "1970-01-01")
                    .check(status().in(HttpResponseStatus.OK.code(), HttpResponseStatus.NOT_FOUND.code()))
                    .check(jsonPath("$[:].transferId").findRandom().saveAs("transferId"))
            )
            .exitHereIfFailed()
            .pause(3, 5)
            .exec(http("[POST] The transfer is booked in the system.")
                    .post("/users/#{userId}/transfers/#{transferId}")
                    .check(status().is(HttpResponseStatus.CREATED.code()))
                    .body(StringBody("{ \"description\": \"My internal uuid is " + UUID.randomUUID() + "\" }"))
                    .asJson()
            )
            .pause(1, 3)
            .exec(http("[GET] The list of all my transfers (COMPLETED, CANCELED, BOOKED) is presented.")
                    .get("/users/#{userId}/transfers")
                    .check(status().is(HttpResponseStatus.OK.code()))
                    .check(jsonPath("$[:].transfer.transferId").findRandom().saveAs("selectedUserTransferId"))
            )
            .exitHereIfFailed()
            .pause(1, 3)
            .exec(http("[GET] One of the transfers (COMPLETED, CANCELED, BOOKED) is retrieved.")
                    .get("/users/#{userId}/transfers/#{selectedUserTransferId}")
                    .check(status().is(HttpResponseStatus.OK.code()))
            )
            .pause(1, 5)
            .exec(http("[PUT] Any (BOOKED) transfer description is updated.")
                    .put("/users/#{userId}/transfers/#{selectedUserTransferId}")
                    .check(status().is(HttpResponseStatus.NO_CONTENT.code()))
                    .body(StringBody("{\"state\": \"BOOKED\", \"description\": \"My new internal UUID " + UUID.randomUUID() + "\" }"))
                    .asJson()
            )
            .pause(1, 3)
            .exec(http("[PUT] Any (BOOKED) transfer is canceled (CANCELED).")
                    .put("/users/#{userId}/transfers/#{selectedUserTransferId}")
                    .check(status().is(HttpResponseStatus.NO_CONTENT.code()))
                    .body(StringBody("{\"state\": \"CANCELED\", \"description\": \"My new internal UUID for canceled transfer " + UUID.randomUUID() + "\" }"))
                    .asJson()
            )
            .pause(1, 3)
            .exec(http("[PUT] Any (BOOKED) transfer is completed (COMPLETED).")
                    .put("/users/#{userId}/transfers/#{selectedUserTransferId}")
                    .check(status().is(HttpResponseStatus.NO_CONTENT.code()))
                    .body(StringBody("{\"state\": \"COMPLETED\", \"description\": \"My new internal UUID for completed transfer " + UUID.randomUUID() + "\" }"))
                    .asJson()
            )
            .pause(1, 3)
            .exec(http("[GET] User is retrieved with her/his transfers data.")
                    .get("/users/#{userId}")
                    .check(status().is(HttpResponseStatus.OK.code()))
            )
            .pause(1, 3)
            .exec(http("[PUT] User updates with her/his own data.")
                    .put("/users/#{userId}")
                    .check(status().is(HttpResponseStatus.NO_CONTENT.code()))
                    .body(StringBody("{ \"firstName\": \" NewFirstName#{userId} \", \"lastName\": \"NewLastName#{userId}\" }"))
                    .asJson()
            )
            .pause(1, 5);

    {
        setUp(scenario.injectOpen(rampUsersPerSec(100).to(200).during(Duration.ofSeconds(1000)).randomized()).protocols(protocol)).maxDuration(Duration.ofSeconds(2000));
    }

}

```

To run in feel free to execute the next command: >gradlew gatlingRun

After the execution, you will be able to find the result in the 'build/reports/gatling' path.
Feel free to open it as an HTML page and investigate the results.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 5: FOR A MARRIAGE OF CONVENIENCE, THIS CAN BE QUITE INCONVENIENT.</h6>

Moreover, there is a :gear:	[repository](https://github.com/fragaLY/performance-researches/tree/master/gatling) regarding this settings.

If you have any question, feel free to contact me direct in [linkedin](https://www.linkedin.com/in/vadzimkavalkou/).

Have a nice day.

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
