# myRepo
myRepo

sbt stuff:

```

https://www.scala-sbt.org/0.13/docs/sbt-reference.pdf - page 78

build.sbt

_______________________________

import sbt._
import Process._
import Keys._
import Dependencies._

// https://www.scala-sbt.org/0.13/docs/sbt-reference.pdf
// Page 78 - Custom settings and tasks

resolvers += Resolver.sonatypeRepo("public")

// lazy val hello = taskKey[Unit]("An example task")

/*
If you’re using an auto plugin that requires explicit enablement, then you have
to add the following to your build.sbt:
lazy val util = (project in file("util"))
.enablePlugins(FooPlugin, BarPlugin)
.settings(
name := "hello-util"
)

Projects can also exclude plugins using the disablePlugins method. For example,
if we wish to remove the IvyPlugin settings from util, we modify our
build.sbt as follows:
lazy val util = (project in file("util"))
.enablePlugins(FooPlugin, BarPlugin)
.disablePlugins(plugins.IvyPlugin)
.settings(
name := "hello-util"
)

If you’re curious which auto plugins are enabled for a given project, just run the
plugins command on the sbt console.

[IJ]> plugins
In file:/Users/john-michaelreed/Downloads/NewDownloads/sbt-0.13/lesson/hello/
 sbt.plugins.IvyPlugin: enabled in root, core, util  // IvyPlugin: Provides the mechanisms to publish/resolve modules
 sbt.plugins.JvmPlugin: enabled in root, core, util  // JvmPlugin: Provides the mechanisms to compile/test/run/package Java/Scala projects.
 sbt.plugins.CorePlugin: enabled in root, core, util  // CorePlugin: Provides the core parallelism controls for tasks.
 sbt.plugins.JUnitXmlReportPlugin: enabled in root, core, util
 sbt.plugins.Giter8TemplatePlugin: enabled in root, core, util
 org.jetbrains.sbt.StructurePlugin: enabled in root, core, util
 org.jetbrains.sbt.IdeaShellPlugin: enabled in root, core, util
[IJ]>
 */

lazy val hello = taskKey[Int]("An example task")

lazy val fourtyTwo = taskKey[Int]("The integer fourty two")

// lazy val fourtyTwo = settingKey[Int]("The integer fourty two")

val derby = "org.apache.derby" % "derby" % "10.4.1.3"

val foo: Scoped.DefinableSetting[Seq[ModuleID]] = Keys.libraryDependencies

val fooooo = taskKey[String]("Options for the Scala compiler.")
// val update = taskKey[UpdateReport]("Resolves and optionally retrieves dependencies, producing a report.")
// val clean = taskKey[Unit]("Deletes files produced by the build, such as generated sources, compiled classes, and task caches.")
// val scalacOptions = taskKey[Seq[String]]("Options for the Scala compiler.")
/*
Here’s a realistic example. This rewires scalaSource in Compile key to a
different directory only when scalaBinaryVersion is "2.11".
scalaSource in Compile := {
val old = (scalaSource in Compile).value
scalaBinaryVersion.value match {
case "2.11" => baseDirectory.value / "src-2.11" / "main" / "scala"
case _ => old
}
}
 */
/*
lazy val projB = (project in file("b"))
.settings(
name := "abc-" + organization.value,
organization := "org.tempuri"
)
 */

// lazy val util = (project in file("util"))
// lazy val core = project

lazy val commonSettings = Seq(
  scalaVersion := "2.12.2"
)

/*
lazy val core = (project in file("core"))
  .settings(
    commonSettings
    // other settings
  )
*/

lazy val util = (project in file("util"))
  .settings(
    commonSettings
    // other settings
  )

lazy val core = project.dependsOn(util) // Now code in core can use classes from util.

organization in ThisBuild := "com.example"

lazy val root = (project in file(".")).
  settings(
    inThisBuild(List(
      fourtyTwo in (Compile, packageBin) := 42,
      hello := { println("Hello!"); 42 },
      scalacOptions := List("-encoding", "utf8", "-Xfatal-warnings", "-deprecation", "-unchecked"),
      scalacOptions := {
        val old = scalacOptions.value
        scalaBinaryVersion.value match {
          case "2.12" => old
          case _ => old filterNot (Set("-Xfatal-warnings", "-deprecation").apply)
        }
      },
      fooooo := {
        val ur = hello.value // hello task happens-before fooooo
        // ---- fooooo begins here ----
        ur.toString
      },
      organization := "com.example",
      scalaVersion := "2.12.5",
      version :=("0.1.0-SNAPSHOT")
    )),
    name := "abc-" + organization.value,
    organization := "org.tempuri",
    libraryDependencies += scalaTest % Test,
    libraryDependencies += derby
  )
/*

val derby = "org.apache.derby" % "derby" % "10.4.1.3"
lazy val commonSettings = Seq(
organization := "com.example",
version := "0.1.0-SNAPSHOT",
scalaVersion := "2.12.2"
)
lazy val root = (project in file("."))
.settings(
commonSettings,
name := "Hello",
libraryDependencies += derby
)

 */
_______________________________

localBuild.sbt

// Any .sbt files in a project (localBuild.sbt) will be merged with the build definition
// for the entire build (build.sbt), but scoped to the project it is in.

version := "0.6.6"

_______________________________

Build.scala (in project folder)
/**
  * In order to share code between .sbt files, define one or more Scala files
  * in the project/ directory of the build root.
  */
class Build {

}

______________
hello/project/site.sbt
addSbtPlugin("com.typesafe.sbt" % "sbt-site" % "0.7.0")

```

