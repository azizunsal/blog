---
title: "Feature Toggles as Permissioning Toggles"
date: 2019-10-24T11:45:41+03:00
lastmod: 2019-10-24T11:45:41+03:00
draft: true
keywords: ["feature flags", "feature toggles", "togglez", "ff4j", "permissioning toggles", "Sentinel HL", "Sentinel HASP", "Spring Boot Profiles"]
description: ""
tags: ["feature toggles", "feature flags", "spring"]
categories: ["programming", "java", "spring"]
author: "Aziz Unsal <unsal.aziz@gmail.com>"

# You can also close(false) or open(true) something for this content.

# P. S.comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false

# You can also define another contentCopyright.e.g.contentCopyright: "This is another copyright."

contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show

hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.

# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

I'm sure that everyone in the software industry has heard of the term "feature toggles". I don't know how many of you have used this feature in your workflow but feature toggles are a great way to on/off a feature that has not been yet implemented fully. This method used mostly in the teams that follow continuous integration practices.

<!--more-->

## Intro to Feature Toggles

What I want to talk about is not the feature toggles nor their usage details. If you want to learn more about feature toggles you may want to read Martin Fowler's post. I want to talk about "permissioning toggles" and how we can use them with a licensing key/dongle to restrict the application's some features in some environments. The term "permissioning toggles" may differ in various contexts and one of the other most known term for it is "business toggles".

![Feature Toggles Overview](/blog/feature_toggles_overview_diagram.png)

