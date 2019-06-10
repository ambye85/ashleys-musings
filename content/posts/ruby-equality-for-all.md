---
title: "Ruby - equality for all"
date: 2018-10-26T12:00:00+01:00
draft: false
---
Having predominantly programmed in Java, when creating classes I’m used to overriding equals() and hashcode(). During an exercise converting Kent Beck’s Money Example from Java to Ruby, to help me learn Ruby and improve my TDD skills, I overrode the equivalent methods (`equal?` and `hash`) in one of my classes.

The need for doing this arose because I wanted to use a class, `Point`, as the key to a hash. Looking up the value from the hash would be done by instantiating a new instance of `Point` and passing this as the key. Having overridden these methods, I was really pleased when the result of this lookup returned nil.

After several hours of head scratching and research, I eventually discovered that there are four types of equality in Ruby. Although I had overridden how my object’s hash was computed, I had not overridden the method that calls this, eql?.

As a not for my future self, the four types of equality in Ruby are as follows:

`equal?` — used to determine if two objects are exactly equal. It shouldn’t be overridden.

`==` — used to compare two objects for equality and should be overridden to give class specific context to equality (as a bonus, `!=` comes for free).

`eql?` — used to compare hash equality. It is often overridden to call `==` but in reality it should call an overridden `hash` method.

`===` — used for case equality.

I couldn’t find a single reference in the Ruby docs about these different types of equality. The following posts on [StackOverflow](https://stackoverflow.com/questions/7156955/whats-the-difference-between-equal-eql-and) and [Skorks](https://www.skorks.com/2009/09/ruby-equality-and-object-comparison/) were helpful in helping me to understand these different equality types.
