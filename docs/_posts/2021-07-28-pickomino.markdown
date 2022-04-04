---
layout: post
title:  "Solving pickomino using a Markov Decision Progress"
date:   2021-07-28 15:04:35 +0200
categories: jekyll update
---
Pickomino is a dice game where the goal is to get as many worms as possible.
Worms are obtained by throwing 8 special dice and trying to get the highest total possible.

![dice](http://www.analoggames.com/wp-content/uploads/2016/01/AnalogGames.com_analog_games_board_game_dice_regenwormen_pickomino_heckmeck_reiner_knizia_geek.jpg)

A turn goes as follows:

1. Throw the dice
2. Pick one number which is among the rolled dice, and which you have not picked before
3. Set aside all the dice with the chosen number
4. Check if you have at least one dice and one number left. If not, stop your turn
5. Go back to 1, and repeat until there are not dice or numbers left

At the end of the turn, count the sum of the dice which were set aside.
If a player has no worms at the end of their turn, their score is set to zero.

### Example turn
On their first roll, a player rolls 6-6-6-5-5-1-2-3.
They set aside the sixes.
Since the player has numbers and dice left, they throw again and roll 6-6-5-1-2.
Sixes were already chosen, so the player chooses 2 and rolls again.
The dice come up 5-5-5-6, so the player is forced to choose the fives.
The player rolls again, and the die comes up 5. Since fives were already taken, the turn is over.

The final score is 5+5+5+2+5+5+5 = 32.
The player has at least one worm, so their score stands.

### Basic observations
By playing the game for a short while, some basic facts become apparent:
- Worms are strictly better than fives: if you have the choice between N worms and N fives, always choose the worms.
- Sometimes it is beneficial to choose a low number early so you only 'lose' one die. Consider the roll 3-3-3-3-3-3-3-1. Here the one should always be chosen, since the chance of getting a worm with one die is very low.

After playing a little more, situations occur where you might not be sure which option is best. Some examples:
- Rolling three fives and two worms in your first turn. Choosing the fives will probably result in a higher score, but only if you manage to get at least one worm later in your turn. Does the probability of getting a zero weigh up to the extra score?
- Rolling three fours and two fives. The fours add more points to your total, but cost an extra die. On the other hand, choosing the fours means you can still roll fives later in your turn.

After playing a lot of games, you get a certain 'sense' of which choices are better.
We generally agreed that on the initial roll three fives are better than two worms.
We had no hard proof of this fact, only the anecdotal experience that choosing the three fives generally resulted in better outcomes than the two worms. Unfortunately there are so many possible game states that this 'sense' cannot be fully developed in every scenario. Thus, we turn to the computer for answers.

### Markov property
An important observation about pickomino is that the order in which the dice are chosen does not matter for the game.
Choosing 6-6 on the first roll and 5-5 on the second gives the exact same game state as choosing 5-5 first and 6-6 second. In mathematics this is called the *Markov property*.

### Dynamic programming
Dynamic programming is a technique to recursively break down a large problem into small subproblem.
As an example, consider sorting a list of N numbers.
This can be broken down into the following easy subproblems:
- Sort a list of 1 number (trivial)
- Given a sorted list of N numbers, insert an additional number such that the list remains sorted

Using these two 'ingredients', one can easily sort the list.
This algorithm is called insertion sort, and although it is known to be inefficient compared to other sorting algorithms, it is easy to implement and serves as a nice demonstration of dynamic programming.

To generate an optimal strategy for pickomino, we will compute a scoring function for each possible dice configuration.
It is easy to see how this result in a strategy: at any point where you have to choose which number to take, just take the one which result in the configuration with the highest score.
So what should this score function be? Ideally, we would like it to be the average of the score when starting with the given configuration.
There is a complication with this idea: the average score will depend which dice the player chooses in later turns, which depends on the strategy, which depends on the scoring function.
Therefore, we need the scoring function to compute the scoring function, which is obviously problematic.

However, there is a key insight which solves this problem instantly: during a players turn, the number of available dice can only go down.
This means that we can compute the scoring function backwards: first compute the score for each configuration where 0 or 1 dice are left.
This is easy, because there are no decisions to be made by the player.
If we have this list of scores, we can compute the scores where 2 dice are left as follows:

1. Create a list of all possible dice rolls with 2 dice, with their corresponding probability p_i.
2. For each possible dice roll, find out which player choices are possible. Since every possible player choice results in a configuration with 0 or 1 available dice, we can look up the scores and find the best option with the best score s_i
3. Compute the score of the current position by multiplying each p_i with the corresponding s_i and adding all these up.

Using this algorithm we can compute the scores of all configurations with 2 dice left. Now that we know all these scores, we can do the same thing for all configurations with 3 dice left, and so on until we have computed the score of the configuration (singular!) with 8 dice left.