Sample:

```
https://github.com/lihaoyi/scala.rx/blob/c5b1389f6a1dc809996f786391985e2221a219ed/build.sbt - c5b1389  on Nov 6, 2016

crossScalaVersions := Seq("2.10.6", "2.11.8", "2.12.0")

val scalarx = crossProject.settings(
  organization := "com.lihaoyi",
  name := "scalarx",
  scalaVersion := "2.12.0",
  version := "0.3.2",

  libraryDependencies ++= Seq(
    "org.scala-lang" % "scala-reflect" % scalaVersion.value % "provided",
    "org.scala-lang" % "scala-compiler" % scalaVersion.value % "provided",
    "com.lihaoyi" %%% "utest" % "0.4.4" % "test",
    "com.lihaoyi" %% "acyclic" % "0.1.5" % "provided"
  ) ++ (
    CrossVersion.partialVersion(scalaVersion.value) match {
      // if scala 2.11+ is used, quasiquotes are merged into scala-reflect
      case Some((2, scalaMajor)) if scalaMajor >= 11 =>
        Nil
      // in Scala 2.10, quasiquotes are provided by macro paradise
      case Some((2, 10)) =>
        Seq(
          compilerPlugin("org.scalamacros" % "paradise" % "2.0.1" cross CrossVersion.full),
          "org.scalamacros" %% "quasiquotes" % "2.0.0" cross CrossVersion.binary)
    }
  ),
  addCompilerPlugin("com.lihaoyi" %% "acyclic" % "0.1.5"),
  testFrameworks += new TestFramework("utest.runner.Framework"),
  autoCompilerPlugins := true,
  // Sonatype

  publishTo := Some("releases"  at "https://oss.sonatype.org/service/local/staging/deploy/maven2"),

  pomExtra :=
    <url>https://github.com/lihaoyi/scalatags</url>
      <licenses>
        <license>
          <name>MIT license</name>
          <url>http://www.opensource.org/licenses/mit-license.php</url>
        </license>
      </licenses>
      <scm>
        <url>git://github.com/lihaoyi/scalatags.git</url>
        <connection>scm:git://github.com/lihaoyi/scalatags.git</connection>
      </scm>
      <developers>
        <developer>
          <id>lihaoyi</id>
          <name>Li Haoyi</name>
          <url>https://github.com/lihaoyi</url>
        </developer>
      </developers>
).jsSettings(
  libraryDependencies ++= Seq(
    "org.scala-js" %%% "scalajs-dom" % "0.9.1" % "provided"
  ),
  scalaJSStage in Test := FullOptStage,
  scalacOptions ++= (if (isSnapshot.value) Seq.empty else Seq({
    val a = baseDirectory.value.toURI.toString.replaceFirst("[^/]+/?$", "")
    val g = "https://raw.githubusercontent.com/lihaoyi/scala.rx"
    s"-P:scalajs:mapSourceURI:$a->$g/v${version.value}/"
  }))
).jvmSettings(
  libraryDependencies ++= Seq(
    if (scalaVersion.value.startsWith("2.10."))
      "com.typesafe.akka" %% "akka-actor" % "2.3.15" % "provided"
    else
      "com.typesafe.akka" %% "akka-actor" % "2.4.12" % "provided")
)

lazy val js = scalarx.js

lazy val jvm = scalarx.jvm

```

Examples:

```
https://github.com/akka/akka/blob/177ac666da323e8af07a35159f35c308e873b585/project/build.properties on Jun 29, 2017
sbt.version=0.13.15
```

Plugins:
```
https://github.com/akka/akka/blob/177ac666da323e8af07a35159f35c308e873b585/project/plugins.sbt

resolvers += Classpaths.typesafeResolver

// need this to resolve http://jcenter.bintray.com/org/jenkins-ci/jenkins/1.26/
// which is used by plugin "org.kohsuke" % "github-api" % "1.68"
resolvers += "Bintray Jcenter" at "https://jcenter.bintray.com/"

// these comment markers are for including code into the docs
//#sbt-multi-jvm
addSbtPlugin("com.typesafe.sbt" % "sbt-multi-jvm" % "0.3.8")
//#sbt-multi-jvm

addSbtPlugin("org.scalariform" % "sbt-scalariform" % "1.6.0")

addSbtPlugin("com.typesafe.sbt" % "sbt-site" % "0.7.1")

addSbtPlugin("com.typesafe.sbt" % "sbt-osgi" % "0.9.1")

addSbtPlugin("com.typesafe" % "sbt-mima-plugin" % "0.1.14")

addSbtPlugin("com.jsuereth" % "sbt-pgp" % "1.0.0")

addSbtPlugin("com.eed3si9n" % "sbt-unidoc" % "0.3.3")

addSbtPlugin("com.thoughtworks.sbt-api-mappings" % "sbt-api-mappings" % "1.0.0")

addSbtPlugin("pl.project13.scala" % "sbt-jmh" % "0.2.26")

addSbtPlugin("pl.project13.sbt" % "sbt-jol" % "0.1.1")

addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.0.0-RC1")

// for advanced PR validation features
addSbtPlugin("net.virtual-void" % "sbt-dependency-graph" % "0.8.2")

libraryDependencies += "org.kohsuke" % "github-api" % "1.68"

addSbtPlugin("io.spray" % "sbt-boilerplate" % "0.6.0")

addSbtPlugin("com.timushev.sbt" % "sbt-updates" % "0.1.10")

addSbtPlugin("com.lightbend.akka" % "sbt-paradox-akka" % "0.3")

addSbtPlugin("com.lightbend" % "sbt-whitesource" % "0.1.2")

addSbtPlugin("com.typesafe.sbt" % "sbt-git" % "0.9.3")

```

