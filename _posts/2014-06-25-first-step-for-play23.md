---
layout: post
title: "First step for Play2.3"
description: ""
category: ""
tags: [scala,play]
---
{% include JB/setup %}


[本家のチュートリアル(英語)](http://www.playframework.com/documentation/2.3.x/Tutorials)に従って作業を進めます。


## Activatorをインストール
[Activator distribution](https://typesafe.com/platform/getstarted)から最新版を取得します。
展開後、読み書きできる場所に起きます。

次にPATHを通します。
~~~
export PATH=$PATH:/relativePath/to/activator
~~~

Windowsの場合はグローバルな環境変数に設定し、Unixの場合は実行可能ファイルにしておきます。
~~~
bash chmod a+x activator
~~~

終わったらコマンドが有効かどうかチェックします。
~~~
activator -help
~~~
~~~
$ activator -h
Usage: activator <command> [options]

  Command:
  ui                 Start the Activator UI
  new [name] [template-id]  Create a new project with [name] using template [tem
plate-id]
  list-templates     Print all available template names
  -h | -help         Print this message

  Options:
  -v | -verbose      Make this runner chattier
  -d | -debug        Set sbt log level to debug
  -mem <integer>     Set memory options (default: , which is -Xms1024m -Xmx1024m
 -XX:PermSize=64m -XX:MaxPermSize=256m)
  -jvm-debug <port>  Turn on JVM debugging, open at the given port.

  # java version (default: java from PATH, currently java version "1.7.0_51")
  -java-home <path>  Alternate JAVA_HOME

  # jvm options and output control
  -Dkey=val          Pass -Dkey=val directly to the java runtime
  -J-X               Pass option -X directly to the java runtime
                     (-J is stripped)

  # environment variables (read from context)
  JAVA_OPTS          Environment variable, if unset uses ""
  SBT_OPTS           Environment variable, if unset uses ""
  ACTIVATOR_OPTS     Environment variable, if unset uses ""

In the case of duplicated or conflicting options, the order above
shows precedence: environment variables lowest, command line options highest.
~~~


## 新しいアプリケーションを作成する
activatorコマンドを利用することでPlayのアプリケーションを作成することが可能です。
様々なテンプレートから作ることができるのですが、今回はvanilla Play Scalaで作ります。
~~~
$ activator new my-first-app play-scala
~~~

アプリケーションを作ってしまえばactivatorコマンドで[Play console](http://www.playframework.com/documentation/2.3.x/PlayConsole)に入ることができます。
~~~
$ cd my-first-app
$ activator
~~~

他にも[Activator UI](https://typesafe.com/activator/docs)を利用して作成する方法や、sbtを利用して作成する方法があります。
~~~
$ activator ui
~~~

sbtを利用する場合、~~~project/plugins.sbt~~~
~~~scala
// The Typesafe repository
resolvers += "Typesafe repository" at "http://repo.typesafe.com/typesafe/releases/"

// Use the Play sbt plugin for Play projects
addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.3.x")
~~~

~~~build.sbt~~~
~~~scala
name := "my-first-app"

version := "1.0.0-SNAPSHOT"

lazy val root = (project in file(".")).enablePlugins(PlayScala)
~~~

~~~
$ cd my-first-app
$ sbt
~~~


## アプリケーションを動かしてみる
さきほどのコンソールから作業します。
デフォルトで9000番ポートが立ち上がるのですが、今回は2000番ポートを指定しました。
~~~
[my-first-app] $ run 2000

--- (Running the application from SBT, auto-reloading is enabled) ---

[info] play - Listening for HTTP on /0:0:0:0:0:0:0:0:2000

(Server started, use Ctrl+D to stop and go back to the console...)

[info] Compiling 5 Scala sources and 1 Java source to /home/hasegawa/my-first-ap
p/target/scala-2.11/classes...
[info] 'compiler-interface' not yet compiled for Scala 2.11.1. Compiling...
~~~

実際にブラウザからアクセスしてみると…
![Play Sample](http://www8096ui.sakura.ne.jp/~hasegawa/img/play.jpg)


## 最後に
これで最低限の準備はできました。
IntelliJを使っている方は、~~~[my-first-app] $ idea with-sources=yes~~~でプロジェクトを作成できます。

