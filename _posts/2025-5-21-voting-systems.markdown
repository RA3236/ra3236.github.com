---
layout: post
title: On preferential voting
---

This is a work-in-progress post that I've made up to help visualize some voting systems (including Australia's lower house system) and to explain the flaws within those systems. Please let me know if there are issues, or if I can expand further on stuff, via the Issues tab on the Github repo below (or on Reddit if I post it there).

This post contains two scenarios: a single seat election, and a full House election.

You can see the code for the simulations here: [https://github.com/RA3236/voting_systems_tests](https://github.com/RA3236/voting_systems_tests). It's actually an application you can run, but you'll need a Rust compiler to build it because frankly I'm not skilled in the deployment department.

## The Single-Winner problem

The subject of this post is primarily about single-winner systems - the one used in the Australian House of Representatives. When talking about voting systems, there are two primary classes that exist:

1. **Single-winner systems**, which elect one candidate into one seat (using the same ballot paper for all voters), and nothing else; and
2. **Multi-winner systems**, which elect multiple candidates into multiple seats at once (using the same ballot paper).

Australians should feel a bit familiar with both of these. The federal House of Representatives uses instant-runoff voting, a single-winner system, to elect its 151 members. The federal Senate uses the single-transferable vote (a multi-winner proportional system) in a bit of an abomination involving the number of states and not population, but that's a discussion for another time.

You might be surprised to learn that there are forms of multi-winner systems that are deliberately *not* proportional, and are actually used. Bolivia uses a mixed-member majoritarian system instead, which involves mixing winner-take-all and proportional results (leaning towards the former).

Single-winner systems are referred to as winner-takes-all systems, for obvious reasons - only one winner can exist. Out of these systems, there are again two main classes: ranked-choice methods, which involve ranking candidates (like the Australian House); and score voting systems, which involve scoring or marking candidates. The latter class includes First Past the Post and STAR voting.

We could go into proportional multi-winner systems in this post, but they are a little harder to visualize without going into party seat counts and such. Single-winner systems are a bit easier to visualize.

I want to explore four different single-winner methods in this post, such that you can get an idea for why different methods exist and what their problems are. Three of them are "serious" methods, while one of them is a little more fun.

>The Senate works by giving each State twelve Senators and both the NT and ACT two each. The States are divided into two electoral districts, elected every six years, such that every federal election elects one of the districts.
>
>The districts are elected using the single transferable vote, which works similarly to the House's system except in a multi-winner fashion. Due to sheer luck, we have managed to dodge America's rural-urban state divide, and as such the total seat counts in the Senate are indeed relatively proportional.
>
>The Senate being proportional does aleviate some problems with the House, but not all of them.

## How am I simulating them?

Simulating politics is hard. Very arguably impossible. So obviously there needs to be some abstractions to help make the system make a little more sense, and to make it easier to create "accurate" simulations.

A lot of you should be familiar with the political compass. It looks like this:

![Political Compass](https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Political_Compass_standard_model.svg/800px-Political_Compass_standard_model.svg.png)

Wait, only two axes? Aren't there hundreds of issues in every election?

Yes, there are. However, social choice theorists have worked out that in many elections, a 2D model works pretty well to describe the individuals and parties and their positions. As such, that basic system is what I'll be using. I'll represent preferences by the distances between a voter and a candidate on the 2D plane - the Euclidean distance (actually the squared-Euclidean distance, but that doesn't matter).

The question is then how we will generate the voters and candidates. In this case, I'm going to primarily use the Beta distribution to generate the (x, y) coordinates of every person. This is a random probability distribution that shows increased odds of a point appearing closer to the centre than towards the edges. Using some basic parameters, we have our voters and candidates:

| ![Voters (Beta)](/assets/voting_systems_1/beta/Voters.svg) | ![Candidates (Beta)](/assets/voting_systems_1/beta/Candidates.svg) |

The Voters graph has a lot less points in it than the Candidates graph due to how I've optimized the simulations, but you should still see how this works. The more red a point is, the denser the populace is around that political position.

Another distribution I'll use, primarily to exaggerate certain effects, is the uniform distribution:

| ![Voters (Uniform)](/assets/voting_systems_1/uniform/Voters.svg) | ![Candidates (Uniform)](/assets/voting_systems_1/uniform/Candidates.svg) |

This looks a lot less natural than the Beta distribution, but this allows some problems with certain systems to become really apparent visually speaking.

You might ask why I've dumped the populace in a line, and the answer is two-fold:

1. It makes it easier to understand certain things, like splitting the vote

2. A lot of elections are kinda on a line like this. Kinda - this is a simplification.

I should mention at this point that these are approximations, and especially later these will become less and less accurate to the Australian reality. However, I don't think this is necessarily bad for understanding. Even an inaccurate voting populace still allows one to understand where the methods went wrong.

## First Past the Post

For those out of the loop, First Past the Post (also known as plurality voting) is the system used in both the UK and the US. Unlike our ranked-choice ballots, a voter can only tick one candidate's box, and no others. The winner is whoever gets the most votes.

Many of you already know why we don't use FPTP, but to summarise: you can't specify your preferences. If you vote for a losing candidate, your vote straight up doesn't matter.

This has a few effects just beyond not preferencing, though. FPTP is famously susceptible to the spoiler effect, where a losing candidate can affect the outcome of an election simply because they ran. Think of it - if two candidates from one major party run at the same time, then the other party is almost guaranteed to win.

The spoiler effect is a bit hard to visualize, but we can simulate a hundred thousand elections with FPTP and the voters and candidates from the earlier Beta distribution to get an idea of how the winner distribution looks like:

![First Past the Post (Beta)](/assets/voting_systems_1/beta/First%20Past%20the%20Post.svg)

>This is a single-seat election, in case you are confused. There are six candidates, and 5000 voters. Because of statistics, 5000 voters is a reasonable number since most differences would have ironed out at a much lower number.
>
>Also my CPU doesn't like 100,000 voters.

As you can see, FPTP elects a very wide range of candidates. They can be either extreme or centre, but more likely extreme. Let's have a look at the uniform distribution:

![First Past the Post (Candidates)](/assets/voting_systems_1/uniform/First%20Past%20the%20Post.svg)

Wait, what's going on here? Why are you more likely to win if you are further away from the centre?

The answer is complicated, but this is a form of spoiler effect called the "centre-squeeze" effect. The centre candidates get a good portion of the votes, sure, but the extremes can get a higher portion because they aren't competing on both sides. As such the centre is "squeezed" out of the election.

Many of you are likely familiar with the idea that a voting system shapes the politics of a country. In America, while the right-Democrats are largely "centre" (in their Overton Window), the Republicans have gone off the rails onto the far right. This is partly because of FPTP (and the Senate, which is an entirely different discussion).

## But wait, what's a "good" election outcome?

This is a bit hard to quantify, and this heavily depends on the person you are talking to.

I personally think that good single-winner systems should tend to the candidate who is the closest to the *average* voter. This way, you get a relatively steady government and prevents the government from having to win over extremists and idiots on both sides of the spectrum that will only lead to the government hurting people.

Extremists and idiots both come from the same place: they've been wronged. Unfortunately, single-winner voting systems do not address the problems that those people face - instead, they encourage corruption and the formation of two major parties (see Duverger's Law). I'm a big advocate of the government stepping up and solving the fundemental issues plaguing our society, but a change in voting system won't do that. So a voting system should aim to keep the worst of those ideas out (not that any are actually successful at doing that, but I digress).

There is another condition that I prefer, but I'll get into that shortly.

>Duverger's law states that FPTP will always lead to a two party system. I would actually like to go further than that and say that *all* single-winner systems will become two-party systems.
>
>Australia is an example of such a two-party system. Every time one of the parties has collapsed or split, another takes it's place (or a coalition does, who agree not to run in each other's seats). And yet we don't use FPTP.