```
https://github.com/akka/akka/blob/177ac666da323e8af07a35159f35c308e873b585/build.sbt on Jun 29, 2017

enablePlugins(akka.UnidocRoot, akka.TimeStampede, akka.UnidocWithPrValidation)
disablePlugins(MimaPlugin)
import com.typesafe.sbt.SbtMultiJvm.MultiJvmKeys.MultiJvm
import com.typesafe.tools.mima.plugin.MimaPlugin
import akka.AkkaBuild._

initialize := {
  // Load system properties from a file to make configuration from Jenkins easier
  loadSystemProperties("project/akka-build.properties")
  initialize.value
}

akka.AkkaBuild.buildSettings
shellPrompt := { s => Project.extract(s).currentProject.id + " > " }
resolverSettings

lazy val aggregatedProjects: Seq[ProjectReference] = Seq(
  actor, actorTests,
  agent,
  benchJmh,
  camel,
  cluster, clusterMetrics, clusterSharding, clusterTools,
  contrib,
  distributedData,
  docs,
  multiNodeTestkit,
  osgi,
  persistence, persistenceQuery, persistenceShared, persistenceTck,
  protobuf,
  remote, remoteTests,
  slf4j,
  stream, streamTestkit, streamTests, streamTestsTck,
  testkit,
  typed, typedTests, typedTestkit
)

lazy val root = Project(
  id = "akka",
  base = file(".")
).aggregate(aggregatedProjects: _*)
 .settings(rootSettings: _*)
 .settings(unidocRootIgnoreProjects := Seq(remoteTests, benchJmh, protobuf, akkaScalaNightly, docs))

lazy val actor = akkaModule("akka-actor")

lazy val actorTests = akkaModule("akka-actor-tests")
  .dependsOn(testkit % "compile->compile;test->test")

lazy val agent = akkaModule("akka-agent")
  .dependsOn(actor, testkit % "test->test")

lazy val akkaScalaNightly = akkaModule("akka-scala-nightly")
  // remove dependencies that we have to build ourselves (Scala STM)
  .aggregate(aggregatedProjects diff List[ProjectReference](agent, docs): _*)
  .disablePlugins(ValidatePullRequest, MimaPlugin)

lazy val benchJmh = akkaModule("akka-bench-jmh")
  .dependsOn(
    Seq(
      actor,
      stream, streamTests,
      persistence, distributedData,
      testkit
    ).map(_ % "compile->compile;compile->test;provided->provided"): _*
  ).disablePlugins(ValidatePullRequest)

lazy val camel = akkaModule("akka-camel")
  .dependsOn(actor, slf4j, testkit % "test->test")

lazy val cluster = akkaModule("akka-cluster")
  .dependsOn(remote, remoteTests % "test->test" , testkit % "test->test")
  .configs(MultiJvm)

lazy val clusterMetrics = akkaModule("akka-cluster-metrics")
  .dependsOn(cluster % "compile->compile;test->test;multi-jvm->multi-jvm", slf4j % "test->compile")
  .configs(MultiJvm)

lazy val clusterSharding = akkaModule("akka-cluster-sharding")
  // TODO akka-persistence dependency should be provided in pom.xml artifact.
  //      If I only use "provided" here it works, but then we can't run tests.
  //      Scope "test" is alright in the pom.xml, but would have been nicer with
  //      provided.
  .dependsOn(
  cluster % "compile->compile;test->test;multi-jvm->multi-jvm",
  distributedData,
  persistence % "compile->compile;test->provided",
  clusterTools)
  .configs(MultiJvm)

lazy val clusterTools = akkaModule("akka-cluster-tools")
  .dependsOn(cluster % "compile->compile;test->test;multi-jvm->multi-jvm")
  .configs(MultiJvm)

lazy val contrib = akkaModule("akka-contrib")
  .dependsOn(remote, remoteTests % "test->test", cluster, clusterTools, persistence % "compile->compile;test->provided")
  .configs(MultiJvm)

lazy val distributedData = akkaModule("akka-distributed-data")
  .dependsOn(cluster % "compile->compile;test->test;multi-jvm->multi-jvm")
  .configs(MultiJvm)

lazy val docs = akkaModule("akka-docs")
  .dependsOn(
    actor, cluster, clusterMetrics, slf4j, agent, camel, osgi, persistenceTck, persistenceQuery, distributedData, stream,
    clusterTools % "compile->compile;test->test",
    testkit % "compile->compile;test->test",
    remote % "compile->compile;test->test",
    persistence % "compile->compile;provided->provided;test->test",
    typed % "compile->compile;test->test",
    typedTests % "compile->compile;test->test",
    streamTestkit % "compile->compile;test->test"
  )

lazy val multiNodeTestkit = akkaModule("akka-multi-node-testkit")
  .dependsOn(remote, testkit)

lazy val osgi = akkaModule("akka-osgi")
  .dependsOn(actor)

lazy val persistence = akkaModule("akka-persistence")
  .dependsOn(actor, testkit % "test->test", protobuf)

lazy val persistenceQuery = akkaModule("akka-persistence-query")
  .dependsOn(
    stream,
    persistence % "compile->compile;provided->provided;test->test",
    streamTestkit % "test")

lazy val persistenceShared = akkaModule("akka-persistence-shared")
  .dependsOn(persistence % "test->test", testkit % "test->test", remote % "test", protobuf)

lazy val persistenceTck = akkaModule("akka-persistence-tck")
  .dependsOn(persistence % "compile->compile;provided->provided;test->test", testkit % "compile->compile;test->test")

lazy val protobuf = akkaModule("akka-protobuf")

lazy val remote = akkaModule("akka-remote")
  .dependsOn(actor, stream, actorTests % "test->test", testkit % "test->test", streamTestkit % "test", protobuf)

lazy val remoteTests = akkaModule("akka-remote-tests")
  .dependsOn(actorTests % "test->test", remote % "test->test", streamTestkit % "test", multiNodeTestkit)
  .configs(MultiJvm)

lazy val slf4j = akkaModule("akka-slf4j")
  .dependsOn(actor, testkit % "test->test")

lazy val stream = akkaModule("akka-stream")
  .dependsOn(actor)

lazy val streamTestkit = akkaModule("akka-stream-testkit")
  .dependsOn(stream, testkit % "compile->compile;test->test")

lazy val streamTests = akkaModule("akka-stream-tests")
  .dependsOn(streamTestkit % "test->test", stream)

lazy val streamTestsTck = akkaModule("akka-stream-tests-tck")
  .dependsOn(streamTestkit % "test->test", stream)

lazy val typed = akkaModule("akka-typed")
  .dependsOn(testkit % "compile->compile;test->test")

lazy val typedTests = akkaModule("akka-typed-tests")
  .dependsOn(typed, typedTestkit % "compile->compile;test->test")

lazy val typedTestkit = akkaModule("akka-typed-testkit")
  .dependsOn(typed, testkit % "compile->compile;test->test")

lazy val testkit = akkaModule("akka-testkit")
  .dependsOn(actor)




def akkaModule(name: String): Project =
  Project(id = name, base = file(name))
    .settings(akka.AkkaBuild.buildSettings)
```

