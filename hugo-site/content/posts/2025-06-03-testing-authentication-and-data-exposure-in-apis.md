---
title: "Testing Authentication and Excessive Data Exposure in REST APIs"
date: 2025-06-03T08:25:31+10:00
draft: false
slug: "testing-authentication-and-data-exposure-in-apis"
categories:
  - OWASP
description: "Practical test cases for broken authentication and excessive data exposure, two of the more common OWASP API Security risks in Java REST APIs."
---

In the [previous post](/blog/2025/05/20/testing-broken-object-level-authorization/), we focused entirely on BOLA and object ownership checks. This post covers two more categories from the OWASP API Security Top 10 that tend to show up together in practice, broken authentication and excessive data exposure. Both are less about a single missing check and more about a general habit of trusting the client too much.

## Broken Authentication: Token Expiry

A surprising number of APIs issue tokens correctly but never quite get around to enforcing their expiry properly. Testing this directly is simple once you have a way to generate an expired token.

```java
@Test
void rejectsExpiredToken() {
    String expiredToken = tokenFactory.createExpired("user-a@example.com");

    given()
        .header("Authorization", "Bearer " + expiredToken)
        .when()
        .get("/accounts/1042/transactions")
        .then()
        .statusCode(401);
}
```

It is worth also testing the boundary directly, a token that expired one second ago, rather than only testing a token generated with an expiry far in the past. Clock skew and off by one errors in expiry checks are common enough to be worth a dedicated test.

```java
@Test
void rejectsTokenExpiredOneSecondAgo() {
    String token = tokenFactory.createWithExpiry(Instant.now().minusSeconds(1));

    given()
        .header("Authorization", "Bearer " + token)
        .when()
        .get("/accounts/1042/transactions")
        .then()
        .statusCode(401);
}
```

## Broken Authentication: Credential Stuffing Resistance

Login endpoints are a common target for credential stuffing, automated attempts at large numbers of username and password combinations. Testing full scale resistance is beyond a normal test suite's scope, but confirming basic protections exist is not.

```java
@Test
void locksAccountAfterRepeatedFailedLogins() {
    for (int i = 0; i < 5; i++) {
        given()
            .body(Map.of("email", "user-a@example.com", "password", "wrong-password"))
            .when()
            .post("/auth/login")
            .then()
            .statusCode(401);
    }

    given()
        .body(Map.of("email", "user-a@example.com", "password", "correct-password"))
        .when()
        .post("/auth/login")
        .then()
        .statusCode(423);
}
```

That last assertion, a 423 locked status even with the correct password after repeated failures, confirms the lockout genuinely blocks further attempts rather than only logging them.

## Excessive Data Exposure

This risk shows up when an API returns its full internal object rather than a response shaped specifically for the client, relying on the client application to only display the fields it needs. That habit works fine until a different client, or someone inspecting network traffic directly, sees the entire object.

```java
@GetMapping("/accounts/{accountId}")
public Account getAccount(@PathVariable Long accountId) {
    return accountRepository.findById(accountId).orElseThrow();
}
```

If `Account` is the JPA entity itself, this endpoint likely serializes every column, including things like an internal risk score, a full card number if one is stored, or a hashed password field that should never leave the service at all.

```java
@Test
void accountResponseDoesNotExposeInternalFields() {
    Response response = given()
        .header("Authorization", "Bearer " + tokenForUserA)
        .when()
        .get("/accounts/{id}", userAAccountId);

    response.then()
        .body("$", not(hasKey("passwordHash")))
        .body("$", not(hasKey("internalRiskScore")))
        .body("$", not(hasKey("fullCardNumber")));
}
```

This test does not care what fields the response should contain, only that specific sensitive ones are absent. That framing is deliberate. A response DTO evolves over time, and a test asserting the full shape of a response breaks constantly for unrelated reasons. A test asserting specific sensitive fields never appear stays focused on the actual risk and stays stable through unrelated changes.

## The Underlying Habit to Fix

Both of these issues come from the same root cause, trusting the client to behave responsibly with what the server gives it, or trusting a token's presence without checking its validity thoroughly. The fix in both cases is the same instinct applied consistently. Build a response object deliberately, with only the fields a client actually needs, and validate every property of a token, not just whether it exists and is signed correctly.

The next post moves from targeted test cases like these to automated scanning, wiring OWASP ZAP into a CI pipeline to catch a broader class of these issues without writing a dedicated test for each one individually.
