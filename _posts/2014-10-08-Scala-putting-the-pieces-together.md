---
layout: post
title: Exploring high order functions and other functional concepts for a simple use case
category: scala
tags: scala functional-programming
---

Scala is a interesting language as it lets developers write code in both functional and object-oriented paradigms. While I have been learning functional programming as well as scala, everyday, I come across features that lets me solve problems in a concise fashion which otherwise would have required a lot of boiler-plate code in a language like Java. 

That being said, it is always good to write code that uses various concepts of a language together to solve a problem. Not only it helps in revising/recollecting whatever one has learnt, but also improves the understanding of those concepts. This is quite similar to learning a new tool or driving a car. More we do it in different interesting ways, better we understand.

This blog provides a solution to a very common, but necessary requirement, finding the performance or computation time of a function or an algorithm. Using higher order functions, parameterization, call-by-name and other features, I have tried to make it generic enough to be used for any function that computes a value.

I will provide the code first, then decompose the various parts. Here it goes:

{% highlight scala linenos %}

object PerfMeasures {

  case class Stats[T](result: T, averageTime: Double, stdDev: Double) {
    override def toString = s"function computes result: +$result " +
      s"in $averageTime ms on average with standard deviation $stdDev"
  }

  def getStats[O](f: => O, n: Int = 5): Stats[O] = {
    assert(n > 1)

    def compute: (O, Double) = {
      val t1 = System.nanoTime().toDouble
      val res = f
      val t2 = System.nanoTime().toDouble
      (res, (t2 - t1) / 1e6)
    }

    val (results, times) = (1 to n + 5).map(x => compute).drop(5).unzip

    val avgTime = times.reduceLeft(_ + _) / times.size

    val stdDev = standardDeviation(avgTime, times)

    Stats(results.head, avgTime, stdDev)
  }

  def standardDeviation(average: Double, list: Seq[Double]) = list match {
    case Nil => 0.0
    case _ =>
      math.sqrt(list.foldLeft(0.0) { 
      		case (acc, x) => acc + math.pow(x - average, 2)
		} / list.length)

  }

  def publishPerformanceData[O](f: => O, n: Int = 5): Unit = {
    val stats = getStats(f, n)
    println(stats)
  }
}

{% endhighlight %}

Lets look at the declaration of the main function

{% highlight scala %}
def getStats[O](f: => O, n: Int = 5): Stats[O] {
    ???
}
{% endhighlight %}

This function takes takes two parameters:

1.  **function**, this is a function/body, when executed returns a value of type O (which can be any type)
2.  **numRuns**, this is number of iterations for the computation to arrive at the statistics, its default value is 5

The important thing to note here is that function is a **call-by-name** parameter, this means, it will be evaluated lazily when required or called. The functions body is self-explanatory where it performs the computation **numRuns** times plus 5, which is later dropped. This extra 5 computation calls let the JVM perform any optimization and also avoid taking into account initial warm-ups. I chose a small number. It can be any number based on the requirements. 

Two statistical values of interest are calculated:

* Average : here list functions and combinators such as map, drop and reduceLeft come handy

{% highlight scala %}
val (results, times) = (1 to n + 5).map(x => compute).drop(5).unzip
val avgTime = times.reduceLeft(_ + _) / times.size
{% endhighlight %}

* [Standard Deviation](http://en.wikipedia.org/wiki/Standard_deviation): its a simple computation using the standard mathematical formula and foldLeft
{% highlight scala %}
math.sqrt(
list.foldLeft(0.0) { case (acc, x) => acc + math.pow(x - average, 2)} / list.length)
{% endhighlight %}

The returned result is an instance of a case class
{% highlight scala %}
case class Stats[T](result: T, averageTime: Double, stdDev: Double) {
    override def toString = s"function computes result: +$result " +
      s"in $averageTime ms on average with standard deviation $stdDev"
  }
{% endhighlight %}
**case classes** are useful for defining immutable types which can also be pattern matched. Apart from that, they provide implementations of hashcode, equals and other useful methods based on their member fields. This obviates a lot of boiler plate code.

While solving various computational problems in scala or writing two different versions of the same computation or even tweaking my algorithm, this code has helped me a lot to reason about the performance of the code. This acts as a one stop destination for all the functions I write and then check their performance. The thing I liked about the **higher order functions** is they let us compose computations without mandating any dependency or coupling between them. In this case, any function I write does not need to have any knowledge or relation in order to be able to called for statistical computation.

On the other hand, using the case class (which is akin to defining an Immutable Class in Java, but far less verbose) helps me return the result and stats in an api-friendly way.

Some example invocations
{% highlight scala %}
PerfMeasures.getStats(distinctPowers(2 to 100), 10) //distinctPowers(2 to 100)
PerfMeasures.getStats(sum) //sum
PerfMeasures.getStats(numOfCircularPrimes(), 10) //numOfCircularPrimes()
{% endhighlight %}
Needless to say, it can be further refined and made more concise. The intent here is to help readers as well as myself to appreciate the power of functional programming; interesting ways to abstract or compose functions and use it to write less code and reuse more!! :)