Local build:

```
https://github.com/akka/akka/blob/177ac666da323e8af07a35159f35c308e873b585/akka-actor/build.sbt

import akka.{ AkkaBuild, Formatting, OSGi, Dependencies, Version }

AkkaBuild.defaultSettings
Formatting.formatSettings
OSGi.actor
Dependencies.actor
Version.versionSettings
unmanagedSourceDirectories in Compile += {
  val ver = scalaVersion.value.take(4)
  (scalaSource in Compile).value.getParentFile / s"scala-$ver"
}

enablePlugins(spray.boilerplate.BoilerplatePlugin)
```

Akka Build:

```

https://github.com/akka/akka/blob/177ac666da323e8af07a35159f35c308e873b585/project/AkkaBuild.scala

/**
 * Copyright (C) 2009-2017 Lightbend Inc. <http://www.lightbend.com>
 */

package akka

import java.io.{FileInputStream, InputStreamReader}
import java.util.Properties

import akka.TestExtras.JUnitFileReporting
import com.typesafe.sbt.pgp.PgpKeys.publishSigned
import sbt.Keys._
import sbt.TestLogger.wrap
import sbt._

object AkkaBuild {

  val enableMiMa = true

  val parallelExecutionByDefault = false // TODO: enable this once we're sure it does not break things

  lazy val buildSettings = Dependencies.Versions ++ Seq(
    organization        := "com.typesafe.akka",
    version             := "2.5-SNAPSHOT"
  )

  lazy val rootSettings = parentSettings ++ Release.settings ++
    UnidocRoot.akkaSettings ++
    Protobuf.settings ++ Seq(
      parallelExecution in GlobalScope := System.getProperty("akka.parallelExecution", parallelExecutionByDefault.toString).toBoolean
    )

  val dontPublishSettings = Seq(
    publishSigned := (),
    publish := (),
    publishArtifact in Compile := false
  )

  val dontPublishDocsSettings = Seq(
    sources in doc in Compile := List()
  )


  lazy val parentSettings = Seq(
    publishArtifact := false
  ) ++ dontPublishSettings

  lazy val mayChangeSettings = Seq(
    description := """|This module of Akka is marked as
                      |'may change', which means that it is in early
                      |access mode, which also means that it is not covered
                      |by commercial support. An module marked 'may change' doesn't
                      |have to obey the rule of staying binary compatible
                      |between minor releases. Breaking API changes may be
                      |introduced in minor releases without notice as we
                      |refine and simplify based on your feedback. Additionally
                      |such a module may be dropped in major releases
                      |without prior deprecation.
                      |""".stripMargin
  )

  val (mavenLocalResolver, mavenLocalResolverSettings) =
    System.getProperty("akka.build.M2Dir") match {
      case null => (Resolver.mavenLocal, Seq.empty)
      case path =>
        // Maven resolver settings
        val resolver = Resolver.file("user-publish-m2-local", new File(path))
        (resolver, Seq(
          otherResolvers := resolver:: publishTo.value.toList,
          publishM2Configuration := Classpaths.publishConfig(packagedArtifacts.value, None, resolverName = resolver.name, checksums = checksums.in(publishM2).value, logging = ivyLoggingLevel.value, overwrite = true)
        ))
    }

  lazy val resolverSettings = {
    // should we be allowed to use artifacts published to the local maven repository
    if(System.getProperty("akka.build.useLocalMavenResolver", "false").toBoolean)
      Seq(resolvers += mavenLocalResolver)
    else Seq.empty
  } ++ {
    // should we be allowed to use artifacts from sonatype snapshots
    if(System.getProperty("akka.build.useSnapshotSonatypeResolver", "false").toBoolean)
      Seq(resolvers += Resolver.sonatypeRepo("snapshots"))
    else Seq.empty
  } ++ Seq(
    pomIncludeRepository := (_ => false) // do not leak internal repositories during staging
  )

  private def allWarnings: Boolean = System.getProperty("akka.allwarnings", "false").toBoolean

  lazy val defaultSettings = resolverSettings ++
    TestExtras.Filter.settings ++
    Protobuf.settings ++ Seq(
    // compile options
    scalacOptions in Compile ++= Seq("-encoding", "UTF-8", "-target:jvm-1.8", "-feature", "-unchecked", "-Xlog-reflective-calls", "-Xlint"),
    scalacOptions in Compile ++= (if (allWarnings) Seq("-deprecation") else Nil),
    scalacOptions in Test := (scalacOptions in Test).value.filterNot(opt =>
      opt == "-Xlog-reflective-calls" || opt.contains("genjavadoc")),
    // -XDignore.symbol.file suppresses sun.misc.Unsafe warnings
    javacOptions in compile ++= Seq("-encoding", "UTF-8", "-source", "1.8", "-target", "1.8", "-Xlint:unchecked", "-XDignore.symbol.file"),
    javacOptions in compile ++= (if (allWarnings) Seq("-Xlint:deprecation") else Nil),
    javacOptions in doc ++= Seq(),
    incOptions := incOptions.value.withNameHashing(true),

    crossVersion := CrossVersion.binary,

    ivyLoggingLevel in ThisBuild := UpdateLogging.Quiet,

    licenses := Seq(("Apache License, Version 2.0", url("http://www.apache.org/licenses/LICENSE-2.0"))),
    homepage := Some(url("http://akka.io/")),

    apiURL := Some(url(s"http://doc.akka.io/api/akka/${version.value}")),

    initialCommands :=
      """|import language.postfixOps
         |import akka.actor._
         |import ActorDSL._
         |import scala.concurrent._
         |import com.typesafe.config.ConfigFactory
         |import scala.concurrent.duration._
         |import akka.util.Timeout
         |var config = ConfigFactory.parseString("akka.stdout-loglevel=INFO,akka.loglevel=DEBUG,pinned{type=PinnedDispatcher,executor=thread-pool-executor,throughput=1000}")
         |var remoteConfig = ConfigFactory.parseString("akka.remote.netty{port=0,use-dispatcher-for-io=akka.actor.default-dispatcher,execution-pool-size=0},akka.actor.provider=remote").withFallback(config)
         |var system: ActorSystem = null
         |implicit def _system = system
         |def startSystem(remoting: Boolean = false) { system = ActorSystem("repl", if(remoting) remoteConfig else config); println("don’t forget to system.terminate()!") }
         |implicit def ec = system.dispatcher
         |implicit val timeout = Timeout(5 seconds)
         |""".stripMargin,

    /**
     * Test settings
     */

    parallelExecution in Test := System.getProperty("akka.parallelExecution", parallelExecutionByDefault.toString).toBoolean,
    logBuffered in Test := System.getProperty("akka.logBufferedTests", "false").toBoolean,

    // show full stack traces and test case durations
    testOptions in Test += Tests.Argument("-oDF"),

    // don't save test output to a file, workaround for https://github.com/sbt/sbt/issues/937
    testListeners in (Test, test) := {
      val logger = streams.value.log

      def contentLogger(log: sbt.Logger, buffered: Boolean): ContentLogger = {
        val blog = new BufferedLogger(FullLogger(log))
        if (buffered) blog.record()
        new ContentLogger(wrap(blog), () => blog.stopQuietly())
      }

      val logTest = {_: TestDefinition => streams.value.log }
      val buffered = logBuffered.value
      Seq(new TestLogger(new TestLogging(wrap(logger), tdef => contentLogger(logTest(tdef), buffered))))
    },

    // -v Log "test run started" / "test started" / "test run finished" events on log level "info" instead of "debug".
    // -a Show stack traces and exception class name for AssertionErrors.
    testOptions += Tests.Argument(TestFrameworks.JUnit, "-v", "-a")
  ) ++
    mavenLocalResolverSettings ++
    JUnitFileReporting.settings ++
    docLintingSettings

  lazy val docLintingSettings = Seq(
     javacOptions in compile ++= Seq("-Xdoclint:none"),
     javacOptions in test ++= Seq("-Xdoclint:none"),
     javacOptions in doc ++= Seq("-Xdoclint:none")
   )

  def loadSystemProperties(fileName: String): Unit = {
    import scala.collection.JavaConverters._
    val file = new File(fileName)
    if (file.exists()) {
      println("Loading system properties from file `" + fileName + "`")
      val in = new InputStreamReader(new FileInputStream(file), "UTF-8")
      val props = new Properties
      props.load(in)
      in.close()
      sys.props ++ props.asScala
    }
  }

  def majorMinor(version: String): Option[String] ="""\d+\.\d+""".r.findFirstIn(version)
}
```

