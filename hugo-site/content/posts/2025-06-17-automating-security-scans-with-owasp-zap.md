---
title: "Automating Security Regression with OWASP ZAP in CI/CD"
date: 2025-06-17T08:25:31+10:00
draft: false
slug: "automating-security-scans-with-owasp-zap"
categories:
  - OWASP
description: "Wiring OWASP ZAP's baseline and full scans into a CI pipeline, and triaging findings without drowning the team in false positives."
---

The last two posts covered specific, hand written test cases for BOLA, broken authentication, and excessive data exposure. Those tests are precise and fast, but they only catch what you thought to write a test for. OWASP ZAP, the Zed Attack Proxy, takes a different approach, actively probing an API for a much broader set of known vulnerability patterns automatically. This post covers wiring it into a pipeline as a regression gate, and just as importantly, how to keep it from becoming noise nobody reads.

## Baseline Scan: Fast and Passive

ZAP's baseline scan is the lighter of its two main modes. It spiders the API, passively observes traffic, and flags obvious issues like missing security headers or verbose error messages, without sending anything that could actually mutate data. That makes it safe to run against a real environment, including one with real data in it.

```bash
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://payment-api-staging.internal \
  -r baseline-report.html
```

This is fast enough to run on every pull request against a staging deployment, and it is a reasonable default gate for a team just getting started with automated security scanning.

## Full Scan: Active and Slower

ZAP's full scan goes further, actively attempting known attack patterns, including basic injection and fuzzing attempts against parameters it discovers. This is meaningfully slower and it does send requests capable of mutating state, which means it belongs against a dedicated test environment, never production, and ideally an environment where a reset between runs is cheap.

```bash
docker run -t owasp/zap2docker-stable zap-full-scan.py \
  -t https://payment-api-test.internal \
  -r full-scan-report.html
```

Running the full scan nightly, rather than on every pull request, is a reasonable middle ground for most teams, since the slower runtime and larger blast radius make it less suited to blocking every single merge.

## Wiring the Baseline Scan Into a Pipeline

```yaml
jobs:
  security-baseline:
    runs-on: ubuntu-latest
    steps:
      - name: Run ZAP baseline scan
        run: |
          docker run -t owasp/zap2docker-stable zap-baseline.py \
            -t ${{ vars.STAGING_URL }} \
            -r baseline-report.html \
            -x baseline-report.xml
      - name: Upload report
        uses: actions/upload-artifact@v4
        with:
          name: zap-baseline-report
          path: baseline-report.html
```

By default, ZAP's baseline scan exits with a warning status rather than failing the build outright, which is a deliberate choice worth keeping in mind rather than fighting.

## Triaging Findings Without Drowning in Noise

This is the part that determines whether a ZAP scan is actually useful or just something everyone learns to ignore. A fresh baseline scan against a real API often returns dozens of findings, many of them low severity or genuinely not applicable, like a missing header on an endpoint that was never meant to be called directly by a browser.

The fix is a maintained rules file that explicitly suppresses findings the team has reviewed and accepted, rather than starting from zero every time.

```yaml
- Missing Anti-clickjacking Header:
    ignore: true
    reason: "API only, no browser rendering, X-Frame-Options not applicable"
- X-Content-Type-Options Header Missing:
    ignore: false
```

```bash
zap-baseline.py -t ${{ vars.STAGING_URL }} -c zap-rules.conf -r baseline-report.html
```

Building this rules file takes an initial time investment, sitting down with the first real scan report and making a genuine call on each finding. After that, new findings on subsequent runs are actually new, which is what makes the scan worth someone's attention going forward instead of a report nobody opens.

## Failing the Build on High Severity Findings

Once the noise is under control, it becomes reasonable to fail the pipeline specifically on high severity findings, while leaving lower severity ones as visible but non-blocking.

```bash
zap-baseline.py -t ${{ vars.STAGING_URL }} -c zap-rules.conf -r report.html
EXIT_CODE=$?
if [ $EXIT_CODE -eq 1 ]; then
  echo "High severity findings detected, failing build"
  exit 1
fi
```

## What ZAP Does Not Replace

ZAP is a strong complement to the hand written tests from the previous two posts, not a substitute for them. It is good at finding generic patterns, missing headers, obvious injection points, common misconfigurations, but it does not understand your specific business logic, which is exactly why the earlier BOLA test, checking one specific user cannot see another specific user's data, still matters and ZAP will not reliably find that particular class of issue on its own.

The final post in this series covers a different layer entirely, dependency and secrets scanning, which catches risks that neither hand written API tests nor a ZAP scan against a running API will ever see.
