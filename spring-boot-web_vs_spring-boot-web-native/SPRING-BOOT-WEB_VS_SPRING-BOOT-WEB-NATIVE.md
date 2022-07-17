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

I will not dive deeper into AOT and GraalVM. This is out of the scope of this article.

So, what I am going to check? I would like to check the performance of applications using Tomcat, Jetty, Undertow, JIB-built image, and Native Application using both native build tools and build pack.

Let's move forward.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 2: MAMA SAYS STUPID IS AS STUPID DOES.</h6>

BUSINESS DESCRIPTION.
DATABASE DESCRIPTION.

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