Using Build.scala:

```

- The recommended approach is to define most settings in a multi-project
build.sbt file, and using project/*.scala files for task implementations or
to share values, such as keys. The use of .scala files also depends on how
comfortable you or your team are with Scala.

- put most of your configuration in build.sbt, but use .scala build definition
files for defining classes and larger task implementations.

```

Using a Dependencies.scala file:

```
Tracking dependencies in one place
One way of using the fact that .scala files under project becomes part of the
build definition is to create project/Dependencies.scala to track dependencies
in one place.

import sbt._
object Dependencies {
// Versions
lazy val akkaVersion = "2.3.8"
// Libraries
85
val akkaActor = "com.typesafe.akka" %% "akka-actor" % akkaVersion
val akkaCluster = "com.typesafe.akka" %% "akka-cluster" % akkaVersion
val specs2core = "org.specs2" %% "specs2-core" % "2.4.17"
// Projects
val backendDeps =
Seq(akkaActor, specs2core % Test)
}

The Dependencies object will be available in build.sbt. To use the vals under
it easier, import Dependencies._.

import Dependencies._
lazy val commonSettings = Seq(
version := "0.1.0",
scalaVersion := "2.12.2"
)
lazy val backend = (project in file("backend"))
.settings(
commonSettings,
libraryDependencies ++= backendDeps
)

This technique is useful when you have a multi-project build that’s getting large,
and you want to make sure that subprojects to have consistent dependencies.
```

