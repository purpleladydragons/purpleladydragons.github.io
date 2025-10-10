I live by a cliffed beach, but due to erosion, it's only rocks most of the time. At low tide though, you get a wide stretch of flat sand perfect for walking.

The caveat is that not all low tides are low enough, in fact most tides are too high. The cobblestone part of the beach is steep and permeable. 
This means the tide can fall several feet from high tide without much noticeable visible change. 
You would still only see cobblestone and it would seem like the water is coming just as close to the cliffs as before because the vertical change
in the beach is much greater than the horizontal change.
However, once the tide reaches low enough to start revealing the elbow-point where the cobblestone meets the sandy bottom, the sand flattens out greatly
which means a relatively small change in tide level results in a drastically larger change in beach width. 
The effect is quite impressive as you can go from almost no beach at all to a beautiful wide salt-flat looking beach in a matter of minutes.

My girlfriend really likes walking on the beach, but it's not intuitive as to when the tide is low enough to actually walk.
Again, low tide doesn't guarantee walkability. Where I grew up, tides basically resembled a sine curve that shifted ~1 hour every day.
Where I live now, there can sometimes be no tide at all or essentially only one high and low tide per day, 
with the other pair of high/low being much closer to midtide.
This variation, in combination with the essentially binary walkability status of the beach, makes it unintuitive to know when the beach is walkable.

So I watched the tide one day to see when it went from unwalkable to walkable. I checked the tide chart and inferred the tide height by assuming
a linear slope between high and low tide that day.

Using that walkable-tide level, I can now write a script to read the upcoming tide predictions from NOAA, find the times when the tide will be lower
than the walkable threshold, and send myself a notification of those times.

I use ntfy to send the push notifications to myself and my girlfriend. Ntfy is great because you don't need to deal with any auth stuff. 
You can just simply POST to your topic and get the notifications on your phone if you the have the app installed. 

Repo: https://github.com/purpleladydragons/tidewalker

![IMG_0D2255D61C18-1](https://github.com/user-attachments/assets/eefd39c9-83a0-47b6-9bbd-9375381a531f)

