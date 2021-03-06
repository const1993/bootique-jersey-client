<!--
  Licensed to ObjectStyle LLC under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ObjectStyle LLC licenses
  this file to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing,
  software distributed under the License is distributed on an
  "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  KIND, either express or implied.  See the License for the
  specific language governing permissions and limitations
  under the License.
  -->

[![Build Status](https://travis-ci.org/bootique/bootique-jersey-client.svg)](https://travis-ci.org/bootique/bootique-jersey-client)
[![Maven Central](https://img.shields.io/maven-central/v/io.bootique.jersey.client/bootique-jersey-client.svg?colorB=brightgreen)](https://search.maven.org/artifact/io.bootique.jersey.client/bootique-jersey-client/)

# bootique-jersey-client

Integrates JAX-RS-based HTTP client in [Bootique](http://bootique.io) with support for various types of 
server authentication (BASIC, OAuth2, etc.). Allows to configure multiple
client runtime parameters, as well as define server URL endpoints.
Implementation is built on top of [Jersey](https://jersey.java.net/)
and Grizzly connector.
 
## Quick Start

Add the module to your Bootique app:

```xml
<dependency>
	<groupId>io.bootique.jersey.client</groupId>
	<artifactId>bootique-jersey-client</artifactId>
</dependency>
```

Or if you want HTTPS clients with health checks and metrics:

```xml
<dependency>
	<groupId>io.bootique.jersey.client</groupId>
	<artifactId>bootique-jersey-client-instrumented</artifactId>
</dependency>
```

Inject `HttpClientFactory` and create client instances:

```java
@Inject
private HttpClientFactory clientFactory;

public void doSomething() {

    Client client = clientFactory.newClient();
    Response response = client
        .target("https://example.org")
        .request()
        .get();
} 
```

## Configuring Connection Parameters

You can specify a number of runtime parameters for your HTTP clients via
the app ```.yml``` (or any other Bootique configuration mechanism):

```yml
jerseyclient:
  followRedirects: true
  readTimeoutMs: 2000
  connectTimeoutMs: 2000
  asyncThreadPoolSize: 10
```

## Mapping URL Targets

In the example above we injected `HttpClientFactory` (that produced instances
of JAX RS `Client`), and hardcoded the endpoint URL in Java. Instead you
can map multiple URLs in the ```.yml```, assigning each URL a symbolic
name and optionally providing URL-specific runtime parameters:

```yml
jerseyclient:
  targets:
    google:
      url: "https://google.com"
    bootique:
      url: "https://bootique.io"
      followRedirects: false
```
Now you can inject `HttpTargets` and acquire instances of `WebTarget`
by name:
```java
@Inject
private HttpTargets targets;

public void doSomething() {

    Response response = targets.newTarget("bootique").request().get();
}
```
This not only reduces the amount of code, but more importantly allows
to manage your URLs (and their runtime parameters) via configuration.
E.g. you might use a different URL between test and production environments
without changing the code.

## Using BASIC Authentication

If your server endpoint requires BASIC authentication, you can associate
your Clients and WebTargets with a named auth configuration. One or more
named configurations are setup like this:

```yml
jerseyclient:
  auth:
    myauth:
      type: basic
      username: myuser
      password: mypassword
```
When creating a client in the Java code you can reference auth name ("myauth"):
```java
@Inject
private HttpClientFactory clientFactory;

public void doSomething() {

    Client client = clientFactory.newBuilder().auth("myauth").build();
    Response response = client
        .target("https://example.org")
        .request()
        .get();
} 
```
Or you can associate a target with it:
```yml
jerseyclient:
  ...
  targets:
    secret:
      url: "https://example.org"
      auth: myauth
```

## Using OAuth2 Authentication

OAuth2 authentication is very similar to BASIC. In fact they are no different
on the Java end. In YAML the type should be "oauth2", and an extra "tokenUrl"
property is required. Here is an example auth for a Twitter client:

```yml
jerseyclient:
  auth:
    twitter:
      type: oauth2
      tokenUrl: https://api.twitter.com/oauth2/token
      username: sdfjkdferefxfkdsf
      password: Efcdsfdsflkurecdsfj 
```