Dependencies:

```
https://github.com/akka/akka/blob/177ac666da323e8af07a35159f35c308e873b585/project/Dependencies.scala

/**
 * Copyright (C) 2016-2017 Lightbend Inc. <http://www.lightbend.com>
 */
package akka

import sbt._
import Keys._

object Dependencies {
  import DependencyHelpers._

  lazy val scalaTestVersion = settingKey[String]("The version of ScalaTest to use.")
  lazy val scalaStmVersion = settingKey[String]("The version of ScalaSTM to use.")
  lazy val scalaCheckVersion = settingKey[String]("The version of ScalaCheck to use.")
  lazy val java8CompatVersion = settingKey[String]("The version of scala-java8-compat to use.")
  val junitVersion = "4.12"
  val sslConfigVersion = "0.2.1"
  val slf4jVersion = "1.7.23"
  val scalaXmlVersion = "1.0.6"
  val aeronVersion = "1.2.5"

  val Versions = Seq(
    crossScalaVersions := Seq("2.11.11", "2.12.2"),
    scalaVersion := System.getProperty("akka.build.scalaVersion", crossScalaVersions.value.head),
    scalaStmVersion := sys.props.get("akka.build.scalaStmVersion").getOrElse("0.8"),
    scalaCheckVersion := sys.props.get("akka.build.scalaCheckVersion").getOrElse(
      CrossVersion.partialVersion(scalaVersion.value) match {
        case Some((2, n)) if n >= 12 => "1.13.4" // does not work for 2.11
        case _                       => "1.13.2"
      }
    ),
    scalaTestVersion := "3.0.0",
    java8CompatVersion := {
      CrossVersion.partialVersion(scalaVersion.value) match {
        case Some((2, n)) if n >= 12 => "0.8.0"
        case _                       => "0.7.0"
      }
    }
  )

  object Compile {
    // Compile

    val camelCore     = "org.apache.camel"            % "camel-core"                   % "2.15.6" exclude("org.slf4j", "slf4j-api") // ApacheV2

    // when updating config version, update links ActorSystem ScalaDoc to link to the updated version
    val config        = "com.typesafe"                % "config"                       % "1.3.1"       // ApacheV2
    val netty         = "io.netty"                    % "netty"                        % "3.10.6.Final" // ApacheV2
    val scalaStm      = Def.setting { "org.scala-stm" %% "scala-stm" % scalaStmVersion.value } // Modified BSD (Scala)

    val scalaXml      = "org.scala-lang.modules"      %% "scala-xml"                   % scalaXmlVersion // Scala License
    val scalaReflect  = ScalaVersionDependentModuleID.versioned("org.scala-lang" % "scala-reflect" % _) // Scala License

    val slf4jApi      = "org.slf4j"                   % "slf4j-api"                    % slf4jVersion       // MIT

    // mirrored in OSGi sample https://github.com/akka/akka-samples/tree/master/akka-sample-osgi-dining-hakkers
    val osgiCore      = "org.osgi"                    % "org.osgi.core"                % "4.3.1"       // ApacheV2
    val osgiCompendium= "org.osgi"                    % "org.osgi.compendium"          % "4.3.1"       // ApacheV2

    val sigar         = "org.fusesource"              % "sigar"                        % "1.6.4"       // ApacheV2

    // reactive streams
    val reactiveStreams = "org.reactivestreams"       % "reactive-streams"             % "1.0.0" // CC0

    // ssl-config
    val sslConfigCore = "com.typesafe"                %% "ssl-config-core"             % sslConfigVersion // ApacheV2

    val lmdb          = "org.lmdbjava"                % "lmdbjava"                     % "0.0.5" // ApacheV2, OpenLDAP Public License

    // For akka-http-testkit-java
    val junit       = "junit"                         % "junit"                        % junitVersion  // Common Public License 1.0

    // For Java 8 Conversions
    val java8Compat = Def.setting {"org.scala-lang.modules" %% "scala-java8-compat" % java8CompatVersion.value} // Scala License

    val aeronDriver = "io.aeron"                      % "aeron-driver"                 % aeronVersion       // ApacheV2
    val aeronClient = "io.aeron"                      % "aeron-client"                 % aeronVersion       // ApacheV2

    object Docs {
      val sprayJson   = "io.spray"                   %%  "spray-json"                  % "1.3.3"             % "test"
      val gson        = "com.google.code.gson"        % "gson"                         % "2.8.0"             % "test"
    }

    object Test {
      val commonsMath  = "org.apache.commons"          % "commons-math"                 % "2.2"              % "test" // ApacheV2
      val commonsIo    = "commons-io"                  % "commons-io"                   % "2.5"              % "test" // ApacheV2
      val commonsCodec = "commons-codec"               % "commons-codec"                % "1.10"             % "test" // ApacheV2
      val junit        = "junit"                       % "junit"                        % junitVersion       % "test" // Common Public License 1.0
      val logback      = "ch.qos.logback"              % "logback-classic"              % "1.2.3"            % "test" // EPL 1.0 / LGPL 2.1
      val mockito      = "org.mockito"                 % "mockito-all"                  % "1.10.19"          % "test" // MIT
      // changing the scalatest dependency must be reflected in akka-docs/rst/dev/multi-jvm-testing.rst
      val scalatest    = Def.setting { "org.scalatest"  %% "scalatest"  % scalaTestVersion.value   % "test" } // ApacheV2
      val scalacheck   = Def.setting { "org.scalacheck" %% "scalacheck" % scalaCheckVersion.value  % "test" } // New BSD
      val pojosr       = "com.googlecode.pojosr"       % "de.kalpatec.pojosr.framework" % "0.2.1"            % "test" // ApacheV2
      val tinybundles  = "org.ops4j.pax.tinybundles"   % "tinybundles"                  % "1.0.0"            % "test" // ApacheV2
      val log4j        = "log4j"                       % "log4j"                        % "1.2.17"           % "test" // ApacheV2
      val junitIntf    = "com.novocode"                % "junit-interface"              % "0.11"             % "test" // MIT
      val scalaXml     = "org.scala-lang.modules"     %% "scala-xml"                    % scalaXmlVersion    % "test"

      // in-memory filesystem for file related tests
      val jimfs        = "com.google.jimfs"            % "jimfs"                        % "1.1"              % "test" // ApacheV2

      // metrics, measurements, perf testing
      val metrics         = "com.codahale.metrics"        % "metrics-core"                 % "3.0.2"            % "test" // ApacheV2
      val metricsJvm      = "com.codahale.metrics"        % "metrics-jvm"                  % "3.0.2"            % "test" // ApacheV2
      val latencyUtils    = "org.latencyutils"            % "LatencyUtils"                 % "1.0.3"            % "test" // Free BSD
      val hdrHistogram    = "org.hdrhistogram"            % "HdrHistogram"                 % "2.1.9"            % "test" // CC0
      val metricsAll      = Seq(metrics, metricsJvm, latencyUtils, hdrHistogram)

      // sigar logging
      val slf4jJul      = "org.slf4j"                   % "jul-to-slf4j"                 % slf4jVersion    % "test"    // MIT
      val slf4jLog4j    = "org.slf4j"                   % "log4j-over-slf4j"             % slf4jVersion    % "test"    // MIT

      // reactive streams tck
      val reactiveStreamsTck = "org.reactivestreams" % "reactive-streams-tck" % "1.0.0" % "test" // CC0
    }

    object Provided {
      // TODO remove from "test" config
      // If changed, update akka-docs/build.sbt as well
      val sigarLoader  = "io.kamon"         % "sigar-loader"        % "1.6.6-rev002"     %     "optional;provided;test" // ApacheV2

      val levelDB       = "org.iq80.leveldb"            % "leveldb"          % "0.7"    %  "optional;provided"     // ApacheV2
      val levelDBNative = "org.fusesource.leveldbjni"   % "leveldbjni-all"   % "1.8"    %  "optional;provided"     // New BSD
    }

  }

  import Compile._
  // TODO check if `l ++=` everywhere expensive?
  val l = libraryDependencies

  val actor = l ++= Seq(config, java8Compat.value)

  val testkit = l ++= Seq(Test.junit, Test.scalatest.value) ++ Test.metricsAll

  val actorTests = l ++= Seq(Test.junit, Test.scalatest.value, Test.commonsCodec, Test.commonsMath, Test.mockito, Test.scalacheck.value, Test.junitIntf)

  val remote = l ++= Seq(netty, aeronDriver, aeronClient, Test.junit, Test.scalatest.value, Test.jimfs)

  val remoteTests = l ++= Seq(Test.junit, Test.scalatest.value, Test.scalaXml)

  val cluster = l ++= Seq(Test.junit, Test.scalatest.value)

  val clusterTools = l ++= Seq(Test.junit, Test.scalatest.value)

  val clusterSharding = l ++= Seq(Provided.levelDB, Provided.levelDBNative, Test.junit, Test.scalatest.value, Test.commonsIo)

  val clusterMetrics = l ++= Seq(Provided.sigarLoader, Test.slf4jJul, Test.slf4jLog4j, Test.logback, Test.mockito)

  val distributedData = l ++= Seq(lmdb, Test.junit, Test.scalatest.value)

  val slf4j = l ++= Seq(slf4jApi, Test.logback)

  val agent = l ++= Seq(scalaStm.value, Test.scalatest.value, Test.junit)

  val persistence = l ++= Seq(Provided.levelDB, Provided.levelDBNative, Test.scalatest.value, Test.junit, Test.commonsIo, Test.commonsCodec, Test.scalaXml)

  val persistenceQuery = l ++= Seq(Test.scalatest.value, Test.junit, Test.commonsIo)

  val persistenceTck = l ++= Seq(Test.scalatest.value.copy(configurations = Some("compile")), Test.junit.copy(configurations = Some("compile")))

  val persistenceShared = l ++= Seq(Provided.levelDB, Provided.levelDBNative)

  val camel = l ++= Seq(camelCore, Test.scalatest.value, Test.junit, Test.mockito, Test.logback, Test.commonsIo, Test.junitIntf)

  val osgi = l ++= Seq(osgiCore, osgiCompendium, Test.logback, Test.commonsIo, Test.pojosr, Test.tinybundles, Test.scalatest.value, Test.junit)

  val docs = l ++= Seq(Test.scalatest.value, Test.junit, Test.junitIntf, Docs.sprayJson, Docs.gson)

  val contrib = l ++= Seq(Test.junitIntf, Test.commonsIo)

  val benchJmh = l ++= Seq(Provided.levelDB, Provided.levelDBNative)

  // akka stream

  lazy val stream = l ++= Seq[sbt.ModuleID](
    reactiveStreams,
    sslConfigCore,
    Test.junitIntf,
    Test.scalatest.value)

  lazy val streamTestkit = l ++= Seq(Test.scalatest.value, Test.scalacheck.value, Test.junit)

  lazy val streamTests = l ++= Seq(Test.scalatest.value, Test.scalacheck.value, Test.junit, Test.commonsIo, Test.jimfs)

  lazy val streamTestsTck = l ++= Seq(Test.scalatest.value, Test.scalacheck.value, Test.junit, Test.reactiveStreamsTck)

}

object DependencyHelpers {
  case class ScalaVersionDependentModuleID(modules: String => Seq[ModuleID]) {
    def %(config: String): ScalaVersionDependentModuleID =
      ScalaVersionDependentModuleID(version => modules(version).map(_ % config))
  }
  object ScalaVersionDependentModuleID {
    implicit def liftConstantModule(mod: ModuleID): ScalaVersionDependentModuleID = versioned(_ => mod)

    def versioned(f: String => ModuleID): ScalaVersionDependentModuleID = ScalaVersionDependentModuleID(v => Seq(f(v)))
    def fromPF(f: PartialFunction[String, ModuleID]): ScalaVersionDependentModuleID =
      ScalaVersionDependentModuleID(version => if (f.isDefinedAt(version)) Seq(f(version)) else Nil)
  }

  /**
   * Use this as a dependency setting if the dependencies contain both static and Scala-version
   * dependent entries.
   */
  def versionDependentDeps(modules: ScalaVersionDependentModuleID*): Def.Setting[Seq[ModuleID]] =
    libraryDependencies ++= modules.flatMap(m => m.modules(scalaVersion.value))

  val ScalaVersion = """\d\.\d+\.\d+(?:-(?:M|RC)\d+)?""".r
  val nominalScalaVersion: String => String = {
    // matches:
    // 2.12.0-M1
    // 2.12.0-RC1
    // 2.12.0
    case version @ ScalaVersion() => version
    // transforms 2.12.0-custom-version to 2.12.0
    case version => version.takeWhile(_ != '-')
  }
}
```

JMH:

```
https://github.com/akka/akka/blob/177ac666da323e8af07a35159f35c308e873b585/akka-bench-jmh/build.sbt

import akka._
import com.typesafe.sbt.pgp.PgpKeys.publishSigned

enablePlugins(JmhPlugin, ScaladocNoVerificationOfDiagrams)
disablePlugins(Unidoc, MimaPlugin)

AkkaBuild.defaultSettings

AkkaBuild.dontPublishSettings
AkkaBuild.dontPublishDocsSettings
Dependencies.benchJmh
```
