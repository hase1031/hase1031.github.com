---
layout: post
title: "sonatypeにpublishした"
description: ""
category: ""
tags: [scala]
---
{% include JB/setup %}

## 経緯

知り合いが作っている[trinity](https://github.com/sisioh/trinity)というリポジトリをpublishすることになった。

以前に権限はもらっていたのだが、今回は変更を加えてリリースする必要があり、初めてのSonatypeへのリリースだったのでメモ。


## 登録からpublishまで

[自分が作ったScalaライブラリをMaven Centralリポジトリに登録する方法 (GaedsをSonatype OSSRHにデプロイしたよ＼(^o^)／)](http://pab-tech.tumblr.com/post/23240310972/scala-maven-central)を参考に設定させていただく。

PGP署名とsonatype.sbtの場所だけ変えたので記述。

- PGP署名

~~~
$ gpg --gen-key
$ gpg --list-keys
$ gpg --send-keys PublicKeyのID // キーサーバを指定するとうまくいかなかった
~~~

- sonatype.sbt

あとで考えれば当たり前ことだが、sbtは0.13系からバージョンごとにディレクトリができたのでその下に置くようにする。

~~~
~/.sbt/0.13/sonatype.sbt
~~~

間違えて ~/.sbt/sonatype.sbt に置くと認証できないので、以下のエラーが出てしまう。

~~~
[error] (*:publish) java.io.IOException: Access to URL ... was refused by the server: Unauthorized
~~~


他は上記URL資料でうまくいくはず。

一応備忘録的に書いておくと、publishは以下コマンドおこなった。

~~~
sbt clean publishSigned
~~~

その後、[sonatypeのホームページ](https://oss.sonatype.org/index.html#welcome)にアクセス。
 Loginをクリック、JIRAアカウントでログインする。
 
 ログインしたら、ステージリポジトリの一覧の中から、自分がpublishしたリポジトリ(今回はsisioh)のものをみつけて、それをCloseして、その後Releaseすると公開される。



これで終わり！