[Image source] (<https://martinfowler.com/articles/feature-toggles.html>)

In this post, We'll create a  Spring Boot application and will read the features from a Sentinel protection key dynamically and then configure the application to bootstrap the classes/beans related to these features.

Spring Profiles are an easy way to segregate parts of your application config and make it available only in certain environments. However, for more complex situations, we currently have sophisticated frameworks like Togglez, FF4J.
Although we will read permissions/features from a license dongle and apply them at runtime, they can also be applied at build time. Both have different benefits.

## The Scenario

We will read licenses from Sentinel HL (formerly Sentinel HASP HL) and then applied them to the application. Before checking the optional features, we have to ensure that a licensing dongle exists and the required permissions were granted. In this sample application, we will use 3 different licensing schemes;

1. Core
2. Video
3. Camera

And according to the licensing API, the application will load proper classes and the others will remain silent during the lifetime of the application with the help of Spring Profiles.

## The Tech Stack We Will Be Used

1. Spring Boot, to build a fully functioning application.
2. The Sentinel Licensing API Java Class Library(hasp-srm-api.jar), provides access to Sentinel protection keys. We will not use JNI and runtime libraries for the sake of this post.
3. The Spring @Profile Annotation.

## Implementation

### Sentinel Login

#### Sentinel Login Code Excerpt

``` java
// Check the presence of the vendor-specific Sentinel protection key.
Hasp hasp = new Hasp(Hasp.HASP_DEFAULT_FID);
boolean haspLoginResult = hasp.login(VENDOR_CODE);
logger.debug("HASP login result for {} is {}", Hasp.HASP_DEFAULT_FID, haspLoginResult);
```

#### Sentinel Login Call Overview

![Sentinel Login Call Overview](/blog/sentinel-login-diagram.png)

``` java
Hasp hasp = new Hasp(Hasp.HASP_DEFAULT_FID);
```

When `HASP_DEFAULT_FID`is used this means that we are not logging into a specific feature, but we are checking the presence of the Sentinel Protection key/dongle and this is the first step we will check then we can continue to check specific features.

More detailed information can be found in [Sentinel Licensing API documentation] (https://docs.sentinel.gemalto.com/ldk/LDKdocs/SPNL/LDK_SLnP_Guide/Protection/Licensing_API_protection/Login%20Function.htm)

### Aligning Application Profiles According To The Licenses

As described above, we must be logging into each specific feature that we want to use.

``` java
private static String checkAndLoginToFeature(final ApplicationFeature applicationFeature) {
    Hasp hasp = new Hasp(applicationFeature.featureId());
    boolean haspLoginResult = hasp.login(VENDOR_CODE);
    logger.debug("HASP login result for '{}' is '{}'", applicationFeature.featureId(), haspLoginResult);
    int statusCode = hasp.getLastError();
    if (statusCode != HaspStatus.HASP_STATUS_OK) {
        getHaspError(statusCode);
        return null;
    }

    return applicationFeature.licenseString();
}
```

And here is the starting point of the application;

``` java
@SpringBootApplication
public class PermissioningTogglesSampleApplication {
    private static final Logger logger = LoggerFactory.getLogger(PermissioningTogglesSampleApplication.class);

    public static void main(String[] args) {
        final HaspResult haspResult = HaspHelper.checkLicense();
        if (haspResult.getStatusCode() == HaspStatus.HASP_STATUS_OK) {
            logger.info("Sentinel HL check is ok, adding additional profiles {}", haspResult.getLicenses());
            SpringApplication app = new SpringApplication(PermissioningTogglesSampleApplication.class);
            app.setAdditionalProfiles(StringUtils.toStringArray(haspResult.getLicenses()));
            app.run(args);
        } else {
            logger.error("Sentinel HL check is NOT ok!");
        }
    }
}
```

You will notice the line `app.setAdditionalProfiles ...` in the main method, `checkAndLoginToFeature` method returns a string like `video`, `camera` and we have corresponding application profiles under resources.

- application.yml
- application-camera.yml
- application-video.yml

The last two profiles are optional and they are aligned by the application at the runtime dynamically.

``` java
@Profile("video")
@RestController
@RequestMapping(value = "videos")
public class VideoController {
```

``` java
@Service
@Profile("camera")
public class CameraServiceImpl implements CameraService {
```

We are using `@Profile`s to bootstrap the appropriate beans.

## The Result

### When Successfully Logged In But No Optional Features Were Found

```
azizunsal:permissioningtoggles/ (master✗) $ ./gradlew assemble && java -jar build/libs/permissioningtoggles-0.0.1-SNAPSHOT.jar

BUILD SUCCESSFUL in 1s
3 actionable tasks: 2 executed, 1 up-to-date
11:14:39.619 [main] DEBUG com.azizunsal.blog.permissioningtoggles.helper.HaspHelper - HASP login result for 0 is true
11:14:39.627 [main] DEBUG com.azizunsal.blog.permissioningtoggles.helper.HaspHelper - HASP login result for '100' is 'false'
11:14:39.627 [main] INFO com.azizunsal.blog.permissioningtoggles.helper.HaspHelper - The HASP error description for code '31' is 'Requested Feature not found!'
11:14:39.631 [main] DEBUG com.azizunsal.blog.permissioningtoggles.helper.HaspHelper - HASP login result for '200' is 'false'
11:14:39.631 [main] INFO com.azizunsal.blog.permissioningtoggles.helper.HaspHelper - The HASP error description for code '31' is 'Requested Feature not found!'
11:14:39.632 [main] INFO com.azizunsal.blog.permissioningtoggles.PermissioningTogglesSampleApplication - Sentinel HL check is ok, adding additional profiles []

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.0.RELEASE)


2019-10-31 11:14:40.753  INFO 22565 --- [           main] .p.PermissioningTogglesSampleApplication : No active profile set, falling back to default profiles: default

```

### When Successfully Logged In And Optional Features Were Found

```
azizunsal:permissioningtoggles/ (master✗) $ ./gradlew assemble && java -jar build/libs/permissioningtoggles-0.0.1-SNAPSHOT.jar

BUILD SUCCESSFUL in 1s
3 actionable tasks: 2 executed, 1 up-to-date
10:33:57.057 [main] DEBUG com.azizunsal.blog.permissioningtoggles.helper.HaspHelper - HASP login result for 0 is true
10:33:57.067 [main] DEBUG com.azizunsal.blog.permissioningtoggles.helper.HaspHelper - HASP login result for '1' is 'true'
10:33:57.081 [main] DEBUG com.azizunsal.blog.permissioningtoggles.helper.HaspHelper - HASP login result for '2' is 'true'
10:33:57.081 [main] INFO com.azizunsal.blog.permissioningtoggles.PermissioningTogglesSampleApplication - Sentinel HL check is ok, adding additional profiles [video, camera]

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.0.RELEASE)


2019-10-31 10:33:57.671  INFO 71763 --- [           main] .p.PermissioningTogglesSampleApplication : The following profiles are active: video,camera

```

### Overall Application Structure

```
azizunsal:permissioningtoggles/ (master✗) $ tree
.
├── HELP.md
├── README.md
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── hasp_darwin_102249.dylib -> /usr/local/lib/corvo/hasp_darwin_102249.dylib
├── libHASPJava.dylib -> /usr/local/lib/corvo/libHASPJava.dylib
├── libHASPJava.jnilib -> /usr/local/lib/corvo/libHASPJava.jnilib
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── azizunsal
    │   │           └── blog
    │   │               └── permissioningtoggles
    │   │                   ├── PermissioningTogglesSampleApplication.java
    │   │                   ├── controller
    │   │                   │   ├── CameraController.java
    │   │                   │   └── VideoController.java
    │   │                   ├── enums
    │   │                   │   └── ApplicationFeature.java
    │   │                   ├── exception
    │   │                   │   ├── CameraNotFoundException.java
    │   │                   │   └── VideoNotFoundException.java
    │   │                   ├── helper
    │   │                   │   └── HaspHelper.java
    │   │                   ├── model
    │   │                   │   ├── dto
    │   │                   │   │   └── HaspResult.java
    │   │                   │   └── entity
    │   │                   │       ├── AbstractModel.java
    │   │                   │       ├── Camera.java
    │   │                   │       └── Video.java
    │   │                   ├── repository
    │   │                   │   ├── CameraRepository.java
    │   │                   │   └── VideoRepository.java
    │   │                   └── service
    │   │                       ├── CameraService.java
    │   │                       ├── VideoService.java
    │   │                       └── impl
    │   │                           ├── CameraServiceImpl.java
    │   │                           └── VideoServiceImpl.java
    │   └── resources
    │       ├── application-camera.yml
    │       ├── application-video.yml
    │       ├── application.yml
    │       └── libs
    │           └── hasp-srm-api.jar
    └── test
        └── java
            └── com
                └── azizunsal
                    └── blog
                        └── permissioningtoggles
                            └── PermissioningTogglesSampleApplicationTests.java

27 directories, 33 files
```

You can find the full code in my [GitHub Repository] (htttps://github.com/azizunsal/XXX)

## Sentinel Envelope

Now, we applied a simple licensing scheme to our application but it is far from a completely secure application. Guess what, the code can easily be read and changed. If we want this application to be safe there is one more important step that we have to complete; securing the application against *reverse engineering*.

![Sentinel Envelope Overview](/blog/sentinel-envelope-overview.png)

Although there are 3rd party solutions like ProGuard, I recommend using commercial support for this boring task and Sentinel Envelope will do the job for you.
