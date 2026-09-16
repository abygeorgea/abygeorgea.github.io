---
title: "Dependency and Secrets Scanning: Closing the Gap ZAP Doesn't Cover"
date: 2025-07-01T08:25:31+10:00
draft: false
slug: "dependency-and-secrets-scanning-in-cicd"
categories:
  - OWASP
description: "Wiring dependency vulnerability scanning and secrets scanning into CI, catching supply chain and credential leak risks that API level testing never sees."
---

In the [previous post](/blog/2025/06/17/automating-security-scans-with-owasp-zap/), ZAP gave us automated coverage against a running API, probing for common vulnerability patterns in requests and responses. There is an entire category of risk that scan never touches, because it does not live in API behavior at all. It lives in what dependencies a service pulls in, and what a commit accidentally includes. This closing post in the OWASP series covers both.

## Why Dependency Scanning Is Its Own Category

Every Spring Boot service pulls in dozens, often hundreds, of transitive dependencies, and any one of them can have a known vulnerability disclosed after you first added it. A service can pass every API test and every ZAP scan cleanly while still shipping a logging library with a critical, publicly known remote code execution flaw. Nothing about API level testing catches this, because the vulnerability is not in your code's behavior, it is in a jar sitting in your classpath.

## OWASP Dependency-Check

OWASP Dependency-Check scans a project's dependencies against the National Vulnerability Database and flags known CVEs by version.

```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>10.0.4</version>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>
    </configuration>
</plugin>
```

```bash
mvn org.owasp:dependency-check-maven:check
```

`failBuildOnCVSS` set to 7 means the build fails on high and critical severity findings, based on the standard CVSS scoring scale, while lower severity issues still show up in the report without blocking the pipeline outright. That threshold is worth tuning to a level the team can realistically act on, since setting it too aggressively on a large existing project can surface an overwhelming first report.

## Wiring Dependency Scanning Into CI

```yaml
jobs:
  dependency-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run OWASP Dependency-Check
        run: mvn org.owasp:dependency-check-maven:check
      - name: Upload report
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: target/dependency-check-report.html
```

Running this on a schedule, nightly for example, in addition to on every pull request, matters here in a way it does not for API tests. A dependency can go from safe to vulnerable overnight, the moment a new CVE is published against a version you already shipped months ago, with no code change on your side at all.

## Secrets Scanning

A different but related risk is a credential accidentally committed to the repository, an API key pasted into a config file during local debugging, or a database password hardcoded while testing something quickly and never removed. Secrets scanning tools like Gitleaks scan commit history and new commits for patterns that look like credentials.

```yaml
jobs:
  secrets-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Running this on every pull request, not just on a schedule, is worth the extra few seconds it adds to the pipeline, since the goal is catching a leaked credential before it merges to main and becomes part of permanent git history, at which point simply deleting the file no longer removes it.

## What to Do When Something Is Found

A dependency finding usually means one of two paths, upgrade to a patched version if one exists, or, if no patched version is available yet, document the accepted risk explicitly with a suppression entry that includes a reason and a reference back to the CVE, rather than silently ignoring the finding.

```xml
<suppress>
    <notes>No patched version available yet, tracked in JIRA-4821, mitigated by network isolation</notes>
    <cve>CVE-2025-XXXXX</cve>
</suppress>
```

A secrets finding is different and more urgent. The credential needs to be rotated immediately, not just removed from the codebase, since the moment it existed in a commit it should be treated as compromised regardless of whether the repository is private.

## Pulling the OWASP Series Together

Across this series we covered BOLA and authentication as targeted, hand written tests catching specific business logic mistakes, ZAP as a broader automated scan against a running API, and now dependency and secrets scanning covering an entirely different layer, what your service depends on and what accidentally ends up in its history. None of these four approaches replaces the others. Together, running continuously in CI rather than as a periodic manual audit, they form a reasonable first layer of security coverage that a QE team can own directly, well before anything needs to escalate to a dedicated security review.
