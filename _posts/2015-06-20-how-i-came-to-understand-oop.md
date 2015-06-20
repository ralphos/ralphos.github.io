---
layout: post
title:  "How I came to understand Object Orientated Design"
date:   2015-06-20 11:00:00
categories: OOP
published: true
---


I've been coding for about 3 years now and whilst I certainly don't proclaim to be an expert in object orientated design I thought it would be worthwhile to share my understanding of it.

You may have heard certain coding buzzwords like

> Managing dependencies, coupling, collaborators, dependency injection, composability, single responsibility principle etc..

A few years ago none of this made any sense to me. I had no context in which to understand these concepts since I didn't understand object orientated design principles.

In fact it wasn't until I read [Practical Object Orientated Design in Ruby](http://www.poodr.com/) by Sandi Metz that things started to make sense.

One of the underlying themes from Sandi's book can be summarized as follows

> Something will change.

Sandi goes on to say

> Something in your application code will change and arranging code to efficiently accommodate change is a matter of design. 

It's "aha!" moments like these that make this book one of the best I've ever read. 

So here's my current understanding of what object orientated design is:

Object orientated design is about **managing dependendies** - where objects in your code don't know too much about one another. In other words, it's about decreasing **coupling**.

Coupling can be defined as the degree to which components in a system rely on each other. 

The reason it's bad is because if we're changing one object in our application we may be forced to change it's **collaborators** (or other objects that depend on it) and if we change one object in our system we could break lots of other objects.

Ideally we want to have the lowest form of coupling, otherwise objects in our application are never going to be able to talk to each other. To do so, we can use **dependency injection**.

Dependency injection is just a fancy word for creating an object and passing it's dependency into it's contstructor (which is the 'injection' part) as opposed to explicity referring to it in a class definition.

```ruby
# With dependency injection
class Example
  def initialize(awesomeness)
    @awesomeness = awesomeness
  end
end

# Without dependency injection
class Example
  def initialize
    @awesomeness = Awesomeness.new  
  end
end

```

If we can manage our dependencies by decreasing coupling through the use of dependency injection, we can improve **composability** in our design where we have lots of small objects that adhere to the Single Responsibilty Principle. Our objects will be more reusable and easier to test as a result.

**So what does this mean for me?**

It means, write lots of small objects. Think about how you may abstract more objects (think about objects that can be added to 'interact' with other objects). Agonize over how you might name these new objects. Decide upon which folders these objects will reside in. Write lots of isolated unit tests. And finally, begin to enjoy another level of weird abstract coding euphoria when all your little objects begin to work in harmony and are easy to change when required.

