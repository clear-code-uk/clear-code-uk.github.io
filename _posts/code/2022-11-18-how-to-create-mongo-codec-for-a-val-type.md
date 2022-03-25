---
layout: post
title: How to create a MongoDb Codec for a val type
category: code
---

```scala
import org.bson.codecs._
import org.bson.{BsonReader, BsonType, BsonWriter}
import shapeless._
import shapeless.ops.hlist.IsHCons

import java.util.Date
import scala.reflect.ClassTag

trait ValueWrapperMongoCodecCompanion[A] extends DefaultMongoCodecs {
  implicit def mongoCodec[Repr <: HList, Head](
      implicit ct: ClassTag[A],
      gen:         Generic.Aux[A, Repr],
      isHCons:     IsHCons.Aux[Repr, Head, HNil],
      codec:       Codec[Head]
  ): Codec[A] =
    ValueWrapperMongoCodec.valueWrapperMongoCodec
}

object ValueWrapperMongoCodec {
  implicit def valueWrapperMongoCodec[A, Repr <: HList, Head](
      implicit ct: ClassTag[A],
      gen:         Generic.Aux[A, Repr],
      isHCons:     IsHCons.Aux[Repr, Head, HNil],
      codec:       Codec[Head]
  ): Codec[A] =
    new Codec[A] {
      override def decode(reader: BsonReader, decoderContext: DecoderContext): A =
        gen.from(isHCons.cons(codec.decode(reader, decoderContext), HNil))

      override def encode(writer: BsonWriter, value: A, encoderContext: EncoderContext): Unit =
        codec.encode(writer, gen.to(value).head, encoderContext)

      override def getEncoderClass: Class[A] =
        ct.runtimeClass.asInstanceOf[Class[A]]
    }
}

trait DefaultMongoCodecs {
  implicit val booleanMongoCodec: Codec[Boolean] = bimap[java.lang.Boolean, Boolean](new BooleanCodec(), Boolean.unbox, Boolean.box)
  implicit val intMongoCodec:     Codec[Int]     = bimap[Integer, Int](new IntegerCodec(), Int.unbox, Int.box)
  implicit val longMongoCodec:    Codec[Long]    = bimap[java.lang.Long, Long](new LongCodec(), Long.unbox, Long.box)
  implicit val floatMongoCodec:   Codec[Float]   = bimap[java.lang.Float, Float](new FloatCodec(), Float.unbox, Float.box)
  implicit val doubleMongoCodec:  Codec[Double]  = bimap[java.lang.Double, Double](new DoubleCodec(), Double.unbox, Double.box)
  implicit val stringCodec:       Codec[String]  = new StringCodec()
  implicit val dateMongoCodec:    Codec[Date]    = new DateCodec()

  def bimap[A, B](codec: Codec[A], f: A => B, g: B => A)(implicit ct: ClassTag[B]): Codec[B] =
    new Codec[B] {
      override def getEncoderClass: Class[B] =
        ct.runtimeClass.asInstanceOf[Class[B]]

      override def decode(reader: BsonReader, decoderContext: DecoderContext): B =
        f(codec.decode(reader, decoderContext))

      override def encode(writer: BsonWriter, value: B, encoderContext: EncoderContext): Unit =
        codec.encode(writer, g(value), encoderContext)
    }
}

object DefaultMongoCodecs extends DefaultMongoCodecs
```

## Example
given:
```scala
case class Email(email: String) extends AnyVal
```
we can declare
```scala
object Email extends ValueWrapperMongoCodecCompanion[Email] {
  implicit val codec: Codec[Email] = mongoCodec
}
```
or
```scala
object Email {
  import DefaultMongoCodecs._
  implicit val codec: Codec[Email] = ValueWrapperMongoCodec.valueWrapperMongoCodec
}
```
