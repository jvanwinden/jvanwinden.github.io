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
- Rolling three fives and two worms in your first turn. Choosing the fives will probably result in a higher score, but only if you manage to roll a worm later in your turn. Does the probability of getting a zero weigh up to the extra score?
- Rolling three fours and two fives. The fours add more points to your total, but cost an extra die. On the other hand, choosing the fours means you can still roll fives later in your turn.

After playing a lot of games, you get a certain 'sense' of which choices are better.
My family generally agreed that on the initial roll three fives are better than two worms.
We had no hard proof of this fact, only the experience that choosing the three fives generally resulted in better outcomes than the two worms. Unfortunately there are so many possible game states that this 'sense' cannot be fully developed in every scenario. Thus, we turn to the computer for answers.

### Markov property
An important observation about pickomino is that the order in which the dice are chosen does not matter for the game.
Choosing 6-6 on the first roll and 5-5 on the second gives the exact same game state as choosing 5-5 first and 6-6 second. In mathematics this is called the *Markov property*.
