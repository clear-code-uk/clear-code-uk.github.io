---
layout: post
category: Code
title: Mongo query DSL
tags:
- Scala
- MongoDB
---

# Filters

```scala
import org.mongodb.scala.bson.Document
import org.mongodb.scala.bson.conversions.Bson
import org.mongodb.scala.model.Filters

trait MongoFilter {
  def toBson: Bson

  def &&(that: MongoFilter): MongoFilter = AndMongoFilter(Seq(this, that))
  def ||(that: MongoFilter): MongoFilter = OrMongoFilter(Seq(this, that))
  def ! : MongoFilter = NotMongoFilter(this)
}

object MongoFilter {
  import scala.language.implicitConversions
  implicit def toBson(filter: MongoFilter): Bson = filter.toBson
}

case object EmptyMongoFilter extends MongoFilter {
  private val empty = Document()
  override def toBson: Bson = empty

  override def &&(that: MongoFilter): MongoFilter = that
  override def ||(that: MongoFilter): MongoFilter = that
  override def ! : MongoFilter = this
}

case class NotMongoFilter(filter: MongoFilter) extends MongoFilter {
  override def toBson: Bson        = Filters.not(filter)
  override def !     : MongoFilter = filter
}

case class AndMongoFilter(filters: Seq[MongoFilter]) extends MongoFilter {
  override def toBson: Bson = Filters.and(filters.map(_.toBson): _*)
  override def &&(that: MongoFilter): MongoFilter = copy(filters :+ that)
}

case class OrMongoFilter(filters: Seq[MongoFilter]) extends MongoFilter {
  override def toBson: Bson = Filters.or(filters.map(_.toBson): _*)
  override def ||(that: MongoFilter): MongoFilter = copy(filters :+ that)
}

case class SimpleMongoFilter(filter: Bson) extends MongoFilter {
  override def toBson: Bson = filter
}
```

# Fields

```scala
import org.mongodb.scala.model.Filters

import java.util.Date

trait MongoField[A] {
  val name: String
  def translation(a: A): Any = a
}

trait EqualComparisons[A] {
  this: MongoField[A] =>

  final def :==(value: A): MongoFilter = SimpleMongoFilter(Filters.eq(name, translation(value)))
}

trait RelaxedOptionComparisons[A] {
  this: EqualComparisons[A] =>

  final def :==(maybeValue: Option[A]): MongoFilter = maybeValue.fold[MongoFilter](EmptyMongoFilter)(:==)
}

trait OrderComparisons[A] extends EqualComparisons[A] {
  this: MongoField[A] =>

  final def :<(value:  A): MongoFilter = SimpleMongoFilter(Filters.lt(name, translation(value)))
  final def :<=(value: A): MongoFilter = SimpleMongoFilter(Filters.lte(name, translation(value)))
  final def :>=(value: A): MongoFilter = SimpleMongoFilter(Filters.gte(name, translation(value)))
  final def :>(value:  A): MongoFilter = SimpleMongoFilter(Filters.gt(name, translation(value)))
}

trait RegexComparison {
  this: MongoField[String] =>
  def matches(exp: String): MongoFilter = SimpleMongoFilter(Filters.regex(name, exp))
}

class StringMongoField(val name: String) extends MongoField[String] with OrderComparisons[String] with RegexComparison
object StringMongoField {
  def apply(name: String): StringMongoField = new StringMongoField(name)
}

class DateMongoField(val name: String) extends MongoField[Date] with OrderComparisons[Date]
object DateMongoField {
  def apply(name: String): DateMongoField = new DateMongoField(name)
}

class TranslatedField[A](val name: String, translationFunction: A => Any) extends MongoField[A] {
  override def translation(a: A): Any = translationFunction(a)
}
```

# Example

```scala
object MongoFields {
  val Path                = new StringMongoField("path") with RelaxedOptionComparisons[String]
  val Filetype            = new StringMongoField("filetype") with RelaxedOptionComparisons[String]
  val Filename            = StringMongoField("filename")
  val OriginalFilename    = StringMongoField("originalFilename")
  val Direction           = StringMongoField("direction")
  val CorrelationId       = StringMongoField("correlationId")
  val ScheduledDeletionAt = DateMongoField("scheduledDeletionAt")
  val IsDeleted           = new TranslatedField[Boolean]("delete", (value: Boolean) => if (value) "Y" else "N") with EqualComparisons[Boolean]
}
```
```scala
collection
  .replaceOne(
    (IsDeleted :== false) && (Path :== updatedEntity.path) && (Filetype :== updatedEntity.filetype) && (Filename :== updatedEntity.filename),
    updatedEntity
  )
```
