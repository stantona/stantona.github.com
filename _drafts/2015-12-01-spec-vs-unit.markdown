---
layout: post
title: "Why I prefer unit style tests over specs"
date: 2015-11-30 16:37
comments: true
categories: [Ruby, Testing]
---

This is a small write up on why I prefer unit to rspec style tests. In short, I prefer **unit** style testing because it guides the design of my code faster than if I were using **specs**.

When I talk about unit style testing, I'll be referring to minitest, as it has been part of Ruby core for a while. Minitest also supports specs, but I've always used rspec.

Both rspec and minitest are fine testing frameworks with different semantics.

It's definitely personal preference which one you use.

## The simple reason

This is a quote from the Minitest [README](https://github.com/seattlerb/minitest/blob/master/README.rdoc):

>rspec is a testing DSL. minitest is ruby.
>
>-- Adam Hawkins, "Bow Before MiniTest"


I find Minitest (unit style testing) is easier to understand since they are plain Ruby classes. There's no magic so there's minimal learning required.

RSpec is a DSL and therefore requires the user to learn the prescribed language. You have to understand how the various blocks are executed and in what order. The code does not reveal this. There are also multiple ways to do the same thing.

For newcomers to Ruby, it doesn't look like normal Ruby. I've observed experienced developers who are new to Ruby take weeks to get up to speed with RSpec.

RSpec's magic is an example of Ruby metaprogramming gone wild.

## The real reason

If you have ever jumped from *spec* style tests to *unit*, you may have found the experience somewhat painful.

>What will I do without *contexts*?

>Dear God, look at the *size* of my test method names?

For complicated classes, I find it easier to test with RSpec. Say you have a large spec covering a fairly complicated class. It could be 500 lines or more. A business logic change is required which point to this class. You could write a context for the change in the spec and away you go.

{% highlight ruby %}

context "when user is not a member of blah group" do
  it { should be true }
end

{% endhighlight %}

But what if the prerequisite of this context are other contexts?

{% highlight ruby %}

context "when this happens" do
  context "when that happens" do
    context "when user is not a member blah group" do
      it { should be true }
    end
  end
end

{% endhighlight %}

This is easy, a spec has been added under nested contexts. If you're familiar with RSpec, this is type of setup is common.

Now what if you had to apply the above to a **unit** test method?

{% highlight ruby %}
def test_when_this_happens_and_that_happens_and_user_is_not_a_member_of_blah_group
  do_this
  do_that
  assert expectation == true
end
{% endhighlight %}

_I feel uncomfortable when I write a test like the one above._ The method name is too long and could be difficult to understand for those unfamiliar with the code.

And this is precisely why I like **unit tests**. This method name defines my test. A method name that is awkward to define because of various contexts and conditions prompts me to ask myself whether I've designed my code correctly. Defining the test itself gives immediate feedback to the complexity and design of your code under test.

_Writing unit tests add more friction than adding nested contexts in RSepc, because for me, they constantly prompt me to question my code_.

To overcome this problem, an approach I use is to create classes for contexts. How these classes are organized can be tricky. Do I nest the classes within each? Use inheritance? This is essentially identical to using the `context` block in RSpec, but generally you need to write more code. This can reduce friction where it seems appropriate. A lot of times it adds to it.

When the friction caused by an unwieldy test method name, and then trying to shove them into context classes, a problem with the code under test is often realized. If I find myself writing context classes, why not break the code under test into different objects themselves which can be tested in isolation? Applying structural changes to your tests to improve simplicity can reveal new objects which become natural collaborators with the main object under test.  I'm ultimately aiming for very short test method names revealing well defined behaviour on a class.

##Conclusion

I find **unit tests** are harder to write, but I ultimately end up with cleaner code and cleaner tests. The extra friction prompts me to reexamine the code I'm testing. Since minitest gives you plain ruby objects, it seems natural that you would test object orientated code with object orientated code. Unit tests are unforgiving in revealing problems with the code you're testing.

In fairness, and I failed to mention this above, rspec can reveal the same problems too. If your rspec file is too long, filled with nested contexts which are difficult to follow, you have a similar problem, and I feel the same friction. However, rspec makes it so easy to add a spec that is easier to digest. It doesn't provide the same awkwardness as a unit test does.

I don't advocate one test framework over the other, as they both do the job well. If you've never tried unit style tests, I recommend doing so, as it reorientates the way you think about your code and your tests. This way of thinking can then be easily applied to rspec.
