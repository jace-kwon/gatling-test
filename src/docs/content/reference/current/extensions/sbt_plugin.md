---
title: "SBT Plugin"
description: "SBT plugin to run Gatling test"
lead: "The SBT plugin allows you to run Gatling test from the command line, without the bundle"
date: 2021-04-20T18:30:56+02:00
lastmod: 2021-04-20T18:30:56+02:00
weight: 9030
---

This SBT plugin integrates Gatling with SBT, allowing to use Gatling as a testing framework.

## Versions

Check out available versions on [Maven Central](https://search.maven.org/artifact/io.gatling/gatling-sbt).

Beware that milestones (M versions) are not documented for OSS users and are only released for [FrontLine](https://gatling.io/gatling-frontline/) customers.

## Setup

In `project/plugins.sbt`, add:

```sbt
addSbtPlugin("io.gatling" % "gatling-sbt" % "MANUALLY_REPLACE_WITH_LATEST_VERSION")
```

You'll also need those two dependencies:

```sbt
"io.gatling.highcharts" % "gatling-charts-highcharts" % "MANUALLY_REPLACE_WITH_LATEST_VERSION" % "test"
"io.gatling"            % "gatling-test-framework"    % "MANUALLY_REPLACE_WITH_LATEST_VERSION" % "test"
```

And then, in your `.scala` build

```sbt
import io.gatling.sbt.GatlingPlugin

lazy val project = Project(...)
  .enablePlugins(GatlingPlugin)
  .settings(libraryDependencies ++= /* Gatling dependencies */)
```

or in your `.sbt` file, for SBT 0.13.6 and later::
```sbt
enablePlugins(GatlingPlugin)

libraryDependencies ++= /* Gatling dependencies */
```
## Demo sample

You can find a [sample project demoing the gatling-sbt-plugin](https://github.com/gatling/gatling-sbt-plugin-demo) in Gatling's Github organization.

You can also use the [Giter8 template]({{< ref "giter8_template" >}}) to bootstrap your project.

## Usage

As with any SBT testing framework, you'll be able to run Gatling simulations using SBT standard `test`, `testOnly`, `testQuick`, etc... tasks.
However, since the SBT Plugin introduces many customizations that we don't want interfering with unit tests, those commands are integrated into custom configurations,
meaning you'll need to prefix them with `gatling` or `gatling-it`, eg. `gatling:test` or `gatling-it:test`.

{{< alert tip >}}
This behavior differs from what was previously possible, eg. calling `test` without prefixing started Gatling simulations.
However, this caused many interferences with other testing libraries and forcing the use of a prefix solves those issues.
{{< /alert >}}

## 'Test' vs 'Integration Tests' configurations

This plugin offers two different custom SBT configurations, named `Gatling` and `GatlingIt`.
They are tied to different sources directories (see next section for more details) and therefore allow to separate your simulations according to your needs, should you desire it.

Ideally:

* Your simulations with low injection profiles, which may serve as functional tests, should live in 'src/test' (the default source directory for the `Gatling` configuration), and run along your unit tests, since they would complete quickly
* Longer, more complex simulations with high injection profiles, should live in 'src/it' (the default source directory for the `GatlingIt` configuration) and be run on a as-needed basis.

Also, since they're tied to separate SBT configurations, your SBT settings can then be customized per configuration.
You can expect a relatively short simulation to run easily with the default JVM settings, but simulations with much higher load can very well require an increase of the max heap memory allowed for example).

{{< alert tip >}}
When using the `GatlingIt` configuration, you must use the `gatling-it` prefix, e.g. `gatling:test` becomes `gatling-it:test`, etc...
{{< /alert >}}

## Default settings

For the `Gatling` configuration :

* By default, Gatling simulations must be in `src/test/scala`, configurable using the `scalaSource in Gatling` setting.
* By default, Gatling reports are written to `target/gatling`, configurable using the `target in Gatling` setting.

For the `GatlingIt` configuration :

* By default, Gatling simulations must be in `src/it/scala`, configurable using the `scalaSource in GatlingIt` setting.
* By default, Gatling reports are written to `target/gatling-it`, configurable using the `target in GatlingIt` setting.

If you override the default settings, you need to reset them on the project, eg:

```sbt
scalaSource in Gatling := sourceDirectory.value / "gatling" / "scala"
lazy val root = (project in file(".")).settings(inConfig(Gatling)(Defaults.testSettings): _*)
```

## Additional tasks

Gatling's SBT plugin also offers four additional tasks:

* `gatling:startRecorder`: starts the Recorder, configured to save recorded simulations to the location specified by `scalaSource in Gatling` (by default, `src/test/scala`).
* `gatling:generateReport`: generates reports for a specified report folder.
* `gatling:lastReport`: opens by the last generated report in your web browser. A simulation name can be specified to open the last report for that simulation.
* `gatling:copyConfigFiles`: copies Gatling's configuration files (gatling.conf & recorder.conf) from the bundle into your project resources if they're missing.
* `gatling:copyLogbackXml`: copies Gatling's default logback.xml.

## Overriding JVM options

Gatling's SBT plugin uses the same default JVM options as the bundle launchers or the Maven plugin, which should be sufficient for most simulations.
However, should you need to tweak them, you can use `overrideDefaultJavaOptions` to only override those default options, without replacing them completely.

E.g., if you want to tweak Xms/Xmx to give more memory to Gatling

```sbt
javaOptions in Gatling := overrideDefaultJavaOptions("-Xms1024m", "-Xmx2048m")
```

## Sources

If you're interested in contributing, you can find the [gatling-sbt plugin sources](https://github.com/gatling/gatling-sbt) on Github.
