---
title: "Getting Started: Setting Up a Gatling and Java Project From Scratch"
date: 2026-01-08T07:30:00+11:00
draft: false
slug: "getting-started-gatling-java-project-setup"
categories:
  - Gatling
description: "How to set up a Gatling performance testing project with Java and Maven, from an empty folder to your first passing build."
---

Performance testing has a reputation for being complicated to get into. Heavy tools, confusing scripting languages, a steep learning curve before you even run your first load test. Gatling is one of the tools that actually breaks that pattern, especially now that it has a proper Java DSL. If you already write Java for a living, you can be productive in Gatling within a day.

This is the first post in a twelve part series on building a real performance testing framework with Gatling and Java. We start right at the beginning, with project setup, and build up from there.

## What Gatling Actually Is

Gatling is a load testing tool built on top of an asynchronous, non blocking engine. That matters practically because a single Gatling instance can simulate a large number of concurrent virtual users without needing a thread per user the way some older tools do. You write a simulation once, describing what a user does and how many users you want to run, and Gatling handles the heavy lifting of actually generating that load.

The part that matters most for this series is that you write simulations in Java, using Gatling's Java DSL. No separate scripting language, no proprietary IDE plugin required. It is a Maven or Gradle project like any other Java project you already know how to work with.

## Setting Up the Project With Maven

The quickest way to get a working project is Gatling's official Maven archetype. Open a terminal and run this.

```bash
mvn archetype:generate \
  -DarchetypeGroupId=io.gatling.highcharts \
  -DarchetypeArtifactId=gatling-highcharts-maven-archetype \
  -DarchetypeVersion=LATEST
```

Maven will ask you for a group id, an artifact id, and a version, the same as any archetype based project. Once it finishes, you get a working folder structure with the Gatling Maven plugin already wired up.

If you prefer to add Gatling to an existing project instead of generating a fresh one, add the plugin and dependency directly to your `pom.xml`.

```xml
<dependencies>
    <dependency>
        <groupId>io.gatling.highcharts</groupId>
        <artifactId>gatling-charts-highcharts</artifactId>
        <version>LATEST</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>io.gatling</groupId>
            <artifactId>gatling-maven-plugin</artifactId>
            <version>LATEST</version>
        </plugin>
    </plugins>
</build>
```

Check the Maven Central page for the current version numbers before pinning them, since both the plugin and the charts dependency move forward together and need to stay in sync with each other.

## A Folder Layout That Scales

The archetype gives you a `src/test/java` folder for your simulations and a `src/test/resources` folder for data files. As the framework grows, it is worth organizing this further, the same way you would structure any real Java project.

```text
src/test/java/
├── simulations/
│   └── CheckoutLoadSimulation.java
├── scenarios/
│   └── CheckoutScenario.java
├── config/
│   └── HttpProtocolConfig.java
└── utils/
    └── EnvironmentConfig.java

src/test/resources/
├── data/
│   └── users.csv
└── bodies/
    └── login-request.json
```

`simulations` holds the actual Gatling simulation classes, the ones you run directly. `scenarios` holds reusable scenario definitions that simulations can compose together. `config` holds shared setup like your HTTP protocol configuration. `utils` holds small helpers, like reading environment variables for base URLs. We will fill most of these in properly over the next few posts.

## Writing a Minimal Simulation

Let's confirm everything works with the smallest possible simulation. Create this file under `src/test/java/simulations`.

```java
package simulations;

import io.gatling.javaapi.core.ScenarioBuilder;
import io.gatling.javaapi.core.Simulation;
import io.gatling.javaapi.http.HttpProtocolBuilder;

import static io.gatling.javaapi.core.CoreDsl.*;
import static io.gatling.javaapi.http.HttpDsl.*;

public class SmokeTestSimulation extends Simulation {

    HttpProtocolBuilder httpProtocol = http
        .baseUrl("https://example.com")
        .acceptHeader("application/json");

    ScenarioBuilder scn = scenario("Smoke Test")
        .exec(
            http("Home Page")
                .get("/")
                .check(status().is(200))
        );

    {
        setUp(
            scn.injectOpen(atOnceUsers(1))
        ).protocols(httpProtocol);
    }
}
```

A quick walk through of what each piece does. `httpProtocol` sets shared HTTP settings, in this case the base URL that every relative path in the simulation resolves against. `scn` describes a scenario, a single named request in this case, with a check that the response comes back with a 200 status. The block inside the curly braces is where you actually configure the run, telling Gatling to inject one single user at once against this protocol.

Run it directly with the Maven plugin.

```bash
mvn gatling:test
```

Gatling will print a live summary in the terminal as the run happens, and once it finishes, it generates a full HTML report on disk. We will dig into reading that report properly a few posts from now, but for now, a green summary with no failed requests is exactly what you want to see.

## Wrapping Up

At this point you have a working Gatling and Java project, a folder structure ready to grow, and one passing simulation to prove the setup works end to end.

Next time, we go deeper into the Java DSL itself. We will build a proper simulation with multiple requests, look at how `exec` and chaining work, and start separating scenario logic from simulation setup the way a real framework should.
