---
layout: post
title: Anatomy of a tail recursive function
category: functional-programming
tags: scala functional-programming

---

This is my first blog about programming and it is more of an effort to publish my thoughts and realisations which I can refer to, later on. It's been more than a year since I started exploring functional programming. I have to admit that this has been an absolutely fantastic journey where I realised the effectiveness of compositions, higher order functions, the value immutability adds to the code maintenance, importance of writing pure code (no side-effects). While I have been learning scala and dabbled a bit in Haskell, I am trying to solve more and more algorithmic problems in a [pure functional](http://en.wikipedia.org/wiki/Functional_programming), side-effect free fashion. 

When solving one such problem: Finding if a number is [pandigital](http://en.wikipedia.org/wiki/Pandigital_number) or not, I came across a very effective and efficient solution as mentioned on [stackoverflow thread](http://stackoverflow.com/questions/2484892/fastest-algorithm-to-check-if-a-number-is-pandigital). After understanding the algorithm, I wanted to write it in  a pure functional and [tail recursive](http://en.wikipedia.org/wiki/Tail_call) manner.

While doing so, I realised there are certain paradigms involved when writing such an algorithm in a tail-recursive fashion. Thats what I wanted to share with fellow programmers and of course, with my future self, when I need to refer it again. :-)

Here is the pure functional tail-recursive version, which I will decompose part-by-part.

{% highlight scala linenos %}
def isPandigital(n: Long): Boolean = {
    def loop(x: Long, digits: Int, count: Int): Boolean = {
      //number of 1's in digits is equal to the length of the number
      if (x == 0) digits == (1 << count) - 1
      else {
        //get a 1 placed on the digits value
        val nextDigits = digits | 1 << ((x % 10) - 1)
        //if nextDigits is not changed, then it is same digit repeated here
        if (nextDigits == digits) false
        //tail recursion
        else loop(x / 10, nextDigits, count + 1)
      }
    }
    loop(n,0,0)
  }
{% endhighlight %}

**Accumulators:**
Since we are looking at a tail-recursive solution, we will most likely need a inner function which can take 1 or more accumulators and one of them might even be the return value on exit. Their value will change with each recursive call.

{% highlight scala %}
def loop(x: Long, digits: Int, count: Int): Boolean
{% endhighlight %}

**Exit Value:**
This is the value that needs to be returned on the base or exit condition, which signifies that we no longer need to recurse. It could be simple return of an accumulator or some calculations based on them.

{% highlight scala %}
if (x == 0) digits == (1 << count) - 1
{% endhighlight %}

**Computations and possible exits:**
This comes in the else part of base or exit-check. It involves doing some computations using function parameters and getting new set of values for the next recursive call. This could also possibly include some condition checks that can mark the exit which is similar to a break case for a while loop.

{% highlight scala %}
val nextDigits = digits | 1 << ((x % 10) - 1)
if (nextDigits == digits) false
{% endhighlight %}

**Tail Recursive call:**
Next comes the recursive part, which is a simple call with next set of accumulators. The important thing to remember here is that it should be the last call which effectively makes the function tail-recursive.
{% highlight scala %}
else loop(x / 10, nextDigits, count + 1)
{% endhighlight %}

**Calling the inner function:**
This is the call from the main function with its parameters and initial values of the accumulators
{% highlight scala %}
loop(n,0,0)
{% endhighlight %}

Thus ends the story of a tail-recursive function. This was slightly more than a simple algorithm. It involved more than one accumulator. That's what makes it a good candidate for analysis. 
Functional Programming is interesting and useful, but with our brains tuned for imperative programming after years of java coding, it some times becomes difficult to see a recursive or a tail-recursive solution right through. Just remember....
if *"All men must die" then "All whiles can be tail-recursed"*   :-)
