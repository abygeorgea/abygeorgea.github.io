---
title: "Correlation and Authentication: Extracting Tokens and Handling Login Flows"
date: 2026-03-05T07:30:00+11:00
draft: false
slug: "gatling-correlation-and-authentication"
categories:
  - Gatling
description: "Extracting session tokens and other dynamic values from Gatling responses with check, and reusing them across an authenticated scenario."
---

In the [previous post](/blog/2026/02/19/gatling-feeders-csv-json-data-driven-tests/), we fed real data into our scenarios. Today we cover something almost every real application needs before you can test anything interesting behind a login screen. Correlation.

Correlation just means grabbing a value out of one response and reusing it in a later request. The most common example by far is authentication. You log in once, get back a token, and then attach that token to every request that follows.

## Extracting a Token With saveAs

Gatling's `check` mechanism does double duty. It validates a response, and it can also save a piece of that response into the session for later use. Here is a login step that does both.

```java
ScenarioBuilder login = scenario("Login")
    .exec(
        http("Login")
            .post("/auth/login")
            .body(StringBody("{\"username\": \"testuser\", \"password\": \"Password123\"}"))
            .check(status().is(200))
            .check(jsonPath("$.token").saveAs("authToken"))
    );
```

`jsonPath("$.token")` pulls the `token` field out of a JSON response body, and `.saveAs("authToken")` stores it in the session under that name. From this point on in the scenario, `#{authToken}` refers to that value anywhere it is needed.

## Using the Token in Later Requests

Once the token is in the session, attaching it to subsequent requests is just a matter of referencing it in a header.

```java
ScenarioBuilder browseAsAuthenticatedUser = scenario("Authenticated Browse")
    .exec(
        http("Login")
            .post("/auth/login")
            .body(StringBody("{\"username\": \"testuser\", \"password\": \"Password123\"}"))
            .check(status().is(200))
            .check(jsonPath("$.token").saveAs("authToken"))
    )
    .exec(
        http("Get Profile")
            .get("/account/profile")
            .header("Authorization", "Bearer #{authToken}")
            .check(status().is(200))
    )
    .exec(
        http("Get Order History")
            .get("/account/orders")
            .header("Authorization", "Bearer #{authToken}")
            .check(status().is(200))
    );
```

Every request after login carries the token automatically, because it lives in the virtual user's session for the rest of the scenario run.

## Sharing an Authorization Header Across a Whole Scenario

Repeating `.header("Authorization", "Bearer #{authToken}")` on every single request works, but it gets repetitive fast, and it is easy to forget on a new request as the scenario grows. A cleaner approach sets it once at the protocol level, since the protocol builder also supports common headers.

```java
HttpProtocolBuilder authenticatedProtocol = http
    .baseUrl("https://api.example.com")
    .acceptHeader("application/json")
    .authorizationHeader("Bearer #{authToken}");
```

As long as `authToken` exists in the session by the time a request under this protocol fires, every request picks up the header automatically, with no need to repeat it anywhere in the scenario itself.

## Extracting Values With Regex Instead of JSON

Not every application returns a clean JSON response. Some older systems embed a token or a CSRF value inside an HTML page, often in a hidden form field. Gatling's `regex` check handles this the same way `jsonPath` handles JSON.

```java
.exec(
    http("Get Login Page")
        .get("/login")
        .check(status().is(200))
        .check(regex("name=\"csrf_token\" value=\"(.*?)\"").saveAs("csrfToken"))
)
.exec(
    http("Submit Login")
        .post("/login")
        .formParam("username", "testuser")
        .formParam("password", "Password123")
        .formParam("csrf_token", "#{csrfToken}")
        .check(status().is(200))
)
```

This pattern, grab a CSRF token from a form page and submit it back with the login request, is one of the most common correlation problems you will run into against traditional server rendered applications.

## Handling Multi-Step Auth Flows

Some login flows involve more than a single request and response. Think a token exchange, or an initial call that returns a session id you need before you can even submit credentials. The approach does not really change, you just chain more `exec` steps together, each one saving what the next one needs.

```java
ScenarioBuilder multiStepLogin = scenario("Multi Step Login")
    .exec(
        http("Start Session")
            .post("/auth/session")
            .check(status().is(200))
            .check(jsonPath("$.sessionId").saveAs("sessionId"))
    )
    .exec(
        http("Submit Credentials")
            .post("/auth/login")
            .body(StringBody("{\"sessionId\": \"#{sessionId}\", \"username\": \"testuser\", \"password\": \"Password123\"}"))
            .check(status().is(200))
            .check(jsonPath("$.token").saveAs("authToken"))
    );
```

Each step is small and focused, and the session object carries whatever the next step needs, exactly the way a real browser or API client would carry that state through the flow.

## Wrapping Up

Correlation is really just extracting a value now and using it later, and Gatling's `check` and `saveAs` cover the vast majority of cases you will run into, whether that value comes from a clean JSON API or an older HTML form. Getting authentication right at this level unlocks testing almost anything behind a login screen.

Next time, we look at checks in more depth beyond simple status codes and saved values, and talk about what actually makes a load test catch a real failure instead of quietly passing when something is wrong.
