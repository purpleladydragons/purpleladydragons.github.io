## Volleyball serve probabilities

In high level volleyball, a team is expected to sideout more often than not. The advantage is with the receiving team. If you give a top team a
freeball, you should expect them to score at an absurdly high rate regardless of how strong their opponent is defensively. The attacking team generally
has the advantage (which makes sense if you think about it at all - unless the attacking team is so incompetent that they are more likely to hit the ball out
than in - you simply have to multiply your probability of siding out yourself by whatever less-than-1 probability you have of digging their attack).

In high level volleyball, you also see a higher rate of jump serves. This is partially due to the high sideout% of top teams.
For example, if you imagine a team with a 100% sideout rate given a freeball, then any difficult serve will be better than giving them a freeball
if you have any chance whatsoever of getting the serve inbounds. The takeaway here is that the better your opponent is at siding out 
(in a generalized sense that also captures the recursive scenario where you may dig their first attack, then they have to dig your first attack, then you may dig their second, etc)
the harder you should serve (hard in terms of difficulty, not ball speed per se) despite an increased risk of faulting.

Intuitively, it might seem that you should vary your serve's difficulty based on how much you are leading or losing, but if you make some 
simplifying assumptions, this becomes obviously untrue. 

The assumptions we need to make essentially boil down to the idea that the probability of winning does not stray from a fixed Bernoulli process
depending on the current score. E.g if the score is 20-19 (assume win by 1 for now), 
then the probability of winning is simply the probability of scoring one point before your opponent scores two points (p + (1-p)(p)). 
Reasons that this might not be true: psychological effects to being up/down a lot of points, 
changes in serve receive quality related to the serves experienced so far, fatgiue from serving and serve receiving.

If you work with this simplifying assumption for now, then you can see that you should simply maximize p. 
p essentially is the % that you ace + the % you prevent a sideout. You can abstract further and say that "% you ace" can be rolled into "% you prevent a sideout".
If you know a team sidesout 100% of easy balls, then you simply have to serve a hard ball. If they sideout 90% of freeballs, then your rationale becomes something like:

ace% + out-of-system% * out-of-system-sideout% + in-system% * 90% (+ 0 * fault%)

If this number is >10%, then you should serve hard. If <10%, then you should take your chances on the freeball sideout. 
This makes it clear that as you play worse teams who struggle with siding out, you should serve easier to avoid faulting. 
TODO: what about teams though that suck at receiving hard serves, but can sideout a freeball consistently? like low level players who can't pass, 
but their setting partner is good and they can hit fine. like basically the formal way of saying this is, i think there is some 
assumption of a universal relation between hard-receive quality and soft-receive quality, but this difference itself can vary
across teams

And whatever decision you make, you should do that every single point of the game (if you make the simplifying assumption).

## Figuring out your highest EV serve

Assume perfect knowledge for now. Every % estimate you make is perfectly accurate. If you think you serve in 70%, then you _do_ serve 70% in. 
To be clear, this is _not_ necessarily a reasonable assumption to make. You'd need a lot of reps to even have a solid estimate and that doesn't account 
for game conditions either (fatigue, wind). 

So say there are a discrete number of serves. You can estimate the difficulty of each serve. Regardless of your opponent, 
you can estimate your in-bounds and out-of-bounds %s (ignoring opponent-dependent things like pyschology etc). But for ace, in-system, and out-of-system,
these depend on your opponent's ability. 

- estimating the opp's ability ("knowing" them just means lots of samples/observations. you can guess based on population stats as well)
- start simple with 2d grid of ball position when it hits sand, to show that serves can transition to other serves

complicating factors
- do you _want_ them to be forced to sideout? fatigue for them (or worse for you)? advantage to defense for learning strategy?
- changes over time in serve quality (you get fatigued over time, they learn a certain serve)
- probability estimates / distros instead of single scalar % values
- generalizing small sets to countably infinite set to continuous set (4 types of serve, vs infinitely many serves in court (discretely feels intuitive, but then that gives you continuous very quickly)
- 
