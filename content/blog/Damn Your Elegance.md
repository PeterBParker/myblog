+++
title= "Damn Your Elegance"
date= "2026-02-07"
tags= ["post"]
draft= false
+++
## My compilers professor sipped his giant mug of Mountain Dew
“Damn your elegance. I don’t want elegant code, and neither should you. Give me a clear solution that works.”

With those words my compilers professor shattered my iridescent idol of beautiful code with a hard truth of writing robust software. Like a spurned artist, I scurried out of that office hour with my laptop clutched to my chest and denial rooted in my heart. Didn't that gruff, Dr.Pepper-guzzling, old veteran know that with enough effort and intelligence a programmer can crystallize logic itself in all its optimized, non-redundant glory?

Perhaps I was bit dramatic then. Did I mention I did theatre in high-school?

Yet, years later, I think I've come to understand him. What young software engineers (like myself and the annual summer interns that sift through my team) can value minimal function signatures and reused pointers as signs of elegant code, in many cases may be unnecessary optimizations that reduce the lifetime of your logic. One such aspect of this is dual-purpose variables.
## Dual-Purpose Variables?
A dual-purpose variable is when a variable is named for one purpose and is reused for a secondary check that isn't directly connected to its initial purpose. This practice adds complexity and hides intent. Here's an example: 

Say we have a struct

```go
ZooEnclosure {
  Resident string
  ...
}
```

If we are writing to a test to make sure our server correctly does all the CRUD operations on the ZooEnclosures as we’d expect, it might be useful to write a helper function that validates the result.

We whip up a quick function: 
```go
ValidateZooEnclosures(
	enclosures []ZooEnclosure,
	expectedResident string
) error {
	var targetEnclosure ZooEnclosure
	for _, enclosure := range enclosures {
		if enclosure.Resident == expectedResident {
			targetEnclosure = enclosure
			break
		}
	}
	found := targetEnclosure != nil
	assert.True(found)
	hasExpectedResident := targetEnclosure.Resident == expectedResident
	assert.True(hasExpectedResident)
}
```

But say in our tests we also want to reuse this function to check for the complete absence of a `ZooEnclosure` from the returned list. So we add this check after our loop:

```go
ValidateZooEnclosures(
	enclosures []ZooEnclosure,
	expectedResident string
) error {
	var targetEnclosure ZooEnclosure
	for _, enclosure := range enclosures {
		if enclosure.Resident == expectedResident {
			targetEnclosure = enclosure
			break
		}
	}
	found := targetEnclosure != nil
	if expectedResident == “” {
		assert.False(found)
		return
	}
	assert.True(found)
	hasExpectedResident := targetEnclosure.Resident == expectedResident
	assert.True(hasExpectedResident)
}

```

So now when using the function, we don’t need another parameter, we just have to pass an empty string for the `expectedResident` to tell the function to assert the absence. The fewer params have to be passed, the better, right! Saved memory and all that.

Well, sometimes! But in this case a `ZooEnclosure` may exist without a `Resident`, say it's being prepared for another exhibit or the penguins finally escaped and made it to Antartica. So it's entirely possible for someone to use an empty string as a valid value they're checking for. To reuse a variable for a disparate purpose by relying on a hardcoded value that "flags" a different branch of logic is obtuse and sneaky.

If another engineer comes along later to debug the tests they are going to have to spend a minute reconciling the variable name with its usage and the assert beneath it. Then they'll need to think through why an empty string would signify that the object shouldn't exist in the array not knowing it was a usage convention defined by an author long ago (at least three months). And this kind of friction accumulates. Especially when reading legacy code, and everything is unfamiliar and every shadow might hold a venomous bug ready to stab me in the back of my tender, soft, programmer hands. Instead consider this slightly more verbose function and ask yourself which you would rather come across in a strange and alien codebase.

```go
ValidateZooEnclosures(
	enclosures []ZooEnclosure,
	expectedResident string,
	shouldExist bool
) error {
	var targetEnclosure ZooEnclosure
	for _, enclosure := range enclosures {
		if enclosure.Resident == expectedResident {
			targetEnclosure = enclosure
			break
		}
	}
	found := targetEnclosure != nil
	if shouldExist {
		assert.True(found)
		hasExpectedResident := targetEnclosure.Resident == expectedResident
		assert.True(hasExpectedResident)
		return
	}
	assert.False(found)
}
```

## Conclusion
Especially in tests, readability is king. So unless you’re writing an embedded program on a mission-critical RTOS, damn the elegance and consider, “Why use one variable, when two would do the trick?” 