## Instant Runoff

This is the voting method the Australian House of Representatives uses to elect their 151 members. The idea behind the method is that you can now rank candidates according to your preferences, and as such you can direct how your vote flows to others.

This is achieved through the following process:

1. All of the first-preference votes are tallied up and counted.
2. Find the candidate who got the least first-preference votes. Mark them as excluded.
3. Transfer all of the candidate's ballots towards the other candidates, by pretending the excluded candidate doesn't exist.

On an intuitive level, this sounds like a pretty good system. This ensures that your vote will not be wasted, and that you can vote for your actual preferred candidate then preference whoever you like.

So let's have a look at the results for the Beta distributed voters and candidates:

![Instant Runoff (Beta)](/assets/voting_systems_1/beta/Instant%20Runoff.svg)

As you can see, the "winning area" is now a bit smaller than FPTP. However, you'll also see that the centre-squeeze effect is actually noticable on this chart. It still seems like candidates are more unlikely to win if they are in the centre.

If you actually look at the numbers, there are about 50% more winning candidates who are in the centre compared to FPTP. An improvement, but not that much.

Let's have a look at the uniformly distributed populace:

![Instant Runoff (Candidates)](/assets/voting_systems_1/uniform/Instant%20Runoff.svg)

Here we can see that the winning area is definitely small, but the centre-squeeze effect is still in full swing.

