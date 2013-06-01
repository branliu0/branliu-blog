---
layout: post
title: "Simulating the Life and Death of Last Names: Initial Findings"
date: 2013-05-31 21:50
comments: true
categories: 
---

A few months ago, my friend put forth an interesting question: What happens
to the number of last names over time? It is common to hear about a family
name dying with a last descendant who never had any children. That suggests
that it is possible that in the future, we will all have just one of a handful
of last names, as most of them will have died off. Yet, I would expect that if
it could happen, it should have happened by now, or at the very least, I would
be able to discern some kind of sign suggesting a trend toward fewer last
names. But the number and diversity of last names on Earth is quite
remarkable, and it's still a uncommon thing for two unrelated people to meet
and have the same last name.

Anyway, in the spirit of curiosity, I decided to write a small Ruby script
that would simulate the progression of last names over many generations, just
to see what the end result might be. I had an enjoyable time exploring the
problem, and ended up with some interesting results.

<!-- more -->

## Basic assumptions for my model

First, I had to model the way that last names are passed down in order to
write a simulation. Here are the basic assumptions that I made:

* Names are always inherited from the father
* Boys and girls are born with equal probability
* Each generation, every woman gives birth to at least two children
* Each woman procreates with exactly one man randomly sampled from the male
  population
* A man can procreate with multiple women, or with none. A man is randomly
  selected for each woman, so a man could be selected multiple times, or not
  at all.

I also decided to cut off each simulation after 1500 generations, which
was a reasonable amount of time for my computer to run a few simulations, and
would also effectively simulate about 45 thousand years of human life, under
the assumption that a generation is 30 years.

Of course, many of these assumptions are not particular representative of the
real world, but they provide a set of rules that we can start experimenting
with.

## Observations

I played around with the problem a bit, trying different starting population
sizes and different ways to determine the number of children a couple would
have. Here's what I found.

### Last Names in the End Game
* The number of last names would quickly dwindle, but often by the 1000th
  generation they would stabilize at some number between 2 and 8. Over 100
  separate trials, here is the histogram of the number of last names in
  existence after 1500 generations.

![Histogram for number of remaining last names](http://chart.apis.google.com/chart?chxr=0,2,8|1,0,30&chxs=0,676767,11.5,1,lt,676767|1,676767,11.5,0,lt,676767&chxt=x,y&chbh=a&chs=400x273&cht=bvs&chd=s:Qxr0USE&chdlp=b&chtt=Histogram+for+number+of+remaining+last+names)

### Population Growth

* If I set the number of children per couple at two, then the population stays
  roughly stable, with some moderate fluctuations due to randomness in the
  gender of the children being born. If I have a starting population of 50 men
  and 50 women, then they die out pretty quickly because the population runs
  out of men within a few generations.
* So in order for an initial population of 50 to survive, I have to set the
  number of children to be greater than 3, to ensure that they don't die out
  due to random fluctuations. But because three children are being born for
  every two adults, the population experience exponential growth, and the
  population size soon becomes too large for my computer to handle.
* I end up settling for an intial population of 5000 men and 5000 women, which
  is large enough to withstand the random fluctuations to survive.

## Analysis

The biggest result of this vastly simplified model is that the last names
indeed diminish down to some small number of last names that becomes stable.
It is interesting to see this, because it suggests that it is a potential
outcome, although my intuition says that it's unlikely due to the unrealistic
assumptions of the underlying model.

On the other hand, some estimates say that 85 percent of China's population
have one of 100 surnames. After all, the Chinese people do call themselves
"Old 100 surnames" (http://en.wikipedia.org/wiki/Baixing). But doesn't seem to
be the case in most parts of the world.

It isn't immediately clear why in particular the model doesn't match what I've
observed in the real world. If it's one or more of my underlying assumptions
for the model, it isn't clear which of them result in the difference.

It is certainly possible that my simulation, which approximates 45,000 years,
does actually predict the future, and that our society is still too early in
the process to have reached the end state. After all, I don't think that the
concept of last names have existed in society for more than a few thousand
years.

In particular, it is interesting to note a significant flaw in my model. Due
to the way that I've simulated the pairing of couples, a large portion of men
actually don't get a chance to mate at all in each generation! When the
population of men and women are equal in size, the expected proportion of the
male population that doesn't get picked by any woman is roughly *1/e*, or
0.368. If there are *n* men and *n* women, then the probability that a man is
not picked by any woman is simply *((n-1)/n)^n*, which approaches *1/e* as *n*
gets large.

## Further Exploration

When I have time, I would like to spend some more time with this problem to
better answer the questions that still remain. Here are some potential avenues
for exploration.

* Devising and implementing a more accurate model of how couples are actually
  formed, so that there isn't a significant portion of the male population
  that is left out each generation.
* Experiment with desirability: Give each person some measure of desirability,
  which affects their rate of procreation and influences their children's
  desriability.
* Play with different lengths of simulations. Perhaps end the simulation
  only after the number of last names stay stable for some number of
  generations. Maybe all simulations will eventually end with two last names,
  or perhaps there could be different stable outcomes.
* Play with randomly adding new last names throughout the process

I may post a followup post at some point with some further results after
I spend some more time with this experiment!

## The code

{% codeblock lang:ruby %}
#!/usr/bin/env ruby

def simulate(num_start_people = 5000)
  males = (1..num_start_people).to_a
  females = (1..num_start_people).to_a
  new_males = []
  new_females = []
  step = 0

  loop do
    step += 1
    females.size.times do |i|
      name = males.sample
      num_children = males.size > 5000 || rand > 0.2 ? 2 : 3
      num_children.times do
        if rand(2) == 1
          new_males << name
        else
          new_females << name
        end
      end
    end
    males = new_males.clone
    females = new_females.clone
    new_males.clear
    new_females.clear
    break if step > 1500
  end
  males.uniq.count
end

100.times do
  puts simulate
end
{% endcodeblock %}
