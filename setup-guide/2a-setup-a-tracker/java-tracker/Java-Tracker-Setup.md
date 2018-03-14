<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2: setup a Tracker**](Setting-up-a-Tracker) > [**Java tracker**](Java-tracker-setup)

This page refers to version 0.8.0+ of the Snowplow Java Tracker.

Documentation for older version of this tracker is available:

* [Java 0.7][java-0.7]

## Contents

- 1 [Overview](#overview)
- 2 [Integration options](#integration-options)
  - 2.1 [Tracker compatibility](#compatibility)
  - 2.2 [Dependencies](#dependencies)
- 3 [Setup](#setup)
  - 3.1 [Hosting](#hosting)
  - 3.2 [Maven](#maven)
  - 3.3 [Gradle](#gradle)
  - 3.4 [SBT](#sbt)

<a name="overview" />

## 1. Overview

The [Snowplow Java Tracker](https://github.com/snowplow/snowplow-java-tracker) lets you add analytics to your [Java][java]-based desktop and server apps, servlets and games.

The Tracker should be relatively straightforward to setup if you are familiar with Java development.

Ready? Let's get started.

[Back to top](#top)

<a name="integration-options" />

## 2. Integration options

<a name="compatibility" />

### 2.1 Tracker compatibility

The Snowplow Java Tracker has been built and tested using JDK7 (JRE 1.7), so should work within any Java application built using JDK7 upwards.

[Back to top](#top)

<a name="dependencies" />

### 2.2 Dependencies

To minimize jar bloat, we have tried to keep external dependencies to a minimum. For the full list of dependencies, please see our [Gradle build file][gradle-build].

[Back to top](#top)

<a name="setup" />

## 3. Setup

<a name="hosting" />

### 3.1 Hosting

The Tracker is published to Snowplow's [hosted Maven repository][maven-snplow], which should make it easy to add it as a dependency into your own Java app.

The current version of the Snowplow Java Tracker is 0.8.2.

You can also manually insert the Tracker by downloading the jar directly: [snowplow-java-tracker-0.8.2.jar](http://maven.snplow.com/releases/com/snowplowanalytics/snowplow-java-tracker/0.8.2/snowplow-java-tracker-0.8.2.jar)

<a name="maven" />

### 3.2 Maven

If you are using Maven for building your Java application, then add the following code into your `HOME/.m2/settings.xml` to be able to use this repository:

```xml
<settings>
  <profiles>
    <profile>
      <!-- ... -->
      <repositories>
        <repository>
          <id>com.snowplowanalytics</id>
          <name>SnowPlow Analytics</name>
          <url>http://maven.snplow.com/releases</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>
</settings>
```

Then add into your project's `pom.xml`:

```xml
<dependency>
    <groupId>com.snowplowanalytics</groupId>
    <artifactId>snowplow-java-tracker</artifactId>
    <version>0.8.2</version>
</dependency>
```

<a name="gradle" />

### 3.3 Gradle

If you are using Gradle in your own Java application, then add our Maven repository in your `build.gradle` file:

```groovy
repositories {
    ...
    maven {
        url "http://maven.snplow.com/releases"
    }
}
```

Then add into the same file:

```groovy
dependencies {
    ...
    // Snowplow Java Tracker
    compile 'com.snowplowanalytics:snowplow-java-tracker:0.8.2'
}
```

<a name="sbt" />

### 3.4 SBT

The Snowplow Java Tracker is also usable from Scala. Add this to your SBT config:

```scala
// Resolvers
val snowplowRepo = "SnowPlow Repo" at "http://maven.snplow.com/releases/"

// Dependency
val snowplowTracker = "com.snowplowanalytics"  % "snowplow-java-tracker"  % "0.8.2"
```

Done? Now read the [Java Tracker API](Java-Tracker) to start tracking events.

[java]: http://www.java.com/en/
[java-0.7]: https://github.com/snowplow/snowplow/wiki/Java-Tracker-Setup-0.7

[gradle-build]: https://github.com/snowplow/snowplow-java-tracker/blob/master/build.gradle
[maven-snplow]: http://maven.snplow.com
