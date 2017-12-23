---
layout: post
title:  "Ruining fun with Haskell"
date:   2017-12-23 20:38:30 +1100
categories: haskell
---
Recently, on a whim, I downloaded a game for my phone. It is a game comprised of series of puzzles called *[Calculator: The Game](https://play.google.com/store/apps/details?id=com.sm.calculateme)*. Each level’s objective is to turn your starting number into the goal number within a limited number of moves. The player is given a set of operations that can be performed against the number. It’s a decent little puzzle game.

There was one level that had me particularly stuck, where the objective was to get from 34 to 3 in 5 moves using the operations -5, +8, /7 and \*-1. After many attempts that would have achieved the goal in 6 moves, I decided that I didn’t care about enjoyment and went over to my computer, opened up the Glasgow Haskell Compiler's interpreter and, after some trial and error, managed to type the following:
```haskell
Prelude> import Data.List (permutations)
Prelude Data.List> map (scanl (\x f -> f x) 34) (permutations [(subtract 5),(+8),(/7),(negate)])
```
This line creates a list of all permutations of the list of partially applied functions, and then takes each of those lists of functions and applies them in order, starting at the base number. Since a scan is being used instead of a fold, the steps along the way are shown, so when looking at the end result it is simple to deduce the actions that are being applied.

After formatting the output, I noticed that none of the lists of transformations ended in the goal number of 3. While the lists had 5 elements, they all started with the starting value, and so it was apparent that I was falling one move short of the alloted 5.

Sure enough, one of the 16 items in the list ended in a -3.0, so it was just a matter of following the steps in that list of operations and then performing a (\*-1).
```
[34.0,29.0,-29.0,-21.0,-3.0]
```
I had figured out how to beat the level, but I know I didn’t have the program spit out the complete solution. I didn’t want to just get the permutations of the array, I wanted to get permutations with repetition of a certain size. This can be achieved in Haskell using replicateM, which takes an Int representing the size of each list that you want returned and a list of what you want permuted with repetitions. This returns a list of k^n lists, where k is the length of the list that you want permuted and n is the length of each list.
```haskell
Prelude> import Control.Monad (replicateM)
Prelude Control.Monad> replicateM 2 ['a','b','c']
["aa","ab","ac","ba","bb","bc","ca","cb","cc"]
```
If replicateM is supplied with the number of moves and the functions to apply and then the new lists of actions are each applied to the starting number, in the same way as was done before, then a set of all possible moves that can be done against this puzzle will be returned. The right answers will be contained in there somewhere, so as a final step the answers that don’t end in the goal number should be filtered out.

What we end up is this:
```haskell
import Control.Monad (replicateM)
calculatorGameSolver :: (Num a, Eq a) => a -> a -> Int -> [a -> a] -> [[a]]
calculatorGameSolver goal start moves actions =
  filter (\x -> last x == goal) $
    map (scanl (\x f -> f x) start) $
    replicateM moves actions
```
Which if called returns this:
```haskell
Prelude> calculatorGameSolver 3 34 5 [(subtract 5),(+8),(/7),(negate)]
[[34.0,29.0,-29.0,-21.0,-3.0,3.0],[34.0,29.0,-29.0,-21.0,21.0,3.0],[34.0,-34.0,-26.0,26.0,21.0,3.0]]
```

You can now ruin the fun of any level of *Calculator: The Game* for yourself.
