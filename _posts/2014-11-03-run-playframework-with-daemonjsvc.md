---
layout: post
title: "Run Play Framework with Daemon(jsvc)"
description: ""
category:
tags: []
---
{% include JB/setup %}

## JSVCで起動する利点

- ルート以外に実行権限を渡せる
    - セキュリティとかポートとか
- 終了コマンドとかを教えられる
    - 終了前にする操作とか実装できる
- ＪＶＭがクラッシュしたときjsvcが再起動してくれる


## Playframeworkでの実装

参考にしたのは[かとじゅんさんのメモ](https://gist.github.com/j5ik2o/2156447).

使用したPlayのバージョンは[2.3.6](https://github.com/playframework/playframework/releases/tag/2.3.6).


- Build.scala

~~~scala
object ApplicationBuild extends Build {
    // ...
    val appDependencies = Seq(
      "commons-daemon" % "commons-daemon" % "1.0.15"
    )
    // ...
}
~~~


- Application.scala

~~~scala
package daemon

import org.apache.commons.daemon.Daemon
import org.apache.commons.daemon.DaemonContext
import play.core.server.NettyServer
import play.core.StaticApplication
import java.io.File

class Application extends Daemon {

  private var nettyServer: NettyServer = _
  private var applicationPath: File = _

  override def init(dc: DaemonContext) {
    applicationPath = new File(dc.getArguments()(0))
  }

  override def start {
    //Play内部で利用しているソースコードをそのままコピペ
    nettyServer = new NettyServer(
      new StaticApplication(applicationPath),
      Option(System.getProperty("http.port")).fold(Option(9000))(p => if (p == "disabled") Option.empty[Int] else Option(Integer.parseInt(p))),
      Option(System.getProperty("https.port")).map(Integer.parseInt(_)),
      Option(System.getProperty("http.address")).getOrElse("0.0.0.0")
    )
  }

  override def stop {
    if (nettyServer != null) nettyServer.stop()
  }

  override def destroy {
    nettyServer = null
  }

}
~~~


- 起動スクリプト
参考にしてて引っかかったのが起動スクリプト

~~~bash
#/bin/sh
CLASSPATH=.:./lib/* #if you use activator with stage, target/universal/stage/lib
PID_FILE=./pidfile

start()
{
./jsvc \
        -home $JAVA_HOME \
        -cp "$CLASSPATH" \ #need double quote because of lib's wildcard
        -pidfile $PID_FILE \
        -Dlogback.configurationFile=./logback.xml \
        -debug -verbose \
        -outfile stdout.log \
        -errfile stderr.log \
        daemon.Application \
        `pwd`
}

stop()
{
./jsvc \
        -stop \
        -pidfile $PID_FILE \
        daemon.Application
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart)
        stop
        start
        ;;
  *)
        echo "Usage $0 start/stop"
        exit 1;;
esac
~~~


コメントとしても書いているのですが，classpathにワイルドカードを利用しているせいでダブルクオートを付けないとうまく動きません．

付けない状態で実行してしまうと，何も表示されなかったり、以下のエラーが出ます.

~~~
Java VM created successfully
Cannot find daemon loader org/apache/commons/daemon/support/DaemonLoader
java_init failed
Service exit with a return value of 1
~~~


debugオプションをつけるよくわかりますが、正しくオプションを指定できていないことがわかります（以下例

~~~
+-- DUMPING PARSED COMMAND LINE ARGUMENTS --------------
| Detach:          True
| Show Version:    No
| Show Help:       No
| Check Only:      Disabled
| Stop:            False
| Wait:            0
| Run as service:  No
| Install service: No
| Remove service:  No
| JVM Name:        "null"
| Java Home:       "/Library/Java/JavaVirtualMachines/jdk1.7.0_67.jdk/Contents/Home"
| PID File:        "/var/run/jsvc.pid"
| User Name:       "null"
| Extra Options:   2
|   "-verbose"
|   "-Djava.class.path=target/universal/stage/lib/ch.qos.logback.logback-classic-1.1.1.jar"
| Class Invoked:   "target/universal/stage/lib/ch.qos.logback.logback-core-1.1.1.jar"
| Class Arguments: 66
|   "target/universal/stage/lib/com.eaio.uuid.uuid-3.2.jar"
|   "target/universal/stage/lib/com.fasterxml.jackson.core.jackson-annotations-2.3.2.jar"
|   "target/universal/stage/lib/com.fasterxml.jackson.core.jackson-core-2.3.2.jar"
|   "targ
~~~


これで出来るはずです。
