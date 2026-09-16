---
title: "Testing for Broken Object Level Authorization (BOLA)"
date: 2025-05-20T08:25:31+10:00
draft: false
slug: "testing-broken-object-level-authorization"
categories:
  - OWASP
description: "Writing practical test cases for Broken Object Level Authorization, the top ranked OWASP API Security risk, against a Java REST API."
---

In the [previous post](/blog/2025/05/06/owasp-api-security-top-10-overview/), we walked through why the OWASP API Security Top 10 deserves attention from a QE team directly, not just a security specialist. Broken Object Level Authorization sits at the top of that list, and for good reason. It is one of the easiest vulnerabilities to introduce by accident, and one of the easiest to test for once you know to look.

## What BOLA Actually Is

BOLA happens when an API checks that a user is authenticated, but does not check that the authenticated user is actually allowed to access the specific object they are requesting. The classic example, a transaction history endpoint.

```
GET /accounts/1042/transactions
Authorization: Bearer <valid token for user A>
```

That request correctly returns user A's transactions. The vulnerability shows up the moment someone tries this instead.

```
GET /accounts/1043/transactions
Authorization: Bearer <same valid token for user A>
```

If the API returns user 1043's transactions instead of a 403, it authenticated the request correctly and then completely failed to check whether user A actually owns account 1043. The token was valid. The authorization was missing.

## Why It Happens So Easily

This bug rarely comes from a careless developer skipping an obvious check. It usually comes from a query that is technically correct and silently trusts the ID in the URL.

```java
@GetMapping("/accounts/{accountId}/transactions")
public List<Transaction> getTransactions(@PathVariable Long accountId) {
    return transactionRepository.findByAccountId(accountId);
}
```

This code does exactly what it looks like it does. It fetches transactions for whatever account ID is in the URL, with no check that the account belongs to the authenticated caller. It compiles, it passes a happy path test with the right account ID, and it ships.

The fix is a single added check, but someone has to think to write it, and more importantly, someone has to write a test that fails without it.

```java
@GetMapping("/accounts/{accountId}/transactions")
public List<Transaction> getTransactions(
        @PathVariable Long accountId,
        @AuthenticationPrincipal AppUser caller) {
    if (!accountService.isOwnedBy(accountId, caller.getId())) {
        throw new AccessDeniedException("Not authorized for this account");
    }
    return transactionRepository.findByAccountId(accountId);
}
```

## Writing the Test

The test itself is straightforward once you frame it correctly. Authenticate as one user, then attempt to access an object belonging to a different user, and assert the request is rejected.

```java
@Test
void userCannotAccessAnotherUsersTransactions() {
    String tokenForUserA = authenticateAs("user-a@example.com");

    given()
        .header("Authorization", "Bearer " + tokenForUserA)
        .when()
        .get("/accounts/{accountId}/transactions", userBAccountId)
        .then()
        .statusCode(403);
}
```

This one test, run against every object level endpoint in the API, catches a surprising number of real issues, precisely because the underlying mistake, trusting an ID from the URL without an ownership check, tends to repeat itself across an API rather than appearing once.

## Making This Systematic Rather Than One-Off

Writing this test for a single endpoint is useful. Writing it as a pattern applied to every endpoint that takes a resource ID is what actually closes the gap. A practical approach is a parameterized test that runs against a list of known object level endpoints, using a second test user's ID for each.

```java
@ParameterizedTest
@ValueSource(strings = {
    "/accounts/{id}/transactions",
    "/accounts/{id}/statements",
    "/accounts/{id}/beneficiaries"
})
void enforcesObjectLevelAuthorization(String endpointTemplate) {
    given()
        .header("Authorization", "Bearer " + tokenForUserA)
        .when()
        .get(endpointTemplate, userBAccountId)
        .then()
        .statusCode(403);
}
```

As new object level endpoints get added to the API, adding one line to this list is a small cost for meaningful, ongoing coverage against the single most common API vulnerability category.

## What This Does Not Catch

This kind of test only covers what you think to list. It does not catch a BOLA vulnerability in an endpoint nobody added to the parameterized list, and it does not catch more subtle variants, like an ID exposed indirectly through a nested object in a response rather than a URL path parameter. That gap is part of why automated scanning, which the OWASP ZAP post later in this series covers, is a useful complement rather than a replacement for tests like these.

The next post covers two more categories from the list together, broken authentication and excessive data exposure, both of which show up constantly in how an API handles tokens and shapes its JSON responses.
