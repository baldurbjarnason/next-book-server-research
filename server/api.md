# The API (or, how you get stuff from here to there)

As you'd guess, the standardised options here range from the too-simple-to-work all the way to the too-complex-to-work. But there should be stuff in the middle there that works, right?

## Micropub

On the too-simple-to-work end of things we have [`micropub`](https://indieweb.org/Micropub) which is laser-focused on being a general-purpose API for blogging. Micropub is tied to the [microformats concept](http://microformats.org/) so using it would require you to build or extend a microformat which can be used for annotations and bookmarking. And unfortunately, very few people outside of the indieweb scene uses microformats or micropub these days.

Almost the entirety of the micropub API consists of relatively simple `POST` requests to a single web endpoint so it's fairly simple to build and use. And the indieweb scene has a decent set of readymade libraries that support their formats.

But, again, it's focused on blogging. Even though it's simple and has a decent ecosystem, it doesn't support annotation. Extending it would mean losing the benefit of the ecosystem which is generally the point of using it.

## Graphql

On the too complicated end you have [graphql](https://graphql.org/), whose primary purpose is to provide a single, flexible entrypoint to query a mixed bag of data sources. If you're front end is querying multiple databases and APIs then graphql _might_ be what you need, as long as you can avoid it's performance pitfalls (caching and a tendency to fall into extremely unoptimised and slow queries).

But for this use case it's an extremely bad fit. Would take a lot of work for very little reward.

## ActivityPub

W3C's ActivityPub looks _great_ for this on paper:

- Standard client-server API
- Standard suggestions for client-server auth
- Standard JSON-LD schema for data transport that provides you with a full set of activities such as notes, articles, and likes and is compatible with other JSON-LD specifications
- Standard server-to-server API (so you could get around CORS issues, for example)
- Growing existing ecosystem

Looks great! It's a standard! Ship it! Right!

Right?

It is in the nature of standards to grow and escalate complexity to the point where a new implementation from scratch is unfeasible for one very simple reason: complex working software inevitably grows from simple working software.

This is why, in my mind [ActivityPub](https://www.w3.org/TR/activitypub/) isn't an option for projects like this one. In theory, should perform as a great foundation for distributed annotations.

But it's also a standard that has, for the past six years, been used as the foundation for a growing ecosystem of federated social media apps. That part _isn't necessarily a feature_ if you are just starting to implement it. ActivityPub isn't just a single specification any more. To be fully compliant you need to implement HTTPS Signatures, Linked Data Signatures, W3ID related features, and make sure you are compatible with all existing implementations such as Mastodon. Committing to it would inevitably present you with a conundrum. You could either:

1. Just use the simplest possible version of ActivityPub that would work for the project and just ignore the rest. Doable but opens you up to constant questioning as to why you don't "just" implement full compatibility.
2. Build a fully-featured federated annotation server that you ship, with great pride and joy, in 2042.

A fully featured federated annotations server would be _very cool_ but it's very cool in the same way as fantasies about retiring to a luxurious lunar colony are very cool. It's not going to happen.

### W3C Web Annotation Protocol

The [Web Annotation Protocol](https://www.w3.org/TR/annotation-protocol/) looks complicated on paper. It's an implementation of the Linked Data Protocol, recommends supporting esoteric formats such as [Turtle](<https://en.wikipedia.org/wiki/Turtle_(syntax)>), and comes with a bunch of implementation requirements that, at a quick glance, look quite involved. RDF all over the place?!

However, once you start peeling away at it, things start to look simpler. The API endpoints are allowed to only support the Web Annotations Data Model. And if they limit themselves that way, and if youd don't actually care about interoperability with the Linked Data Platform, then the API is a fairly straightforward variation on REST:

- The data model (including paging) is defined in [Web Annotation Data Model](https://www.w3.org/TR/annotation-model/)
- Each user has at least one annotation _container_, which is an endpoint for listing, creating, updating, and deleting annotations
- Status codes tell you what happened
- POST to the container endpoint to create a new annotation
- GET, PUT, and DELETE work as usual.
- Proper use of the `ETag` header is necessary for conditional updates.

Overall, the protocol is a good example of how to use HTTP to create an API with features such as using the `Slug` http header to indicate a preferred URL slug name for a created annotation and more. The complicated bits are all about Linked Data interoperability which I personally think is of limited value.

The protocol doesn't define querying or filtering of the annotation listing. So, either there needs to be a way to create a container for each publication, possibly as a part of the auth process, or there needs to be a way for the API consumer to filter the listing somehow.

One probably sensible default would be for the container to automatically filter the listing based on the `CORS` request. A request from `library.example.com/book1` to `annotations.example.com/user1` would only list annotations that applied to `library.example.com`. That might not be enough if it's likely that single domains might list hundreds or thousands of books. In that case a query parameter on the container endpoint might do the trick. Something like `annotations.example.com/user1?scope=https://library.example.com/book1` to limit the results to the previously defined scope of a publication.

With scope filtering, getting or adding a bookmark for a reader position, for example, wouldn't be too much of an onerous process:

1. Retrieve the user's container URL somehow (probably as a part of the auth process).
2. `GET` the container, possibly filtering by scope (see above) to see the latest annotations.
3. The bookmark annotation (see the data model discussion which will be separate) with the newest `created` or `modified` property should be the reader's furthest position.
4. `POST` a new annotation to the container to add a bookmark for a new position.

Might work?

## A more detailed proposal for how the Web Annotations Protocol could work for an API server
