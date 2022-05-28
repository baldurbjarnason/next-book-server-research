### ActivityPub

W3C's ActivityPub looks _great_ for this on paper:

- Standard client-server API
- Standard suggestions for client-server auth
- Standard JSON-LD schema for data transport that provides you with a full set of activities such as notes, articles, and likes and is compatible with other JSON-LD specifications
- Standard server-to-server API (so you could get around CORS issues, for example)
- Growing existing ecosystem

Looks great! It's a standard! Ship it! Right!

Right?

It is in the nature of standards to grow and escalate complexity to the point where a new implementation from scratch is unfeasible for one very simple reason: complex working software inevitably grows from simple working software.

(Those of sharp withs and canny minds will spot that as a variation of [Gall's Law](<https://en.wikipedia.org/wiki/John_Gall_(author)#Gall's_law>). Many in software don't seem to register that it applies to them so by reformulating to be specific to software will hopefully cut a bit deeper.)

This means that we can't just answer every question on APIs or software design with 'just use the standard!'. Those are very often unusable for new projects and are only viable if you were one of the initial implementations of that standard.

This is why, in my mind [ActivityPub](https://www.w3.org/TR/activitypub/) isn't an option for projects like this one. In theory, should perform as a great foundation for distributed annotations.

But it's also a standard that has, for the past six years, been used as the foundation for a growing ecosystem of federated social media apps. That part _isn't necessarily a feature_ if you are just starting to implement it. ActivityPub isn't just a single specification any more. To be fully compliant you need to implement HTTPS Signatures, Linked Data Signatures, W3ID related features, and make sure you are compatible with all existing implementations such as Mastodon. Committing to it would inevitably present you with a conundrum. You could either:

1. Just use the simplest possible version of ActivityPub that would work for the project and just ignore the rest. Doable but opens you up to constant questioning as to why you don't "just" implement full compatibility.
2. Build a fully-featured federated annotation server that you ship, with great pride and joy, in 2042.

A fully featured federated annotations server would be _very cool_ but it's very cool in the same way as fantasies about retiring to a luxurious lunar colony are very cool. It's not going to happen.

### W3C Web Annotations (API)

### Micropub

## A high level proposal for pushing reader position back to a reader-chosen API server
