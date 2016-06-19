---
layout: post
title: Property Based Testing
---

# Overview

Property based testing (PBT) is a form of unit testing which is focused on defining conditions and behaviours we expect our functions and types to obey. Traditional unit tests may construct a set of inputs and assert results on the output. In contrast, a PBT may construct a myriad of inputs via generators and assert high level properties that should hold for all cases.

For example, a set of traditional unit tests might take a number of lists as input and assert a set of results on the outputs. The key here is that we, as a developer, define the input list and output assertions for a specific set of inputs. We must therefore be careful to ensure we don't miss any critical cases for lists (be it empty lists, singleton lists, or lists of length n). Property based testing differs from this approach. In Property based testing, we can specify generators for random sets of input types (in this case Lists), and post-conditions that all these lists must satisfy. Provided the input set size is randomised effectively and uses a large enough sample size, we alleviate ourselves from the burden of missing input cases and can focus on the the properties that are under scrutiny.

In this post we will cover ScalaCheck, which is a Scala library inspired by Haskell's QuickCheck library, which enables Property based testing.

## First steps...

To get started download the appropriate version of the [ScalaCheck](https://www.scalacheck.org/download.html) jar and launch the Scala Interpreter:

{% highlight scala %}
scala -cp scalacheck.jar
{% endhighlight %}

Now we can get started with a basic example of properties. Suppose we want to verify the following property of Lists:
{% highlight scala %}
head ( x::ls ) == x
{% endhighlight %}

We could verify this is the case using the forAll (universal quantifier) property as shown below:
{% highlight scala %}
import org.scalacheck.Prop.forAll

val propConsList = forAll { (ls: List[Int], x: Int) => 
(x::ls).head == x }

propConsList.check
{% endhighlight %}

Two interesting things are happening here. The first is that ScalaCheck is asserting our property holds over a number of arbitrary lists of Int and arbitrary Ints. Notice we did not have to define the input to our test - it was **generated** randomly a number of times for us. Behind the scenes, there exist implicit definitions which define Arbitrary instances tasked with forming the input to our property - one for Lists[Int] and one for the set of Integers. You can of course define your own Arbitrary instances or even specific Generators if you are so inclined - we will cover this shortly.
{% highlight scala %}
The result should be as follows:
propConsList.check
+ OK, passed 100 tests.
{% endhighlight %}

It might be worth experimenting with some other truths using the forAll property before moving on:
{% highlight plaintext %}
Lists:
1. ls.reverse.reverse == ls
2. (ls1 ::: ls2).size == ls1.size + ls2.size

Strings:
3. (s1 + s2).endsWith(s2)
{% endhighlight %}

**Implication**:

From the ScalaCheck examples, we see that you can also represent implication (i.e. given a set of conditions a second condition must hold):
{% highlight scala %}
import org.scalacheck.Prop.{forAll, BooleanOperators}

val propMakeList = forAll { n: Int =>
  (n >= 0 && n < 10000) ==> (List.fill(n)("").length == n)
}
{% endhighlight %}

Finally, you an group together logically similar properties using the Properties trait. An example from the ScalaCheck user guide demonstrates this nicely for properties of the String type:
{% highlight scala %}
import org.scalacheck._

object StringSpecification extends Properties("String") {
  import Prop.forAll

  property("startsWith") = forAll { (a: String, b: String) =>
    (a+b).startsWith(a)
  }

  property("endsWith") = forAll { (a: String, b: String) =>
    (a+b).endsWith(b)
  }

  property("substring") = forAll { (a: String, b: String) =>
    (a+b).substring(a.length) == b
  }

  property("substring") = forAll { (a: String, b: String, c: String) =>
    (a+b+c).substring(a.length, a.length+b.length) == b
  }
}
{% endhighlight %}
Each of these grouped properties can be composed into a single set using the include method:
{% highlight scala %}
object MyAppSpecification extends Properties("MyApp") {
  include(StringSpecification)
  include(...)
  include(...)
}
{% endhighlight %}

## Custom Generators
So far we've seen how to verify universal quantifiers for properties over standard types (Int, String, List[Int] and so on so fourth). Now we will see how to implement our own generators (in the form of the Arbitrary type) so we can use them in our property tests.

Suppose you have a trait such as:
{% highlight scala %}
trait myType {
  type T 
  type E 
  def ord: Ordering[E]

  def empty: T
  def isEmpty(t: T): Boolean
  def insert(x: E, t: T): T
}
{% endhighlight %}

We can define a generator for arbitrary instance of myType using for comprehensions, arbitrary primitives (such as on Int) and simple generators such as OneOf and const:
{% highlight scala %}
  lazy val genMyType: Gen[T] = for {
    v <- arbitrary[Int]
    m <- oneOf(const(empty), genHeap)
  } yield insert(v, m)

  implicit lazy val arbMyType: Arbitrary[T] = Arbitrary(genMyType)
{% endhighlight %}

## Execution Semantics

When the check method is executed, ScalaCheck will run the property tests with a set parameter size for each of the generators and incrementally increase this attribute up to a given limit n on each execution. Execution is performed concurrently on a configurable number of workers.

One final point worth noting is how to minimise the number of test cases needed to prove/disprove a property. The ScalaCheck documentation uses the following example to demonstrate the various semantics of the default shrink methods:
{% highlight scala %}
import org.scalacheck.Arbitrary.arbitrary
import org.scalacheck.Prop.{forAll, forAllNoShrink}

val p1 = forAllNoShrink(arbitrary[List[Int]])(l => l == l.distinct)

val p2 = forAll(arbitrary[List[Int]])(l => l == l.distinct)

val p3 = forAll( (l: List[Int]) => l == l.distinct )
{% endhighlight %}
These all correctly falsify the property as one would expect - but the number of tests required to do so varies greatly. This is because p2 & p3 are able to shrink the lists under test down to a minimal failing test to disprove this property, whereas p1 generates larger lists and thus requires more checks to complete.

## Conclusion

To recap, we have touched on Property based testing and how it relates to traditional unit testing techniques, how to write basic properties over types, how to group/compose properties, implementing custom Arbitrary instances, and the execution semantics which govern ScalaCheck's property based testing system.

Thanks for reading - I hope you found this useful! Stay tuned for more insights into Functional Programming, Data Science & Engineering.

