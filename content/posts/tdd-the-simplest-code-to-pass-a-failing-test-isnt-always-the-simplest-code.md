---
title: "TDD - the simplest code to pass a failing test isn't always the simplest code"
date: 2018-10-22T12:00:00+01:00
draft: false
---
In my previous post, I wrote about setting up a simple Ruby project structure and getting a 'Hello, world!' unit test to pass. The next step on my journey to learn Ruby is to complete the coin change kata. For readers not familiar with this kata, the premise is simple: for a given monetary amount in pence produce a list containing the smallest possible number of coins needed to have that amount. e.g., `amount = 388` yields a result of `[200, 100, 50, 20, 10, 5, 2, 1]` or if `amount = 43`, get `[20, 20, 2, 1]`.

Kata’s are a great way to learn a new language because they are relatively simple and quick to complete. In an ideal world, I’d practice the same kata once a day for a few days in a row as they help to develop muscle memory. However, I want to use this post to explore one of the key points to effective test driven development:

**Write the simplest possible code that will make a failing test pass.**

I’d not attempted this kata previously, so had no preconceived ideas about the order in which I should approach writing tests. As Uncle Bob says, it’s good to start with the most degenerate test case you can think of, so I started with `amount = 0, -1 and nil`:

```ruby
require 'rspec'
require 'coin_changer'

describe CoinChanger do
  context "when a non-value is specified" do
    it "returns a list containing zero" do
      assert_changed(0, [0])
      assert_changed(-1, [0])
      assert_changed(nil, [0])
    end
  end
end

def assert_changed(amount, expected)
  changer = CoinChanger.new()
  expect(changer.change(amount)).to eq expected
end
```

Continuing along these lines, I then started with the simplest possible tests — when `amount = 1, 2, 5, 10, 20, 50, 100 and 200`. Next, I started to tackle the problem of calculating amounts requiring different valued coins, such as 3, 4, 27, etc. Following this logic through, I ended up with the following implementation:

```ruby
class CoinChanger
  COINS = [200, 100, 50, 20, 10, 5, 2, 1]

  def change(amount)
    if (amount == nil or amount <= 0)
      [0]
    else
      build_list_of_coins(amount)
    end
  end

  def build_list_of_coins(amount)
    coins = []
    amount_remaining = amount
    COINS.each do |coin|
      number_of_coin = amount_remaining / coin
      coins = add_coins(coins, coin, number_of_coin)
      amount_remaining = amount_remaining % coin
    end
    coins
  end

  def add_coins(current_coins, coin, number_of_coin)
    coins = current_coins.dup
      for i in 1..number_of_coin do
      coins.push(coin)
    end
    coins
  end
end
```

I hope it’s obvious to see that, despite trying to follow clean coding guidelines as best I could (such as small, descriptive methods), this is not very good code. Why? Because I wrote my tests in the wrong order.

Let’s take a look at the final solution when I tested for `amount = 0, 1, 2, 3, 4`, etc:

```ruby
class CoinChanger
  COINS = [200, 100, 50, 20, 10, 5, 2, 1]

  def change(amount)
    if amount > 0
      compute_coins_for_change(amount)
    else
      [0]
    end
  end

  private def compute_coins_for_change(amount)
    coins = []
    COINS.each do |coin|
      while amount >= coin
        amount -= coin
        coins.push(coin)
      end
    end
    coins
  end
end
```

Apart from a guard clause to deal with the case when `amount = 0` (this could be extended to include negative numbers, nil and even decimal amounts) the bulk of the implementation is in the `compute_coins_for_change(amount)` method. This is a much simpler implementation consisting of two loops. The first, present in both examples, takes each coin and does some processing with it before moving onto the next. The second simply keeps subtracting the coin value from the amount until the amount is less than the coin value, and for each subtraction it adds an occurrence of that coin to the final result list.

Why did the first example turn out so differently? I think it is because test order is important. Having written the tests for each of the single-coin values, it was my responsibility to keep those tests passing when writing the code needed to make the tests pass for the multi-coin values. I was therefore at an immediate advantage when I attempted the kata for a second time but wrote the tests in total value order, despite trying to write the simplest code possible to make the tests pass in both examples.

