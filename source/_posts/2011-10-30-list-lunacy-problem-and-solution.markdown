---
layout: post
title: "List Lunacy: Problem and Solution"
date: 2011-10-30 17:53
comments: true
categories: [algorithms]
---

I came across this interesting problem a few weeks ago, and I had a lot of fun
thinking about it and solving it. In this post, I'll cover:

1. The problem statement
2. Approaching the problem and coming to a solution
3. Experimentally verifying the solution with an implementation
4. A proof of the solution for theoretical verification
5. Interesting things I found looking around after solving the problem

## The Problem Statement
Write a function that returns a random node from a linked list of unknown
length in **O(1) space** and **O(n) time**. (Finding a solution in O(1) time is
not only algorithmically interesting, but is also realistic if the linked list
is too large to fit in memory). You may only traverse through the linked list
**once** (i.e. you can't traverse it once to get the length of the list, then
again to pick a node).

<!-- more -->

## The Approach
The key thing to think about while approaching this problem is that the
algorithm has to be ready to terminate at any moment because you don't know
when the list is going to end. The first thing that this tells you is that the
function should always be holding onto a node that it will return if the list
comes to an end. This implies that the function should pick up the first node
that it comes to, and then with every node it traverses, it should replace the
node it is holding with some probability. The probability of replacing the node
that it's holding with the one it's traversing should be decreasing with the
number of nodes seen because as the list gets longer, the probability of
picking up a node gets smaller. The probability also depends on the number of
nodes that have been traversed, but it's not yet clear what the function is.
(Think about why the probability can't depend on something like the number of
times the function replaced the node it's holding or the number of nodes it has
seen since the last replacement).

The second thing to realize will yield the solution to the problem, although
initially it may not be obvious why it works. If we think about the fact that
the algorithm has to be ready at any moment to terminate because it might reach
the end of the list at any moment, then we realize that the algorithm always
has to pick up the *k*th node with probabiliy *1/k*. It's clear that if the
algorithm uses this method, then once the list ends, the algorithm would have
picked the very last node in the list with the correct probability. If the
length of the list were *n*, then the algorithm would pick up the last node in
the list with probability *1/n*. But because the algorithm has to treat every
single node as if it were going to be last node in the list, then it has to do
this for all the nodes in the list!

This seems unintuitive because it isn't clear why all the other nodes also get
returned with the same probability, but at the same time the reasoning appears
to be correct in an odd, nonconstructivist way. Let's write a quick
implementation of this function to get some initial verification that it is
correct, and then afterward we'll do a more rigorous analysis and proof so that
we can see why this solution is correct. Fortunately in this case it doesn't
take very long to write an implementation (at least in a high-level language
like Ruby), so we can do that before proving the correctness of the solution.
It'll be fun, and if we're wrong, the output may give some clues to the correct
solution. But in other cases, it may be more effective to prove that the
solution is correct before diving into an implementation, because then if your
code gives you the wrong answer, you can't be entirely sure that it's your
solution and not your code that is the problem.

## An Implementation