So what's going on here? Is it impossible to elect an "average" candidate in single-winner systems?

It turns out that it isn't (at least the majority of the time). Instant Runoff voting violates a major voting method criterion: the Condorcet winner criterion. The Condorcet winner criterion states that, if in every matchup a candidate beats every other candidate, then that candidate must win (essentially, the majority-preferred candidate must win).

Think for a second about why First Past the Post also violates this criterion.

IRV doesn't just violate the Condorcet criterion - it fails it hard. As you'll see, it fails to elect either the Condorcet winner or the closest-to-average winner over 80% of the time. So why does it fail this criterion?

I've seen a lot of people state that IRV is a preferential voting system. It is more accurately an ordered voting system. It doesn't take into account the preferences of the voting populace as a whole, and as such it misses key information that prevents it from electing the majority preferred candidate.

>Someone is going to mention it, so I have to address it. The difference between this simulation and real life is that I'm not accounting for tactical and strategic voting, as well as the long-term movements of parties and candidates. This is why real life numbers show only ~10% or so violations of the Condorcet winner - it's because IRV has squeezed out the "true" Condorcet winner, preventing them from ever winning, so they give up and stop running. This in effect makes someone else the Condorcet winner, but not even close to the "average" voter.

A recurring theme in politics is choosing the voting system to strengthen one's own position at the expense of their competitors. IRV was introduced in Australia in 1918 because the predecessors to the modern Coalition were splitting the vote amongst themselves, and as such they couldn't win elections as easily.

## What happened in the 2025 election?

One of the greatest weaknesses of both FPTP and IRV is dealing with situations where there are many candidates with relatively close amounts of votes. Ironic, given the reasons behind IRV's introduction.

In the 2025 election there were multiple seats where multiple parties gained a significant portion of the vote, which led to interesing outcomes. I'm not going to go over all of them, but plenty of you will be familiar with the situation in the (now former) Greens electorates. Let's have a look at Brisbane:

![Brisbane 2025](/assets/voting_systems_1/brisbane_2025.png)

(Sourced from the ABC website)

The LNP, Labor, and Greens all got over 25% of the first-preference vote, with the remaining parties getting the remaining 10% or so.

The AEC doesn't publish the ballots for the House of Representatives (apparently it's not in the Electoral Act 1918), so I can't perform simulations to determine who is the Condorcet winner etc. However I think it's pretty clear that Labor in this case would have been a pretty big contender - and they did win the seat.

Let's look at the results for 2022 then:

| (this is an image, not a table) |
|![Brisbane 2022](/assets/voting_systems_1/brisbane_2022.png)|

(Sourced from Wikipedia)

The Greens managed to pick up the seat, despite not being anywhere close to the centre. Let's have a look at the preference flows (because Wikipedia also has a nice graph for it):

