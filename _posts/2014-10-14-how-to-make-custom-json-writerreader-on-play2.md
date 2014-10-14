---
layout: post
title: "How to make custom json writer/reader on Play2"
description: ""
category: 
tags: []
---
{% include JB/setup %}

## 基本的な実装

Playには最初からInt型やBoolean型のような基本的な型に対するReader/Writerが用意されています。

### Reader
~~~ scala
/**
* Default deserializer type classes.
*/
trait DefaultReads {
  /**
   * Deserializer for Int types.
   */
  implicit object IntReads extends Reads[Int] {
    def reads(json: JsValue) = json match {
      case JsNumber(n) => JsSuccess(n.toInt)
      case _ => JsError(Seq(JsPath() -> Seq(ValidationError("error.expected.jsnumber"))))
    }
  }
  /**
   * Deserializer for Boolean types.
   */
  implicit object BooleanReads extends Reads[Boolean] {
    def reads(json: JsValue) = json match {
      case JsBoolean(b) => JsSuccess(b)
      case _ => JsError(Seq(JsPath() -> Seq(ValidationError("error.expected.jsboolean"))))
    }
  }
  ...
}
~~~

### Writer
~~~ scala
/**
 * Default Serializers.
 */
trait DefaultWrites {
  /**
   * Serializer for Int types.
   */
  implicit object IntWrites extends Writes[Int] {
    def writes(o: Int) = JsNumber(o)
  }

  /**
   * Serializer for Boolean types.
   */
  implicit object BooleanWrites extends Writes[Boolean] {
    def writes(o: Boolean) = JsBoolean(o)
  }
  ...
}
~~~


## Customな型をRead/Writeできるようにする

Customで作る場合も上記と書き方は変わりません。

例えば、Date型をJsonで書きこめるようにしてみます。（Date型は標準でサポートされているのですが…

以下の場合、java.util.Dateを2014-10-14という形式で書き出します。

~~~ scala
implicit object DateWriter extends Writes[Date] {
  def writes(o: Date) = JsString(new SimpleDateFormat("yyyy-MM-dd").format(o))
}
~~~

その他には、UUID型を利用したCompanyIdを読み込むためには以下のように記述します。

~~~ scala
/**
 * Example of self-made type.
 */
case class CompanyId(value: UUID) 

implicit object companyIdReader extends Reads[CompanyId] {
  def reads(json: JsValue) = json match {
    case JsString(n) => JsSuccess(CompanyId(new UUID(n)))
    case _ => JsError(Seq(JsPath() -> Seq(ValidationError("error.expected.companyId"))))
  }
}
~~~


このImplicitで定義したReader/Writerを使う場合は以下のようにできます。

~~~ scala
implicit val companyWriter: Writes[Company] = (
  (JsPath \ "identifier").write[CompanyId] and
  (JsPath \ "name").write[String] and
  (JsPath \ "create_time").write[Date] and
  (JsPath \ "update_time").write[Date]
)(unlift(Company.unapply))
~~~



実際に作るときはPlayのコードを見ながら作ると良いと思います。

[Read](https://github.com/playframework/playframework/blob/2.3.5/framework/src/play-json/src/main/scala/play/api/libs/json/Reads.scala)

[Write](https://github.com/playframework/playframework/blob/2.3.5/framework/src/play-json/src/main/scala/play/api/libs/json/Writes.scala)

