# Ardalo Digital Platform Development Guide
Provides guidelines and FAQ for the development of the Ardalo Digital Platform.

## Table of Contents
* [Guideline structure explained](#guideline-structure-explained)
* [Service Implementation Guide](#service-implementation-guide)
    * [General](#general)
        * [\[Must\] Service is released under the MIT license](#must-service-is-released-under-the-mit-license)
        * [\[Must\] Service is developed in English](#must-service-is-developed-in-english)
    * [Monitoring](#monitoring)
        * [\[Must\] Service exposes Health Check Endpoints](#must-service-exposes-health-check-endpoints)
        * [\[Must\] Service exposes a Prometheus Metrics Endpoint](#must-service-exposes-a-prometheus-metrics-endpoint)
    * [Documentation](#documentation)
        * [\[Must\] Service provides an Open API documentation](#must-service-provides-an-open-api-documentation)
        * [\[Could\] Service contains status badges in README](#could-service-contains-status-badges-in-readme)
    * [Infrastructure](#infrastructure)
        * [\[Must\] Service is Dockerized](#must-service-is-dockerized)
        * [\[Must\] Service is added to README.md of adp-overview](#must-service-is-added-to-readmemd-of-adp-overview)
    * [Tracing](#tracing)
        * [\[Must\] HTTP Requests contain Request-ID](#must-http-requests-contain-request-id)
        * [\[Must\] Log messages contain Request-ID](#must-log-messages-contain-request-id)
    * [QA](#qa)
        * [\[Could\] Service uses SonarCloud static code analysis](#could-service-uses-sonarcloud-static-code-analysis)
* [Service Implementation Checklist](#service-implementation-checklist)
* [FAQ](#faq)
    * [Update Project Dependencies](#update-project-dependencies)
        * [Update Gradle Wrapper](#update-gradle-wrapper)

## Guideline structure explained

Each rule in the Development Guide follows the same structure. It consists of two parts:
* _Details_: Contains in depth explanation of the rule
* _Background_: Provides the answer to the question, why the rule exists

There are two types of rules:
* **\[Must\]**: These rules need to be satisfied except there is a very good reason not to do so
* **\[Could\]**: These rules can be seen as best practices. It is recommended to comply with them, but it is up to you

## Service Implementation Guide

### General

#### [Must] Service is released under the MIT license

**Details:**
* Service contains a `LICENSE` file in the root directory containing the [MIT License](https://opensource.org/licenses/MIT)
* `LICENSE` file contents:
  ```text
  MIT License
  
  Copyright (c) 2022 Torsten Blasche

  Permission is hereby granted, free of charge, to any person obtaining a copy
  of this software and associated documentation files (the "Software"), to deal
  in the Software without restriction, including without limitation the rights
  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
  copies of the Software, and to permit persons to whom the Software is
  furnished to do so, subject to the following conditions:

  The above copyright notice and this permission notice shall be included in all
  copies or substantial portions of the Software.

  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
  SOFTWARE.
  ```

**Background:**
* Services of the Ardalo Digital Platform are developed and released as free software.

#### [Must] Service is developed in English

**Details:**
* The English language is used for source code, documentation, database tables etc.

### Monitoring

#### [Must] Service exposes Health Check Endpoints

**Details:**
* Service exposes a liveness health check endpoint
    * This endpoint indicates whether the service itself is running
    * Endpoint example: `/alive`
    * Returns HTTP Status Code `200` if the service is alive
    * If the service does not respond with HTTP Status Code `200`, it is considered dead and will be restarted
* Service exposes a readiness health check endpoint
    * This endpoint indicates whether the service is able to process requests
    * Endpoint example: `/ready`
    * Returns HTTP Status Code `200` if the service is ready
    * May return a JSON object providing some context about the checked dependencies and its statuses, e.g.
      ```json
      {
        "status": "UP",
        "database": "UP"
      }
      ```
    * If the service does not respond with HTTP Status Code `200`, it will be removed from load balancing
      until ready again
    * The endpoint must only check own dependencies, e.g. its own database. To prevent cascading errors
      and circular dependencies, no external dependencies must be checked

**Background:**
* Health Check Endpoints are essential for an automated infrastructure:
    * If a service is not alive, i.e. does not respond with HTTP Status Code `200` to a liveness request,
      the service can be detected as dead and restarted automatically.
    * If a service is not ready, i.e. does not respond with HTTP Status Code `200` to a readiness request,
      the service will be removed from load balancing and only added again once the readiness endpoint
      returns HTTP Status Code `200`.

#### [Must] Service exposes a Prometheus Metrics Endpoint

**Details:**
* Service exposes an endpoint providing Prometheus metrics

**Background:**
* Prometheus is the default monitoring solution of the Ardalo Digital Platform. Thus every service
  needs to play well with it.

### Documentation

#### [Must] Service provides an Open API documentation

**Details:**
* Service provides an Open API documentation accessible via browser

**Background:**
* With every service providing an Open API documentation it is easy to explore existing APIs using
  a familiar interface. Familiar because this is set as a standard for every service and thus exploring
  different services works quite equal.

#### [Could] Service contains status badges in README

**Details:**
* Service may contain badges in its README file which provide insights of its current status
* Common badges:
    * Build Status (from GitHub Actions), e.g. ![Build Status](https://github.com/ardalo/adp-customer-account-service/workflows/Main/badge.svg)
    * Code Coverage (from SonarCloud), e.g. ![Code Coverage](https://sonarcloud.io/api/project_badges/measure?project=ardalo_adp-customer-account-service&metric=coverage)

**Background:**
* The README file of a Service is the place to go when looking for information. Status badges provide
  a brief overview of interesting service measures like build status or the rough size of the service
  based on the lines of code.

### Infrastructure

#### [Must] Service is dockerized

**Details:**
* Service needs to provide the infrastructure to build a Docker Image of itself, so the service
  can be run as a Docker container

**Background:**
* Kubernetes has been chosen as the runtime environment of the Ardalo Digital Platform. Thus every service
  must provide a Docker Image of itself.

#### [Must] Service is added to README.md of adp-overview

**Details:**
* Service needs to be added to [`README.md`](https://github.com/ardalo/adp-overview/blob/main/README.md)

**Background:**
* The [adp-overview](https://github.com/ardalo/adp-overview) repository provides  a `README.md` file which
  aims to give an overview to the whole platform.

### Tracing

#### [Must] HTTP Requests contain Request-ID

**Details:**
* Outgoing HTTP requests contain a `X-Request-ID` header with a request ID
* If an incoming request contains a request ID (i.e. an `X-Request-ID` request header), this ID has
  to be passed via `X-Request-ID` header to all downstream HTTP requests
* If an incoming request does not contain a request ID (i.e. an `X-Request-ID` request header), a new
  request ID has to be generated and passed via `X-Request-ID` header to all downstream HTTP requests
* Request-IDs have to be treated as strings without a special format
* Example request header: `X-Request-ID: 3858f62230ac3c915f300c664312c63f`
* Example for generating a request ID in `Java`:
  ```java
  String requestId = java.util.UUID.randomUUID().toString().replace("-", "");
  ```
* Example for generating a request ID in `JavaScript`:
  ```js
  import crypto = require('crypto');
  const requestId = crypto.randomBytes(16).toString('hex');
  ```

**Background:**
* Request IDs can be used to correlate log entries (access logs as well as application logs) to easily
  find all log entries belonging to an origin request. This helps to trace requests through the whole platform
  and to find all application logs that were written in the context of this origin request.

#### [Must] Log messages contain Request-ID

**Details:**
* Log messages which belong to an incoming HTTP request contain the request ID of this request
* The request ID can be read from request header `X-Request-ID` or must be self-generated in case
  no request ID has been sent along the incoming request

**Background:**
* Request IDs can be used to correlate log entries (access logs as well as application logs) to easily
  find all log entries belonging to an origin request. This helps to trace requests through the whole platform
  and to find all application logs that were written in the context of this origin request.

### QA

#### [Could] Service uses SonarCloud static code analysis

**Details:**
* Static code analysis can be performed via [SonarCloud](https://sonarcloud.io/organizations/ardalo/projects)
* Code coverage report can be uploaded to [SonarCloud](https://sonarcloud.io/organizations/ardalo/projects) to
  also have code coverage visible

**Background:**
* Static code analysis helps to find issues in the source code, e.g. code smells, bugs or security hotspots.
  Therefore it helps to improve the overall quality of the source code.

## Service Implementation Checklist
This checklist shall help to evaluate whether a service conforms to this guide and additionally show some
best practices which should be adopted by services of the Ardalo Digital Platform.

- General
    - [ ] Service contains a `LICENSE` file containing the MIT License ([ℹ](#must-service-is-released-under-the-mit-license))
- Monitoring
    - [ ] Service exposes a liveness health check endpoint ([ℹ](#must-service-exposes-health-check-endpoints))
    - [ ] Service exposes a readiness health check endpoint ([ℹ](#must-service-exposes-health-check-endpoints))
    - [ ] Service exposes an endpoint providing Prometheus style metrics ([ℹ](#must-service-exposes-a-prometheus-metrics-endpoint))
- Documentation
    - [ ] Service provides an Open API documentation accessible via browser ([ℹ](#must-service-provides-an-open-api-documentation))
    - [ ] (Optional) Service contains a "Build Status"-Badge in its README ([ℹ](#could-service-contains-status-badges-in-readme))
    - [ ] (Optional) Service contains a "Code Coverage"-Badge in its README ([ℹ](#could-service-contains-status-badges-in-readme))
- Infrastructure
    - [ ] Service is dockerized ([ℹ](#must-service-is-dockerized))
    - [ ] Service is added to [`README.md` of `adp-overview`](https://github.com/ardalo/adp-overview/blob/main/README.md)
      ([ℹ](#must-service-is-added-to-readmemd-of-adp-overview))
- Tracing
    - [ ] HTTP Requests contain Request-ID ([ℹ](#must-http-requests-contain-request-id))
    - [ ] Log messages contain Request-ID ([ℹ](#must-log-messages-contain-request-id))
- QA
    - [ ] (Optional) Service uses SonarCloud static code analysis ([ℹ](#could-service-uses-sonarcloud-static-code-analysis))

## FAQ

### Update Project Dependencies

#### Update Gradle Wrapper

1. Find current Gradle version: https://gradle.org/releases/
2. Check release notes for breaking changes, new features etc.
3. Run Gradle wrapper update command:
   ```console
   $ ./gradlew wrapper --gradle-version=<gradle version, e.g. 7.6>
   ```
