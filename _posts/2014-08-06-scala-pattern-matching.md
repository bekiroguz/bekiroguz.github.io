---
layout: post
title: Scala Pattern Matching? Be careful :)
comments: true
---

A few days ago I experienced one of the scala pattern matching rules in action. Luckily I already had a unit test that prevented this very hidden bug :) This is the test and the first version of the code which was perfectly working:

{% highlight scala linenos %}
import org.scalatest.{ShouldMatchers, FunSuite}
class LongToOptionTest extends FunSuite with ShouldMatchers {
  test("convert zero orderId to None") {
    LongToOption(0) should be(None)
  }
  test("convert non-zero orderId to Some") {
    LongToOption(1) should be(Some(1))
  }
}

object LongToOption {
  def apply(orderId: Long) = orderId match {
    case 0 => None
    case id => Some(id)
  }
}
{% endhighlight %}

After I saw both of the tests are green I wanted to make a very small change in the code and defined `noOrderId` constant to make the code more readable. And guess what happened? The second test started failing after that.

{% highlight scala linenos %}
object LongToOption {
  private val noOrderId = 0
  def apply(orderId: Long) = orderId match {
    case noOrderId => None
    case id => Some(id)
  }
}
{% endhighlight %}

While developing in Scala, thinking with a Java mindset can cause these kind of bugs. You could easily assume that using a variable name starting with a capital or non-capital letter does not make a difference, but in Scala it does! Well, this is not completely correct, let me clarify a bit more: "Starting with capital or non-capital letter matters inside pattern matching block."

`case noOrderId => None` actually means match any value and assign it to a variable called noOrderId to be used later whether you already have the same variable defined before or not. So the code was always matching the first pattern whatever the input is. To make our lives less miserable, Scala allows usage of some variables or constants inside pattern matching but only if they start with a capital letter :) Then it tries to find a variable with the same name and does not compile if cannot find. So, the corrected code will be as following:

{% highlight scala linenos %}
object LongToOption {
  private val NoOrderId = 0
  def apply(orderId: Long) = orderId match {
    case NoOrderId => None
    case id => Some(id)
  }
}
{% endhighlight %}
