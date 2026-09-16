---
title: "Inside the OWASP API Security Top 10"
date: 2025-05-06T08:25:31+10:00
draft: false
slug: "owasp-api-security-top-10-overview"
categories:
  - OWASP
description: "Why the OWASP API Security Top 10 exists as a separate list from the general OWASP Top 10, and what it covers at a glance."
---

Most testers have at least heard of the OWASP Top 10, the well known list of the most critical web application security risks. Fewer have spent real time with its sibling list, the OWASP API Security Top 10, which exists because APIs fail in ways that are genuinely different from traditional web applications, and a checklist built for server rendered pages with form submissions misses most of what actually goes wrong in a REST API sitting behind a mobile app or a partner integration.

This post is an overview, framing why the API specific list matters, before the next several posts get hands on with testing for specific risks against a real Spring Boot service.

## Why APIs Are a Different Attack Surface

A traditional web application usually has a browser in front of it, rendering pages, handling cookies, and enforcing some baseline behavior through the browser itself. An API has no such intermediary. It exposes its full surface directly, often to a mobile client, another internal service, or a third party integration, none of which behave like a browser and none of which can be trusted to only send well formed requests.

That difference means risks like broken object level authorization, mass assignment through unexpected fields, and excessive data exposure in a JSON response show up constantly in APIs and comparatively rarely in the exact same form in a browser rendered page.

## The List, Briefly

The current OWASP API Security Top 10 covers ten categories. The ones we will spend the most time on in this series, because they show up constantly in payment and account focused APIs specifically, are:

* **Broken Object Level Authorization**, where an authenticated user can access or modify data belonging to another user simply by changing an ID in the request.
* **Broken Authentication**, covering weak token handling, missing expiry, and credential stuffing resistance.
* **Excessive Data Exposure**, where an API returns more fields than the client actually needs, relying on the client to filter, which leaks data the moment a different client consumes the same endpoint.
* **Lack of Resources and Rate Limiting**, where nothing stops a single client from exhausting a shared resource or brute forcing an endpoint.
* **Security Misconfiguration**, the broad category covering everything from verbose error messages leaking stack traces to permissive CORS settings nobody meant to ship.

## Why This Belongs in a QE Practice, Not Just Security

There is a temptation to treat this list as something a separate security or penetration testing team owns exclusively. That is a mistake for two reasons. First, most of these issues are functional in nature, testable with the same tools and mindset used for any other API test, no specialized security tooling required to at least catch the obvious cases. Second, waiting for a periodic security audit means these issues sit in production, potentially for months, between audits.

A BOLA vulnerability, for example, is really just a missing authorization check on an endpoint that otherwise works perfectly. Any API test suite already exercising that endpoint with valid data is one small addition away from also exercising it with another user's ID, which is exactly the kind of test a QE team is well positioned to own directly.

## What This Series Actually Builds

Rather than staying abstract, the next four posts each take one or two of these categories and build real, runnable tests against a sample payment API. We cover BOLA testing directly, authentication and data exposure together since they tend to overlap in practice, automating a security scan with OWASP ZAP inside a CI pipeline, and closing with dependency and secrets scanning, which catches an entirely different class of risk that API level testing alone does not touch.

The goal by the end is not a security certification. It is a concrete, automated first layer of defense that runs on every pull request, catching the mistakes that are common enough to be worth catching mechanically, before anything reaches a dedicated security review.
