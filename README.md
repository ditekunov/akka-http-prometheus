# akka-http-prometheus

[![Build Status](https://travis-ci.org/RustedBones/akka-http-prometheus.svg?branch=master&style=flat)](https://travis-ci.org/RustedBones/akka-http-prometheus)
[![Software License](https://img.shields.io/badge/license-Apache%202-brightgreen.svg?style=flat)](LICENSE)

This library aims to easily expose and collect prometheus formatted metrics in your akka-http server.

For more details about promethus, please see the [official documentation](https://prometheus.io/docs/introduction/overview/)
and the [Java client library](https://github.com/prometheus/client_java)


## Getting Akka Http Prometheus

akka-http-prometheus is deployed to Maven Central. Add it to your `build.sbt`:

```scala
libraryDependencies += "fr.davit" %% "akka-http-prometheus" % "0.0.1"
```

## Quick Start

`akka-http-prometheus` enables you to easily record metrics from an akka-http server into a prometheus registry, 
and expose all the registry's metrics on an HTTP endpoint.


The simplest way to add those capabilities to your server is to import content from the `HttpMetricsRoute` and define the
`HttpMetricsSettings` as implicit. Your route will have the `handleWithMetrics` capability that setup everything:

```scala
import akka.actor.ActorSystem
import akka.http.scaladsl.Http
import akka.http.scaladsl.server.Route
import akka.stream.ActorMaterializer
import fr.davit.akka.http.prometheus.scaladsl.server.HttpMetricsRoute._
import fr.davit.akka.http.prometheus.scaladsl.server.settings.HttpMetricsSettings


implicit val system = ActorSystem("my-system")
implicit val materializer = ActorMaterializer()
// needed for the future flatMap/onComplete in the end
implicit val executionContext = system.dispatcher
implicit val httpMetricsSettings = HttpMetricsSettings()


val route: Route = ...


Http().bindAndHandle(route.handleWithMetrics, "localhost", 8080)
```

### Settings

`HttpMetricsSettings` allows you to parametrize:

- `resourcePath`: path where the prometheus metrics will be exposed for scrapping (default: `metrics`)
- `exports`:  the `HttpMetricsExports` that contains the prometheus registry used to collect the metrics. 
(default: `DefaultHttpMetricsExport` using the prometheus default registry)


### Using a custom prometheus collector

If your application makes use of a custom `CollectorRegistry`, you can use it in the akka-http-prometheus library by
creating a `HttpMetricsExports` and passing it to the directives:

```scala
import  io.prometheus.client.CollectorRegistry
import fr.davit.akka.http.prometheus.scaladsl.server.HttpMetricsExports._
import fr.davit.akka.http.prometheus.scaladsl.server.settings.HttpMetricsSettings

// the custom prometheus registry that you use in your app
val customCollectorRegistry = new CollectorRegistry()

val httpMetricsExports = new HttpMetricsExports {
  override val registry = customCollectorRegistry
}

implicit val httpMetricsSettings = HttpMetricsSettings(exports = httpMetricsExports)
```

## Directives (advanced)

### collectMetrics

The following snippet shows how to use the `collectMetrics` directive to expose metrics collected into the
prometheus default collector registry on the `/metrics` resource path

```scala
import fr.davit.akka.http.prometheus.scaladsl.server.HttpMetricsDirectives._


val route: Route = path("metrics") {
  get {
    collectMetrics()
  }
}
```

### withMetrics

The `withMetrics` directive allows you collect metrics from your akka-http server. You simply have to wrap your API
by the directive:

```scala
import fr.davit.akka.http.prometheus.scaladsl.server.HttpMetricsDirectives._


val route: Route = withMetrics() {
    ... (your api)
}
```