It’s this idea of writing the simplest code possible that I want to return to. I’ll use my second example to highlight what I mean. To pass the test for `amount = 0`, all we need is the following code:

```ruby
def change(amount)
  [0]
end
```

To make this pass when `amount = 1 and 2`, we can make the following simple alteration:

```ruby
def change(amount)
  [amount]
end
```

Making the tests pass when `amount = 3` is not so simple. The simplest change I could think of to make the tests all pass is:

```ruby
def change(amount)
  if amount > 0
    coins = []
    if amount >= 2
      amount -= 2
      coins.push(2)
    end
    if amount == 1
      coins.push(1)
    end
    coins
  else
    [0]
  end
end
```

At this point, I think it’s quite clear that this is not a simple change. In fact, I’ve pretty much completely re-written the entire method at this point. I’ll come back to these first few steps in a moment, but for now, let’s finish the example through and look at the change from `amount = 3` to `amount = 4`:

```ruby
def change(amount)
  if amount > 0
    coins = []
    while amount >= 2
      amount -= 2
      coins.push(2)
    end
    if amount == 1
      coins.push(1)
    end
    coins
  else
    [0]
  end
end
```

Here, we are back to a simple change. if `amount >= 2` transforms into `while amount >= 2`.

You can probably already see a pattern forming, but we’ll take a look at the next value, `amount = 5`, just to be certain that it exists:

```ruby
def change(amount)
  if amount > 0
    coins = []
    while amount >= 5
      amount -= 5
      coins.push(5)
    end
    while amount >= 2
      amount -= 2
      coins.push(2)
    end
    if amount == 1
      coins.push(1)
    end
    coins
  else
    [0]
  end
end
```

Indeed there is a pattern emerging. It’s more obvious if we take the if `amount == 1` condition and re-write it to match the previous two while loops:

```ruby
while amount >= 1
  amount -= 1
  coins.push(1)
end
```

We can refactor these while loops into a single while loop:

```ruby
[5, 2, 1].each do |coin|
  while amount >= coin
    amount -= coin
    coins.push(coin)
  end
end
```

From here, the final code should be obvious: iterate through each of the valid coin amounts. This can be tested with the following three tests:

```ruby
assert_swaps(88, [50, 20, 10, 5, 2, 1])
assert_swaps(98, [50, 20, 20, 5, 2, 1])
assert_swaps(388, [200, 100, 50, 20, 10, 5, 2, 1])
```

We could have arrived at the same solution if we had started the initial `amount = 0, 1 and 2` implementations slightly differently.

For `amount = 0`:

```ruby
def change(amount)
  [0]
end
```

For `amount = 1`:

```ruby
def change(amount)
  if amount > 0
    [1]
  else
    [0]
  end
end
```

For `amount = 2`:

```ruby
def change(amount)
  if amount > 0
    coins = []
    if amount == 2
      coins = [2]
    end
    if amount == 1
      coins = [1]
    end
    coins
  else
    [0]
  end
end
```

The changes between each of these steps more intuitively lead to a refactoring into a for-each and while loop based solution. The only difference between this code and my solution for `amount = 3` is the `if amount == 2` conditional, which would become `if amount >= 3`.

Writing the solution this way means that we can much more easily find patterns in out code, can avoid large 'unexplained' changes to our codebase, and can continue to write the simplest code possible to make our tests pass.

Except we can’t. Because writing the first few tests in this way immediately violates this rule. The approach I took in the first example was the simplest approach. It took in all the degenerate cases and then the simplest test cases before moving onto the more complex scenarios. I find that this raises more questions, which I don’t have time to answer today:

1. How do we know when not to order our tests from degenerate to simple to complex?

1. How do we know when not to write the simplest implementation to make the tests pass?

1. How do we determine if our implementation is the best possible or if we’ve just failed at 1 and 2?

1. Do I really understand what it means to write the simplest code to make a failing test pass?

Perhaps, for the time being, I should just try and write nearly the most simple test and passing code I can?

*The code for each of my examples (plus a sneaky third example, which was actually my second attempt) in my [GitHub repository](https://github.com/ambye85/coin-changer-kata).*