![Brisbane 2022 Preference Flows](https://upload.wikimedia.org/wikipedia/commons/thumb/2/22/2022_Australian_federal_election_Brisbane_alluvial_diagram.svg/1920px-2022_Australian_federal_election_Brisbane_alluvial_diagram.svg.png)

And right here we see the centre-squeeze effect. After the smaller parties are eliminated, we are left with the LNP, Greens and Labor, in order of their current vote share. You would think that Labor would win the election here, because they are in the centre and thus should receive the preference flows. However, they are also in last place currently with first-preference votes. Because of this, IRV eliminates Labor, and then elects the Greens.

There are very good arguments for why the Greens actually should have won this seat (given their seat share in the House compared to their votes), but at the seat-level this is a pretty clear failure of IRV.

## Ranked Pairs

Because of the failings of voting systems to elect the Condorcet winner, the ranked pairs system was proposed by Nicolaus Tideman in 1987. It is a Condorcet system, which means it always elects the Condorcet winner.

Here's how it works (simplified):

1. Find every pair of candidates `A -> B` such that more voters prefer `A` over `B`.
2. Sort those pairs by the margin of votes (or the number of votes for the first candidate, depending on what you want).
3. Go through the list of pairs. Draw arrows between candidates according to the pairs and who is preferred over who. If you add a pair to your diagram and end up in a situation where you get a cycle (like `A -> B -> C -> A`, where A is preferred over B, B over C, and C over A), then delete the pair.
4. The winner is the candidate who has no other candidates who are preferred to them.

Take a minute to think about how and when the Condorcet winner is found and elected. The Condorcet winner may not always exist - you may have a Condorcet cycle like that in the 3rd step involving all of the candidates. So ranked pairs finds the candidate that is the most Condorcet-like in that situation.

There is a good example on Wikipedia that explains the method with an election in Tennessee, comparing the results to FPTP and IRV: [https://en.wikipedia.org/wiki/Ranked_pairs#Example](https://en.wikipedia.org/wiki/Ranked_pairs#Example)

Now let's have a look at the results. We will do both distributions at the same time:

| ![Ranked Pairs (Beta)](/assets/voting_systems_1/beta/Ranked%20Pairs.svg) | ![Ranked Pairs (Uniform)](/assets/voting_systems_1/uniform/Ranked%20Pairs.svg) |

In terms of electing the closest-to-average candidate, this is a substantial improvement. While the uniformly distributed populace shows some variation in winners, it is still significantly more compact than both FPTP and IRV.

>Again, I haven't taken into account tactical/strategic voting. I don't know how much those affect ranked pairs.

Some of you may be wondering whether ranked pairs is still susceptible to the spoiler effect: it is. In fact all ranked-choice systems are susceptible thanks to a proof known as Arrow's theorem. You can look that up if you like. Ranked pairs is a lot less susceptible to spoilers, but it can still rarely happen.

## Anti-Plurality Voting

This is the fun method I mentioned early. Where the goal of the other methods was generally to elect a representative candidate (at least presumably), this one says the bigger issue is keeping the extremists out.

Essentially, the vote is identical to FPTP, except you vote for the candidate that you hate the most. This can be some random woman who is alright but everyone else better, or it can be Hitler. The point is you vote for the worst candidate.

Then the votes are tallied. Whoever gets the least votes wins!

This can lead to some pretty funny results later one, but let's have a look at the winner distributions first:

| ![Anti-Plurality (Beta)](/assets/voting_systems_1/beta/Anti-Plurality.svg) | ![Anti-Plurality (Uniform)](/assets/voting_systems_1/uniform/Anti-Plurality.svg) |

Now yes, this looks bad, but consider that the most extreme candidate is always prevented from winning. That's pretty neat, isn't it?

Yes, ranked pairs does this but substantially better. But it's less fun.

## A quick comparison

Here, I'm quickly going to throw out two charts. The first is the average distance between the winning candidate, and the Condorcet winner or the closest-to-average winner (these can be different!):

![Distances (Beta)](/assets/voting_systems_1/beta/distances.svg)

The exact numbers are probably a bit confusing to most, but the maximum values here correspond to about a quarter of the way out from the "centre". As you can see FPTP is pretty terrible, and IRV is better. Ranked pairs almost never deviates from the average voter. Anti-Plurality voting looks bad again, but you'll see it's absurdity soon enough.

>In my testing Anti-Plurality did significantly better up until I fixed a bug with my distribution calculations. So under certain parameters the method does actually perform decently.

Now here are the types of winning candidates:

![Winning Candidates (Beta)](/assets/voting_systems_1/beta/winner%20types.svg)

The way the voters and candidates are distributed means that the Condorcet winner exists 100% of the time. As per theory, ranked pairs always elects this Condorcet winner. However, a very small amount of iterations saw the Condorcet winner being different to the candidate who was the closest to the average voter.

The other methods fail to elect the Condorcet winner or the closest-to-average winner about 50% of the time or more. IRV is the best, but pales in comparison to ranked pairs.

>In real life, its essentially impossible to find the "average voter". I have the luxury of being able to calculate that, but even in real life you can get a general idea of what the average person wants by looking at polls of different issues.
>
>The exact numbers here are different from every run, and substantially different depending on whether I've left a bug in the code. I probably haven't caught them all.

## So we should switch over to ranked pairs, then?

The answer is "yes, but it's not good enough".

Remember how I mentioned that extremists and idiots should both be excluded from the voting system as much as possible? This is to protect democracy (obviously), but this only applies to single-winner systems.

In multi-winner systems, you can indeed represent extremists and idiots in Parliament without compromising democracy. In fact you really want to. The idea is that you catch their issues early so that you don't end up electing Donald Trump later. By addressing problems early - and I mean the real issues - you avoid having those people radicalise and stop believing what the government says.

Think about this would work in America, for example, in 2010. The Democrats and Republicans would each be split up into at least four parties total - let's say, the Social Democrats (big government and free healthcare), the Classical Liberals (small government, pro-liberty), the Neoliberals (pro-business), and the Conservatives (social conservatives/reactionaries). The neoliberal bloc (the corrupt politicians for a bad simplification) would be substantially weaker under a multi-winner proportional system, as they would be forced to compete with the more popular Social Democrats, Liberals and Conservatives.

It's possible that America might not have elected Trump if they were in a proportional system, as the issues that led to the 2016 election - namely, being fed up with corruption and poverty - may have been addressed sooner thanks to those parties being forced to compromise more.

>Remember what I stated about the voting method choosing the allowable parties - our politics are heavily affected by the voting method. If a party consistently fails to form government, either in a coalition or on their own - they are more incentivised to give up. Parties can also move their policy positions to better take advantage of the system.

At this point, it's helpful to explore an entire election of 151 seats for the Australian House of Representatives.

The voters and candidates (now called parties, as it's a bit more difficult to represent independents given the varied amount of candidates) are distributed roughly the same way. My algorithm for generating them essentially spreads out the voters per seat such that the average voter in each seat is different from every one.

>My algorithm is absolutely flawed and I encourage you to suggest better ones. I essentially linearly space points in some range, and use those as midpoints of line segments to scale the generated voters into (only on the x-axis). This results in a slightly higher density in extreme seats, but I believe this is still good enough to show the problems here.

I am using party list proportional representation as a baseline. With this, all voters vote for one party each, and the percentage of votes corresponds roughly to the percentage of seats. Such a system encourages that collaboration and compromise I mentioned earlier.

The code I wrote will check for adjacent parties (in the sense of left-right) and whether they can form a coalition, compared to the outcome of PLPR. If those are different, it will produce a couple of graphs showing this difference. I plan on improving the application to show all outcomes, but that won't be done until after I release this post.

Let's start by looking at the amount of times my algorithm noticed a different governement coalition compared to PLPR while computing the simulations:

![Differences](/assets/voting_systems_1/seat_projection_diff.svg)

As expected, there are substantial differences between each method and proportional. This is because some minor parties might only get 10% of the vote in every electorate, and thus win zero seats, despite being relatively popular.

Let's have a look at some examples where IRV differed from proportional:

![Seats](/assets/voting_systems_1/seat%20projections%20(125).svg)
![Positions](/assets/voting_systems_1/positions%20(125).svg)

>Yes, I know the legend colours are different. I can't fix that with the voters distribution overlay. Sorry.

Here, the parties are spread very far around the voters caucus, and thus there is no true centre party. Partially because of this, the three main methods all elect party 3, who is the closest to the centre - yet party 3 only got about a third of the vote. In the case of IRV, this is because the other parties have smaller core voter bases (where the voter is closer to the candidate than the other candidates).

>I won't comment on Anti-Plurality voting. I'll let you think of funny explanations for it.

![Seats](/assets/voting_systems_1/seat%20projections%20(187).svg)
![Positions](/assets/voting_systems_1/positions%20(187).svg)

This one is extremely concerning. The more centre party (party 4) is squeezed out in IRV, so the more radical party 3 is given power in IRV. Meanwhile, ranked pairs provides too much power to party 4 compared to PLPR, and FPTP elects party 5 despite quite obviously not being preferred.

![Seats](/assets/voting_systems_1/seat%20projections%20(312).svg)
![Positions](/assets/voting_systems_1/positions%20(312).svg)

In this case ranked pairs gives government to party 3, which is the centre party. But IRV and FPTP both elect party 5, who is more right-wing.

Let's bump up the seat spread, to see what happens.

![Seats](/assets/voting_systems_1/seat%20projections%20(140).svg)
![Positions](/assets/voting_systems_1/positions%20(140).svg)

Yet again, all three methods leave party 4 out of government.

>It's desirable to allow even close candidates a chance to govern - those minor differences might be because of corruption!

![Seats](/assets/voting_systems_1/seat%20projections%20(156).svg)
![Positions](/assets/voting_systems_1/positions%20(156).svg)

FPTP shows obvious signs of a two-party system.

![Seats](/assets/voting_systems_1/seat%20projections%20(187)%20copy.svg)
![Positions](/assets/voting_systems_1/positions%20(187)%20copy.svg)

Here we see an extreme candidate and a bunch of centre-to-centre-right candidates. This might be pretty accurate to our current party system; from left to right, Greens, Labor, Teals (parties 2 and 3), Liberals, and Nationals/One Nation etc. You can see that the Greens get squeezed out of most seats and thus governance.

![Seats](/assets/voting_systems_1/seat%20projections%20(125)%20copy.svg)
![Positions](/assets/voting_systems_1/positions%20(125)%20copy.svg)

Here we see that IRV provides the left-wing coalition to form government (without the far-right party) with many more seats than they would with proportional.

## What are our conclusions?

One of the goals of the House of Representatives (and the Senate) is to provide localized representation. The first part of this article shows that our current IRV system fails to properly do that, as it can actively prevent the Condorcet winner from winning.

One thing I have forgotten to mention is the concept of swing electorates. In Australia, ten or so electorates per election are usually considered "swing" electorates, which means they are tossups in the polling. This means that they receive special attention from parties - most people tend to vote the same way every time, so the parties use that to spend more resources in the swing electorates and even change policies because of it.

As these simulations have hopefully indicated to you now, these swing electorates aren't even likely to be in the political centre. Worse, "swing" voters in non-swing electorates end up being effectively silenced because their votes cannot outweight the large portion of constant voters.

These are consistent failings of single-winner methods, and it's highly likely our politics have been substantially affected by them. IRV, in Australia, benefited the Coalition up until the 1990s. Soon, as the voting caucus moves further and further left, the Greens will become the main left-wing party, and Labor has a possibility of being squeezed out of the centre - under any single-winner method.

This is why I think that our system needs to be reformed. Our goal is to make every voice heard, and yet our voting system can restrict it. 