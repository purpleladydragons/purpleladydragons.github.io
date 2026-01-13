
First impressions were that I could use reinforcement learning to build a bot to do it, but that wouldn't be very fun. 
I personally wanted to learn optimal play, and writing a RL bot wouldn't help that (think about how letting something like AlphaZero rip probably didn't make its creators that much better at chess)
So instead, I thought that since the game was relatively simple, that it would be feasible to solve with basic game theory tools

That ultimately ended up being wrong afaict. It's because of a couple reasons. I'll try to show my learning process as I experienced it.

I wanted to simplify things as much as possible. I ended up with some pretty trivial toy models, like 2-player 1-round, or multiplayer 1-round, but gems can be split into fractions instead of being left as remainders.
For these, it's easier to decide what to do. Ultimately, it's because if you have superrational players, then they shoud all decide the same thing at each turn.

Things immediately become more difficult when you start thinking about n-player with artifacts and normal gem remainder properties.
There is now an incentive to leave beyond the chance that you would die by staying. So I tried stripping the game down and modeling it as a congestion game.
This would be beneficial, since finite congestion games are potential games and therefore we could find the (exact?) Nash equilibrium by optimizing the potential function.

Unfortunately, I was naive in my modeling. I was hoping I could take the risk-neutral EV of the deck and use that, but it's not that simple.
The problem with counting gems as utility is that a player would be indifferent between one opponent having 20 gems and two opponents having 10 gems each. 
If the player has 15 gems, these clearly are different results. So we need a way to map the EV to some kind of fractional win/loss value.

This immediately started ringing the RL bell for me. I figured maybe I could use RL to build a valuation engine and then find a potential function using this to build a pretty good approximation.
But it felt somewhat convoluted (why go through the trouble of using RL just to provably construct a NE of not even the true utility?)

Upon further reading, I got the impression that my approach was somewhat doomed. Extensive games only assign values to their terminal nodes. 
My problem has been that every turn and round feels so similar that we ought to be able to treat the game on the per-round or perhaps even per-turn level
to learn behavior.

But a general extended game doesn't have this property. Decisions within the tree could be of totally different nature from preceding decisions.

So it's possibly that I'm failing to exploit this repetitive nature of the game, but based on poker literature, I think we are expected
to treat the whole 5-round game as the extended game. And improve through self-play comes not from playing several turns, or several rounds, but several full-games.

Of course, we can use abstractions/simplifications to make things tractable, but to truly optimize for Incan Gold, we need to play the actual game with its true payoffs many times.