I chose to write a quick implementation in Ruby because it'd be fast to write
up (and also because it's currently my favorite language). I used an array
instead of a real linked list not only because it doesn't change the core of
the algorithm and it saves time, but also because it improves clarity and
avoids possible bugs in dealing with linked lists.

{% codeblock list_lunacy.rb lang:ruby %}
def get_random_node(arr)
  node = nil
  arr.each_with_index do |el, i|
    if rand(i+1) == 0 # returns true with probability 1/i
      node = el
    end
  end
  node
end

a = (0...10).to_a
count = [0] * a.size

1_000_000.times do
  count[get_random_node(a)] += 1
end

p count
{% endcodeblock %}

{% codeblock lang:bash %}
$ ./list_lunacy.rb
[100061, 100070, 100450, 99472, 99986, 99762, 100038, 99957, 100092, 100112]
{% endcodeblock %}

This data yields a mean of 100,000 and a [standard
error](http://en.wikipedia.org/wiki/Standard_error) of 0.2519,
which gives us a good deal of confidence about the correctness of our solution.
Hurray!

## A Proof (no, actually two!)
So now that we have a good experimental verification that our solution is
correct, let's return back and think about why it is that our solution works.
In particular, it isn't yet clear why our method of holding onto a node at all
time and replacing it with the *k*th node with probability *1/k* results in all
the nodes being picked with equal probability. Let's take a look.I'll go
through two different proofs with two different approaches: one that
immediately proves our solution correct for lists of arbitrary length, and
another that proceeds using proof by induction.

### Proof 1: For lists of length *n*
Suppose that we have a linked of length *n*. For the *k*th node in the linked
list, where *k = 1, 2,...,n*, we pick it with probability *1/k* and we hold
onto our previously picked node with probability *(k-1)/k*. In order for the
*k*th node to be picked and held onto until the very end to be returned, the
algorithm must pick node *k*, and then cannot pick any of the successive nodes.
It doesn't matter what the algorithm does for nodes *1* through *k-1* because
all that matters is the last node that is picked by the algorithm.

And so, the probability that the *k*th node is picked and returned by the
algorithm is simply the product of the probability of the *k*th node being
picked and the probabilities that all of the successive nodes are not picked,
which turns out to be:

![](http://brandonkliu.com/public/images/list_lunacy.png)

Since this is true for all *k*, we see that our algorithm indeed does return a
random node with equal probability. The *k*th node initially may be picked with
probability *1/k*, but with every additional node that is traversed, the
probability that it is still held onto "erodes" down to *1/n*.

### Proof 2: A proof by induction
We proceed using proof by induction on the length of the linked list.

**Basis**: The linked list has length 1.

In the case of length 1, our algorithm immediately picks up the first node
(with probability 1), and because the linked list ends there, it returns that
node with probability 1, as desired.

**Induction Step**: Assume our inductive hypothesis true for all linked lists
of length at most *k*, where *k* is greater than 1, and show that our inductive
hypothesis holds for a linked list of length *k+1*.

Suppose that we have a linked list of length *k+1*. By the time the algorithm
has traversed the first *k* nodes in the linked list, the node it is currently
holding onto could be any of the first *k* nodes, each with probability *1/k*,
by our inductive hypothesis. When the algorithm reaches node *k+1*, it could
pick the last node with probability *1/(k+1)*, or it may hold onto the previous
node with probability *k/(k+1)*, which means that if the algorithm held onto
the previous node, it would be holding onto any one of the first *k* nodes,
each with probability *(1/k) * (k/(k+1)) = 1/(k+1)*. Hence, when the algorithm
returns, it will return any of the *k+1* nodes, each with probability
*1/(k+1)*. This completes the induction step.

## Additional interesting information
While writing this blog post, I was googling around, curious to see whether
anyone else had writting on this problem. I ended up discovering that this
problem is actually a special case of the Reservoir sampling problem, which can
be described as follows: Suppose that you have a incoming stream of pieces of
data. You can only see each piece of data once and you don't know how much
total data is coming through. You need to return *k* pieces of data from this
stream, **evenly distributed** from the original stream.

Sounds similar, right? The algorithm for the reservoir sampling problem turns
out to be quite similar to the one we've come up here. If you're interested,
you can read a little bit more on
[Wikipedia](http://en.wikipedia.org/wiki/Reservoir_sampling) or the [Dictionary
of Algorithms and Data
Structures](http://xlinux.nist.gov/dads//HTML/reservoirSampling.html), although
there's not too much information there. But there's a very interesting post on
this reservoir sampling problem and interesting extensions, such as the
weighted reservoir sampling problem and the distributed reservoir sampling
problem here: http://gregable.com/2007/10/reservoir-sampling.html. All very
interesting things to think about.
