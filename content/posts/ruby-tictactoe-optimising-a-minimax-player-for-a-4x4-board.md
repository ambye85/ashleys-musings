---
title: "Ruby tic-tac-toe - optimising a minimax player for a 4x4 board"
date: 2018-11-12T12:00:00+01:00
draft: false
---
If you’ve been following my previous posts, you’ll know that I’ve been learning Ruby and developing a game of TicTacToe in the process. I’ve just finished implementing a new set of requirements to allow games to be played on a 4x4 board, against an unbeatable computer player. With some algorithmic optimisations, the computer player can choose a first move in less than a second. I’ll explain how I achieved that in the remainder of this post.

I’d already implemented the unbeatable computer player for my 3x3 board, but hadn’t completed one of the non-functional requirements — for a move to be played in less than 200ms. It was taking about 8s for the computer player to make a move on an empty 3x3 game board. I tackled this requirement as part of the process of implementing the 4x4 board.

When implementing these features, I tried to adhere to Kent Beck’s guideline of [make it work, make it right, make it fast](http://wiki.c2.com/?MakeItWorkMakeItRightMakeItFast). This led me to the following implementation for determining whether a given board had been won, tied, or was still being played:

```ruby
def game_over?(board)
    win?(board) || tie?(board)
end

private

def win?(board)
    p = board.positions
    winning_combination?(p[0], p[1], p[2]) ||
    winning_combination?(p[3], p[4], p[5]) ||
    winning_combination?(p[6], p[7], p[8]) ||
    winning_combination?(p[0], p[3], p[6]) ||
    winning_combination?(p[1], p[4], p[7]) ||
    winning_combination?(p[2], p[5], p[8]) ||
    winning_combination?(p[0], p[4], p[8]) ||
    winning_combination?(p[2], p[4], p[6])
end

def winning_combination?(a, b, c)
    a == b && b == c
end

def tie?(board)
    board.available_positions.empty?
end
```

It’s about as fast as it can be because it’s purely reading object references from the array of board positions and using binary logic to determine whether a win or tie has been achieved. However, it’s not necessarily obvious what is being done and it’s certainly not going to work for my 4x4 board, since it assumes a fixed sized array. These methods are in the GameRules class, so there is an amount of coupling between the board and the methods that describe the rules of the game — since these are related bits of functionality, I’m happy with this level of coupling. If I needed to get rid of it, I’d move these evaluation functions into the Board class. However, I did need to refactor the win?(board) method to be able to deal with a 4x4, or NxN, board.

I decided to use a constructor argument to the board initialiser to set its size, with a default value of 3. This prevented my tests for a 3x3 board from breaking and avoided a lot of duplication that would have been needed if I’d chosen subclassing:

```ruby
def initialize(size = 3, positions = (1..size*size).to_a)
    [@size](http://twitter.com/size) = size
    [@positions](http://twitter.com/positions) = positions
end
```

I also like that once a method parameter has been declared, it is available to the following method parameters. This makes it trivial to set the empty positions array based on the size parameter whilst still allowing updated positions to be passed into the constructor. The only change to my codebase was an amendment to the `place_token(token, position)` method due to the addition of the size parameter:

```ruby
def place_token(position, token)
    positions = Array.new([@positions](http://twitter.com/positions))
    positions[position — 1] = token if available_positions.include?(position)
    Board.new([@size](http://twitter.com/size), positions)
end
```

If you’ve been following along previously, you might notice that I’ve moved some functionality out of the Board class. I’m not going to go through all of the changes, but the responsibility for determining whether the position is valid and handling invalid positions has been moved into the `Game` class.

Now that we have a working 4x4 board, we need to address how to compute whether or not a win has been achieved. To do this, I added a public method to the Board that would compute the possible winning positions and return an array of array’s with the current values of the board:

```ruby
def possible_winning_positions
    get_rows
        .concat(get_columns)
        .concat(get_diagonals)
end

private

def get_rows
    [@positions](http://twitter.com/positions).each_slice([@size](http://twitter.com/size)).to_a
end

def get_columns
    get_rows.transpose
end

def get_diagonals
    [get_top_left_bottom_right_diagonal, get_top_right_bottom_left_diagonal]
end

def get_top_left_bottom_right_diagonal
    get_rows.select.with_index.map { |row, i| row[i] }.to_a
end

def get_top_right_bottom_left_diagonal
    get_rows.select.with_index.map { |row, i| row.reverse[i] }.to_a
end
```

I also changed the evaluation method in `GameRules` so that it could make use of this array of possible winning positions:

```ruby
def win?(board)
    win = false
    p = board.possible_winning_positions
    p.each do |combination|
        win = winning_combination?(combination) if win == false
    end
    win
end

def winning_combination?(combination)
    combination.uniq.length == 1
end
```

Here, I used the property that a winning combination would only consist of one value, either X or O, so the length should equal one if it is a win.

Great, let’s check it works. Oh. Well, the functional tests all passed, but the computation time for a 3x3 board had increased to about 20s on my machine. The computation time for a 4x4 board? I got bored waiting after about 20 minutes!

Hmm. Having made it work and made it right, I now had to make it fast. I had a couple of strategies up my sleeve for doing so:

1. Use [alpha/beta pruning](https://www.geeksforgeeks.org/minimax-algorithm-in-game-theory-set-4-alpha-beta-pruning/) in my minimax algorithm.
2. Limit the depth of searches in my minimax algorithm.

I addressed both of these optimisations at the same time. Alpha/beta pruning essentially means that the algorithm stops searching down a particular tree of possible moves if it has already found the best possible move for that branch. This reduces the number of computations required, so improves the performance of the algorithm. By limiting the depth of searches, performance is also improved — before a certain number of moves have been made, it doesn’t matter where the computer plays. For a 3x3 board, this is 5 moves. Let’s take a quick look at the maximise method of the minimax algorithm to support these optimisations:

```ruby
def minimax(board, depth, is_maximising, alpha, beta)
    return score(board, depth) if [@rules](http://twitter.com/rules).game_over?(board) || depth >= 5

    if is_maximising
        maximise(board, depth, is_maximising, alpha, beta)
    else
        minimise(board, depth, is_maximising, alpha, beta)
    end
end

def maximise(board, depth, is_maximising, alpha, beta)
    max_score = -1000
    board.available_positions.each do |position|
        b = board.place_token(position, [@token](http://twitter.com/token))
        score = minimax(b, depth + 1, false, alpha, beta)
        max_score = [max_score, score].max
        alpha = [max_score, alpha].max
        break if beta <= alpha
    end
    max_score
end
```

The minimise function is broadly similar to maximise. However, we can see that the guard clause of the recursive minimax function has been updated to terminate once the depth limit has been reached. We are also breaking out of the recursive loop of available positions if there are no better scores available in the tree.

With these optimisations in place, I had hoped that game play times would be reasonable. Alas, that was not the case. For an empty 3x3 board, computation time was back to around 8s, whilst for a 4x4 board it was still taking just over forever. An improvement, but not brilliant either.

The next optimisations that sprang to mind were:

1. Only checking for a win after sufficient moves had been played.
2. Memoizing the winning positions to reduce array assignment computations each time the method is called (and it’s called a lot).

The minimum number of moves needed to be made before a win can be achieved is 5 and 7 for a 3x3 and 4x4 board respectively. That can be expressed as the total number of positions minus twice the board size minus one. Or:

```ruby
def enough_moves_made_for_a_win?(board)
    board.available_positions.length > board.positions.length — ((2 * board.size) — 1)
end
```

That left one final question — how to memoize the winning positions, since they need to be looked up each time the board changes? I spent a bit of time at the beginning of this post looking at my previous implementation of the win? method. What made this efficient was the ability to do a direct read from the array, given the correct combination of indices for known winning positions.

With this in mind, I updated the GameRules to be initialised with an empty instance of the game’s board. This is used to call the `compute_winning_indices` method on the board instance and store the result as an instance variable within the game rules.

```ruby
def compute_winning_indices
    row_indices = compute_row_indices
    column_indices = compute_column_indices(row_indices)
    diagonal_indices = compute_diagonal_indices(row_indices)
    row_indices
        .concat(column_indices)
        .concat(diagonal_indices)
        .flatten
end

private

def compute_row_indices
    [@positions](http://twitter.com/positions).select.with_index.map { |p, i| i }.each_slice([@size](http://twitter.com/size)).to_a
end

def compute_column_indices(row_indices)
    row_indices.transpose
end

def compute_diagonal_indices(row_indices)
    compute_top_left_bottom_right_diagonal(row_indices)
        .concat(compute_top_right_bottom_left_diagonal(row_indices))
end

def compute_top_left_bottom_right_diagonal(row_indices)
    row_indices.select.with_index.map { |row, i| row[i] }.to_a
end

def compute_top_right_bottom_left_diagonal(row_indices)
    row_indices.select.with_index.map { |row, i| row.reverse[i] }.to_a
end
```

There’s note much difference between these methods and those that were returning the values of the relevant positions. But now, instead of computing all possible winning moves and the considerable number of assignments to arrays that my previous method used, the program was doing this just once. Each time `win?` is called, these indices are mapped to the positions of the current board and then evaluated. Assignment to one array being significantly more computationally effective than many assignments:

```ruby
def win?(board)
    win = false
    unless enough_moves_made_for_a_win?(board)
        possible_winning_positions(board).each do |combination|
            win = winning_combination?(combination)
            break if win
        end
    end
    win
end
```

With these two additional optimisations in place, execution time for a 3x3 board decreased to under 200ms. For a 4x4 board, execution time is consistently less than 1s. I’m happy with that. If I were going to make any further tweaks to these optimisations, I’d start with the following:

1. Calculate the depth limit based on board size.
2. Select an available position at random until the critical number of moves have been played.

I found that following the principle of make it work, make it right, make it fast, really helped develop this algorithm. Had I not taken this approach, I’m not sure I’d have ever gotten to a point where I had a working implementation to be improved, as I’d have been so hung up trying to make it run quickly. By taking this approach, I was able to isolate possible areas for optimisation whilst maintaining readable code.

To see the full source code for this post, visit my [GitHub repository](https://github.com/ambye85/tictactoe-ruby/tree/4x4-board).
