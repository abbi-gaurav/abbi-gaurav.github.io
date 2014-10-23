---
layout: post
title: Functional Thoughts in Java Programming - Lifting the Exceptions
category: java
tags: java functional-programming

---

Lambdas in Java 8 is a nice feature and let the developers write code in a concise and elegant way. It lets one think about the structure of a program functionally. I am writing series of posts that explore functional paradigms that one can apply while writing code in Java. This is first of those posts and I hope to end up writing a lot of them.

Almost all the Java Projects are written in a modular fashion, where the product is split into various modules and they interact with each other via APIs. Normally those structures follow two paths:

1. Abstractions, where low-level APIs such as IO, Messaging, DB access etc. are wrapped and decorated
2. sub-domains, where the product's functionality is split into separate domains and they as an aggregation make the full-fledged product. e.g. Accounts module and a loans module for a Banking solution

Normally these sub-domains deal in terms of their DSL both for objects as well as Exceptions (e.g. AccountsManager API will throw an AccountMgrException when things go wrong) and I can bet as a java programmer, you might have come across similar code umpteen times

{% highlight java %}
public java.util.List<String> getAccountsDetailsNormalWay()
throws AccountMgrException {
        try {
            //a low level API to get data from IO or over the wire
            return delegator.perform("/sm/path/to/Account/Records/");
        } catch (IOException e) {
            //logging can be done here
            throw new AccountMgrException(e, "Error during Fetch");
        }
}
{% endhighlight %}

this is a standard paradigm where we want to propagate the failure back to the caller code and also retain the full information about the cause (catch block). The **perform** API signature looks somewhat like this

{% highlight java %}
public java.util.List<String> perform(String fileName) throws IOException;
{% endhighlight %}

Now this try-catch is boilerplate and if we are calling the low-level API **perform** at many places, then we will end up duplicating the paradigm at all those places.

By putting some functional thoughts and using lambdas, we can get rid of this without compromising on the contract of our API **getAccountsDetailsNormalWay**. Below is the piece of code that uses Java generics and lambdas to help us lift the exceptions to a DSL type we want.

{% highlight java %}
public <E extends Exception> List<String>
perform(String fileName, Function<IOException, E> exceptionLifter) throws E {
    try {
        return Files.readAllLines(Paths.get(fileName));
    } catch (IOException e) {
	//logging can be added here
	throw exceptionLifter.apply(e);
    }
}

public List<String> perform(String path) throws IOException {
    return perform(path, ex -> ex);
}
{% endhighlight %}
With this our **getAccountsDetails** API implementation looks like this

{% highlight java %}

public java.util.List<String> getAccountsDetails() throws AccountMgrException {
        return delegator.perform("/sm/path/to/Account/Records/",
			ex -> new AccountMgrException(ex, "Error during Fetch"));
}

{% endhighlight %}

The second argument to the perform call is a lambda that lifts the IOExcetion to AccountMgrException.
 
Now there is no boiler-plate try-catch, but we still get the same contract. Java's type inference has improved enough to figure out the necessary types for the exception. Thus the same **perform** API can also be used by the Loans module to do their domain work and propagate failures in terms of their DSL. 

{% highlight java %}

public java.util.List<String> getLoanRecords() throws LoanManagerException {
    return recordFetcher.perform("/sm/path/to/Loan/Records/",
		ex -> new LoanManagerException("Error fetching loan records", ex));
}

{% endhighlight %}

Notice there is a overloaded **perform** method that still provides the old behavior of throwing a IOException which is again defined using lambdas. This is useful for scenarios where domain APIs need to do something different like retrying.

Similar pattern can be achieved in Java7 also although that will be a bit verbose.

Using functional paradigms will always let one write code that is concise, more type-safe and with lesser possibility of bugs but serves the same purpose as that of its imperative equivalent. With lambdas in java 8, it has become easier and we as developers should always look for possibilities to utilize it.

Full code is available as a gist [BankAPIClient.java](https://gist.github.com/abbi-gaurav/6d1a9ffa19cd4bc6640d)


Please provide any suggestions which can help me make the content better since I am a novice as far as blogging is concerned :)
