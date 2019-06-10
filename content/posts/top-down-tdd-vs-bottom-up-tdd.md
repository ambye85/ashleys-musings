---
title: "Top down TDD vs bottom up TDD"
date: 2018-11-27T12:00:00+01:00
draft: false
---
One of the benefits of undertaking a software crafting apprenticeship at 8th Light is having opportunity to experiment with new technology stacks and new techniques. Having been asked to implement a console game of Tic Tac Toe in Java, a language I am relatively familiar with, I decided to take the opportunity to give top down TDD a spin.

What is top down TDD? There are two schools of thought when it comes to TDD. Before I look at each of these, I want to remind readers that either approach requires some element of up front design. I’m not talking about listing out every single class, method, etc., but enough so that you have a feel for where the system is going. With that said, let’s look at the two approaches.

### Bottom Up TDD

First is bottom up TDD, which some may know as the ‘Detroit School’ of TDD. With bottom up TDD, we start by writing tests and implementation for small aspects of the system. The aim is to grow the design through a process of refactoring, generalising the codebase as tests become increasingly specific.

With Tic Tac Toe, I’d probably begin with placing a mark on a game board and growing upwards to incorporate the logic of the game, such as taking turns, determining the result, adding varying difficulties of computer player, different board sizes, etc. I’d then repeat this process for the user interface, starting with playing a single game and growing upwards to include features such as changing game settings, i.e., a player’s mark, or player’s type.

This approach makes use of mocks when needed, either through a mocking framework or by writing manual mocks. The API would gradually develop and change as new insights became apparent. Every part of the code would be covered with many tests, with components at a higher level of abstraction most likely depending on the implementation of lower level concerns. This makes refactoring, and thus the generalisation of the codebase, safe because of the vast number of tests.

### Top Down TDD

Second is top down TDD, which some may know as the ‘London School’ of TDD. Using this approach, development begins at the very top of the system’s architecture and is grown downwards. The aim is to progressively implement increasing functionality, one section of the system at a time. As a result, a reliance on mocks is required to simulate the functionality of lower level components.

For a game of Tic Tac Toe, this would probably mean starting with the game and mocking out its various responsibilities, such as getting a player’s move, processing it and then outputting the updated game. At each consecutive lower level, the layer below that would be mocked out and the system components ‘drown downwards’ until the required functionality is implemented. Developing the user interface would follow a similar pattern.

The top down approach makes judicious use of mock objects. It is easy to write the API you wish you could have, because you’re starting from the top and injecting collaborators as required. The tests for the code tend to cover the development and verification of the API, meaning that components with higher levels of abstraction do not have the safety net of the tests for those in the lower level. Whilst refactoring is not impossible, the nature of the tests means that there is likely to be less coverage than with the bottom up approach — integration/acceptance tests become vital to ensure everything continues to work cohesively.

### My experience with Top Down TDD

So how did I approach implementing my game of Tic Tac Toe using the top down approach? Much like I suggested above, I started with the API I wanted for my game class. This basically boiled down to getting a player’s move, updating (or not) the game board and outputting the (possibly) updated board. I wrote manual mocks to simulate each of these responsibilities and merrily proceeded on my way. Next up, I decided to implement the game updating responsibility and again mocked out classes with the varying different responsibilities. I carried on this way for a while, then encountered two problems. First, I had split up a lot of responsibilities and found that they all eventually came back to one class — imagine a class diagram which forms the shape of a diamond. In trying to split these responsibilities into their own classes, I realised that my API was completely wrong and needed changing. To do this, I needed to go back to the beginning and rewrite most of what I had done previously.

### My opinion

I think the top down approach is harder than the bottom up approach. The following list of pro’s and con’s sums up my experience:

**Top Down Pros:**

1. Lot’s of little classes, meaning functionality, and therefore responsibility, is neatly encapsulated.

1. Easy to mock due to the reliance on mocking out lower layers..

1. Results in a well defined and clean API.

**Top Down Cons:**

1. Hard to change API if the wrong abstractions are chosen.

1. Making changes mean re-writing lots of mocks and tests — makes mocking frameworks favourable.

1. Fewer tests, therefore less confidence that refactoring will not break some other functionality.

### Summary

I enjoyed taking time to experiment with the top down TDD approach. I’m not fully sold on its merits, but do see value in combining it with the bottom up school. I probably should have spent a bit more time thinking about lower level responsibilities early on. By focusing on the wrong thing, I ended up with a whole bunch of classes that formed a diamond of responsibilities that all went back to the game class.

Did I finish my game of Tic Tac Toe? Yes, but only after heeding Kent Beck’s call to throw away useless code and start again when required. I went back to my comfort zone and developed from the bottom up — sorry London, but in this instance I prefer Detroit! However, the experience did lead me to drive my implementation towards the API I wanted from my botched attempt at top down TDD. Overall, it was a valuable exercise and I will certainly try to find time to give another project a go using this method.
