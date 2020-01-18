# Why Management Structures Dictate Software Architecture

![structure](../images/MANAGE_STRUCT.png)

[Conway's Law](https://en.wikipedia.org/wiki/Conway%27s_law) a perennial favorite for being quoted around the office defines how software architecture tends towards the communication (and by extension, the management structure) of the organization

> [...] organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations.
>
> — M. Conway

Why is it that this law from the 1960 still holds true today? I think I've gotten some where in my thinking about why that is. Unfortunately, it's not all that innovative or surprising but using [Occam's Razor](https://en.wikipedia.org/wiki/Occam%27s_razor) seems to be the least amount of amount of over head to setup.

Put simply, the reason why Conway's Law holds true comes down to consolidation of power by individuals within a hierarchy.

Let's back up a second and add some context. Generally speaking, there are two career tracks for the software engineer: `individual contributor` and `engineering manager`. The first is what it says on the tin but the second is a little more nuanced. Technically speaking, an engineering manager should be parallel in the hierarchy from the individual contributor, however, often this dynamic is changed or exploited. Either by an over-eager engineering manager or individual contributors looking to abdicate responsibility. Either way, a small number of individuals end up making the decision for the organization. By virtue of that fact, software architectures grow in the directions they're allotted.

Let's take an example from my past experience:

I worked at a large organization many years ago that was tasked with creating a content management system. The organization of the two teams was two heads of the project (one individual contributor, one engineering manager), spread out across two floors 2nd and 24th, and one siloed to the API while the other was to create consumers of the API to do various tasks. That second team (the team I was on) was broken up into many sub-teams building various features. Now, based on the pretense, how do you think the project laid itself out? If you guessed that the API ended up doing things nobody really needed, and had features nobody really used whilst, on the other side of the proverbial fence, the API consumers ended up building applications that used the API in nonsensical ways, and often had to rig a bunch of scaffolding and additional small extraneous applications to get their desired feature set.

But how did we get there? One step at a time, really.

At the top of our management hierarchy, we had two leaders who weren't cooperating. They also weren't not cooperating but it was enough to have a trickle down effect to us boots on the ground. We worked independent of our cohorts on the API team and ended up with a set of applications that weren't really suited for the API; likewise, we ended up with an API that served something other than what we were doing.

When I stood back, a year after the project had been in maintenance, I could have easily drawn you a map of the command structure just based on how the application worked.

I was almost depressed when acknowledging that fact lo those many years ago. And the fact that I still see teams making this same mistake today. It's like the realization you have when you realize "Peopleware" was written in 1987 but everything it outlines still exists today, some 33 years later. /J

[Home](../index.md)
