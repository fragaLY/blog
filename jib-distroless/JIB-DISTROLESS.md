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
|242MB|39(0 critical, 2 high)    |

![](./static/dive_distroless.jpeg)

![](./static/grype_distroless.jpeg)

I know what you are thinking about. The size is still pretty big. Use Native, Vadzim. Migrate to Go, Rust...
This is kinda a miracle in the business world. I believe, you understand me and migration costs and risks.

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER 3: -TREVOR, IS SOMEONE CHASING YOU? -NOT YET. BUT THEY WILL WHEN THEY FIND OUT WHO I AM.</h6>

To be honest, I am sharing with you results and it's up to you to bring this approach in your project or not.

**But for us it was a significant improvement:**
- 3 times less size
- 13 times less vulnerabilities

**And as a result:**

- less time to test
- less time to build and deliver
- less time to fix vulnerabilities
- less time to deploy and rollback
- less time to fix production issues
- less money to spend on infrastructure, for storing, delivering, and transferring images
- more time to drink coffee and find new ways to improve your product

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
