+++
title= "KISS Your Dependencies"
date= "2026-01-31"
tags= ["post"]
draft= false
+++

In every developer there are two wolves. The first is a pack animal, and it hungers to hunt alongside the sexy, magical, new libraries and frameworks. When the first wolf is awake, the developer is looking for any opportunity to squeeze in something like a stream processing library just to run a map function on an array, or adopt an entire web application framework just to write a few CRUD endpoints ([Martini](https://github.com/go-martini/martini), anyone?). But when something breaks, or a race condition is detected and the library's documentation falls short, the second wolf emerges from its cave ready to bonk the first wolf for making life so hard. 

The second wolf is the lone wolf. The one that growls suspiciously at import statements and multiple layers of leaky abstractions. It needs no one and can write whatever the project needs in pure assembly the way God intended. His canine motto is 

>	"Awooo rrrrrgh"

which roughly translates to 

> "Whatever time was gained by using a new dependency will be lost in learning it or debugging it or dealing with the fallout when it is deprecated."

Balancing these two wolves is one of the many skills a software developer must master in order to maintain velocity on a project over a long period of time. This skill is something that will be increasingly valuable as a distinguishing discernment between human SWEs and their AI agentic assistants. 

I picked up an MR where a coworker of mine introduced a new dependency, a Go implementation of ReactiveX, into the distributed platform we were building. If you don't know anything about ReactiveX it's more than an async library, it's a whole paradigm and vernacular attempting to revolutionize stateful multi-processing. Now this coworker is a fantastic engineer. In no way is this story meant to accuse or demean this guy. He is intelligent, motivated, personable, and regularly invests his expertise into developing the rest of the team. The "trust capital" is strong with this one, and so me and my co-reviewers dug in and didn't question why he didn't just use a Go channel. It took us a substantial amount of extra time to read through the docs on this library to understand what he was doing and why and how to test it.

{{< infobox title="Note" >}}
Lesson 1: Dependencies can increase the barrier of entry even for other engineers skilled in the implementation language.
{{< /infobox>}}

Months later, a critical bug was reported in our distributed system that was preventing certain deployment topologies. I donned my detective hat, spyglass, and bubble pipe and dove right in. Fairly quickly I was able to determine that the problem was related to the update function not getting triggered under certain situations. The agent responsible for triggering this update function? The very ReactiveX Observer pattern I had reviewed and approved months before. I spent hours relearning the library, combing through the documentation, and where the docs failed me I drilled into the library source. 

{{<infobox title="Note">}}
Lesson 2: The barrier to entry cost may be paid more than once.
{{</infobox>}}

Finally, I discovered there was a bug in the source of the dependency that was blocking subsequent updates due to an underlying Observer failing to drain a channel. Why we used the library in the first place was that we were preparing in case we ever needed to broadcast certain events to multiple listeners, and yet nobody on the team could think of a specific example of why we would need that. It just seemed like good "future proofing" at the time. After spending some time confirming the bug, I finished replacing the entire dependency with a single Go channel. 

{{<infobox title="Note">}}
Lesson 3: Use the primitives first. Then reach for dependencies if you truly TRULY need them.
{{</infobox>}}

And here my story ends. My second wolf a bit stronger and bit more trusted, I herald the tried and true KISS (Keep It Simple, Stupid) rule as a guiding light. For critical code, keep your implementations simple and boring and KISS your dependencies goodbye.

