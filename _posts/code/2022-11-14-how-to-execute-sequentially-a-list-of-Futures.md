---
layout: post
title: How to execute sequentially a list of Futures
category: Code
tags:
  - Scala
---

Simple code:
```scala
import scala.concurrent.{ExecutionContext, Future}

def sequentially[A, B](
    items: IterableOnce[A]
)(
    f: A => Future[B]
)(implicit
    ec: ExecutionContext
): Future[List[B]] =
  items.iterator.foldLeft(Future.successful(List.empty[B])) {
    (accFuture, item) =>
      for {
        acc <- accFuture
        processedItem <- f(item)
      } yield processedItem :: acc
  } map (_.reverse)
```
  
Keeps the container type the same as the argument:
```scala
import scala.collection.BuildFrom
import scala.concurrent.{ExecutionContext, Future}

def sequentially[A, B, M[X] <: IterableOnce[X]](
    items: M[A]
)(
    f: A => Future[B]
)(implicit
    bf: BuildFrom[M[A], B, M[B]],
    executor: ExecutionContext
): Future[M[B]] =
  items.iterator.foldLeft(Future.successful(List.empty[B])) {
    (accFuture, item) =>
      for {
        acc <- accFuture
        processedItem <- f(item)
      } yield processedItem :: acc
  } map (list => bf.newBuilder(items).addAll(list.reverse).result)
```
  
  with parasitic:
  ```scala
import scala.collection.BuildFrom
import scala.concurrent.ExecutionContext.parasitic
import scala.concurrent.{ExecutionContext, Future}

def sequentially[A, B, M[X] <: IterableOnce[X]](
    items: M[A]
)(
    f: A => Future[B]
)(implicit
    bf: BuildFrom[M[A], B, M[B]],
    executor: ExecutionContext
): Future[M[B]] =
  items.iterator
    .foldLeft(Future.successful(List.empty[B]))((accFuture, item) =>
      for {
        acc <- accFuture
        processedItem <- f(item)
      } yield processedItem :: acc
    )
    .map(list => bf.newBuilder(items).addAll(list.reverse).result)(parasitic)
  ```
