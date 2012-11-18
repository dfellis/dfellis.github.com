---
layout: site
title: Opinions and Engineering
subtitle: 
---
I'd like to start out this blog -- my first ever -- on a topic that affects everyone ever involved in a real engineering project, and yet doesn't seem to be addressed much.

Engineers are people, and people have opinions. What do you do when two engineers have conflicting opinions? [This article](http://www.cs.cmu.edu/%7Ebmclaren/ethics/cases/foundational/63-6.html) concludes that there is no *ethical* problem with conflicting opinions on an engineering problem. But how to reconcile the differences in opinion? If you need to make an engineering or business decision, this *must* occur eventually, or the project will go over time and over budget. But having no systematic way of determining how to reconcile the opinions removes the engineering from the solution into something superstitious: "We're taking this path because I trust this guy, and I trust this guy because he has my trust."

First we need to figure out *why* there are differences of opinion. (Essentially, we're answering the old journalist's question list of "Who? What? Where? When? Why? How?" only "who, what, where, and when" are usually very easy and implicit to this discussion.)

The first source to check is a difference in experience. This is not seniority; experience is a multi-dimensional thing that grows in its total volume (area? size? what's the right word for an n-dimensional space?) over time, but not uniformly. A difference in opinion may be coming from the differing engineering backgrounds of the two (or more) engineers. Having the engineers discuss and debate the merits of the different proposals is an obvious solution, but a [recent proposal](http://www.nytimes.com/2011/06/15/arts/people-argue-just-to-win-scholars-assert.html?_r=2) that we're wired to try to *win debates* rather than find the objective truth would explain why this can often devolve into egotistical pissing matches.

Perhaps putting this solution on its head can help: have the engineers argue for *each other's* opinions. If the engineers reason out the valid points of the other's suggestion, it may not seem so anathema afterwards (they may come to feel as though they "own" some of the points as well -- especially if they find a point in favor that the originator did not). Then actual reconciliation of the opinions could occur, throwing out the ego-game.

But the difference of opinion produced by a difference in experience may not produce a situation where one engineering opinion is better than another. If the [problem space](http://ai.eecs.umich.edu/cogarch0/common/theory/prob.html) of the particular problem has multiple peaks, two engineers starting at different points in the problem space (starting out with different focuses of experience) may come to wildly different, but equally valid solutions.

In this situation, any reconciliation could produce a result that is *worse* than if either opinion was followed unaltered.

There is also the possibility that one of the opinions is actually *wrong* because of a lack of relevant experience; a reconciliation of opinions would only dilute the more correct opinion, in that case. It would be hoped that the procedure above would cause the less-experienced engineer to about-face on his original proposal, but he is still human and may not do so.

Peer-review can help here, but also runs the risk of debate manipulation, e.g., *My solution is similar to previous solutions we've used and is familiar to you, his is different and unfamiliar!*

Ideally, a group of peers with drastically different backgrounds and no investment in either solution would be available to judge the opinions. Scientific journals attempt to produce this sort of environment, but are of course limited by the finite number of researchers, the researchers' finite time to read published papers, and the built-in egotistical drive to be the one who produces the game-changing solution. These published, public reviews also take quite some time to be resolved, often measured in years, and therefore are not appropriate for all issues.

There is also the limitation that such a review is public to those allowed to review, so any problem treated as proprietary by those attempting to solve it have a drastically reduced audience for review.

One possible solution to the deadlock produced by the last two scenarios where reconciliation is undesirable is to analyze the opinions through [cost-benefit analysis](http://en.wikipedia.org/wiki/Cost-benefit_analysis), specifically, what is the cost of attempting each proposed solution, and what is the expected benefit. Cost, in this case, can refer to money, time, and resources (meaning a particular resource that is finite and cannot be bought or possibly even remade, such as a prototype). How they are weighed is another engineering problem, itself, but any large engineering group most likely already has these at least roughly weighed out.

In this scenario, the "cheapest" solution would be tried first. For the situation where both opinions are "right" but different, the problem is immediately solved and the group can move forward along the cheaper path (which could therefore be seen as the better path, in some sense). In the scenario where one is right and one is wrong, if the one who was right was more expensive, the group has thus disproved the original proposal by the failure of the attempt, and can move forward on the correct, more expensive path. If the one who was right was the cheaper, then the group has not wasted an inordinate level of resources on the problem.

That is, assume that both projects total to a cost of 1. The cheaper project costs 0.3, and the more expensive costs 0.7. If this strategy is applied consistently on good/bad engineering opinions, and assuming that the distribution of good and bad opinions between cheaper projects and more expensive projects is 50/50, always attempting the cheaper solution would cost on average 0.65 versus 0.85 for always choosing the more expensive option. Some overhead versus the ideal 0.5, but better than the alternative of always choosing the more expensive option and most likely better than trying to be "lucky".

An example: In [this research paper](http://dx.doi.org/10.1109/TED.2010.2053864) I wrote, I am quite proud of the fact that all of the equations were written prior to any measurements being performed, with the resulting measurement data exactly matching my predicted results. However, I was challenged (by one of the co-authors) that I made too many assumptions in the derivation (I did make assumptions, and spelled them out explicitly, along with a brief explanation as to why I believed it was justified), that some of the assumptions should be removed, and that this be tested first.

I was able to argue that this would require more time for me to re-derive the equations, and would require more measurements, in which the very measurements needed for the equations I proposed would be just a subset for the more generic equations. I also stated that I could complete all of the needed measurements in a week, so I was given permission to use the material and perform the measurements, and sure enough, I was justified in the assumptions I made.

In this case, the total cost of both proposed measurements is 1, while my measurements were 0.3 and his were 1 (as mine were a complete subset of his -- my proposed solution would produce measurements that did not match the predicted behavior if my assumptions were not correct).

Opinions in Engineering happen, and they can cause an otherwise ordered group to turn raucous. One or both parties in the argument over engineering opinions may be right. The solution to dealing with opinions in engineering can itself be treated as an engineering process (yes, that can recurse infinitely), first attempting to manipulate human nature to nullify its impact as much as possible, and then attempting to mitigate the cost involved with choosing poorly. This process has been demonstrated as an empirically better process (the 0.65 average cost versus the 0.85 average cost), and anecdotally, too! ;)

David