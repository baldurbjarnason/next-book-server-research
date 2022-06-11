# The API (or, how you get stuff from here to there)

## The value of standard APIs

We're all familiar with how useful standardisation has been when it comes to browser APIs. The W3C has been invaluable when it comes to making sure that the browsers we use behave in (mostly) the same way.

So, the same should apply to APIs that go over the network, right?

Right?

Unfortunately, this is where the ideals preached by theory and practice diverge. When you're a browser vendor, trying to find a balance between interoperability and competitiveness, you have plenty of incentive to make sure that the APIs being designed are implementable and maintenable. And if it's neither, then it gets backed out. At least for a while.

MathML support, for example, was dropped by Chrome a few years ago and is only now [being re-developed](https://bugs.chromium.org/p/chromium/issues/detail?id=6606). For a browser API to be standardised, implemented across multiple browser engines, and _stay_ supported, it needs to be well thought out. For the most part, that rule seems to apply.

The network APIs standardised by the W3C or the IETF have next to no such limiting incentives. W3C's history when it comes to HTTP APIs is littered with disaster. [SOAP](https://en.wikipedia.org/wiki/SOAP) and the [Web Services standards](https://en.wikipedia.org/wiki/List_of_web_service_specifications) have a deserved reputation for complexity and are almost exclusively used in aging enterprise applications. IETF has it's failures as well such as the [Atom Publishing Protocol](https://datatracker.ietf.org/doc/html/rfc5023) an API for weblogs and CMSes that isn't used by any major blog service or CMS everywhere.

This means that a healthy scepticism is warranted when you're assessing standardised APIs. There is a real risk that they come with baggage, both in terms of implementation and in terms of _ecosystem_. These APIs come in a variety of shapes and sizes. Some are complex; probably too complex. Others are simple; probably too simple. And finally, there are the _implicit_ standards, the conventions that form around idiomatic use of HTTP, its structure and its grammar.

## Micropub

On the probably too simple end we have [`micropub`](https://indieweb.org/Micropub) which is laser-focused on being a general-purpose API for blogging. It didn't originate with the W3C as it was developed and implemented by the IndieWeb community before being brought to the W3C for standardisation. Micropub is tied to the [microformats concept](http://microformats.org/) so using it would require you to build or extend a microformat which can be used for annotations and bookmarking. And unfortunately, very few people outside of the indieweb scene uses microformats or micropub these days.

Almost the entirety of the micropub API consists of relatively simple `POST` requests, where the payload is a urlencoded (or JSON-encoded) microformat to a single web endpoint. It's fairly simple to build and use. And the indieweb scene has a decent set of readymade libraries that support their formats.

The [specification itself](https://micropub.spec.indieweb.org/) is fairly readable as far as specifications go.

Here's an example of saving a [bookmark](https://indieweb.org/bookmark) from the indiweb page:

```
POST /micropub HTTP/1.1
Host: aaronparecki.com
Content-type: application/x-www-form-urlencoded

h=entry
&bookmark-of=https%3A%2F%2Fplus.google.com%2F%2BKartikPrabhu%2Fposts%2FUzKErSbfmHq
&name=To+everyone+who+is+complaining+about+Popular+Science+shutting+down+comments...
&content=%22Why+is+there+this+expectation+that+every+website+should+be+a+forum%3F+No+website+has+any+obligation+to+provide+a+space+for+your+rants.+Use+your+own+space+on+the+web+to+do+that.%22
&category[]=indieweb&category[]=comments
```

As a specification it only concerns itself with the create, update, and delete aspect of each entry. It doesn't specify how you discover earlier entries to update or delete but it's generally expected that you either just use feeds ([JSON Feed](https://www.jsonfeed.org/) is the modern JSON version of the concept) or just literally scrape the blog's HTML using microformats2. Basically, you can use whatever is the simplest for your use case.

The issue with micropub is that while [microformats2](http://microformats.org/wiki/microformats2) is an open-ended and extensible format, it's also generally not used that way. Most of the microformats that people use are the standard ones. This means that, if the fairly small robust community is micropub's strength, you lose that advantage as soon as you start adding complex annotation extensions to the format.

However...

If your annotation needs can be expressed in the form of a _URL_, then it might be a strong fit.

A microformat entry can be a bookmark. So, it can easily express something like a specific location in a text _provided_ that location can be expressed as a fragment identifier. That's easy if you have a document where every major element has an `id` attribute, but what if they don't or what if you need to highlight a run of text (in other words: bookmark a text selection)?

In that case, micropub wouldn't really work.

Unless...

One possibility is the proposed [text fragment standard](https://wicg.github.io/scroll-to-text-fragment/) proposed by the Chrome team. It's already shipping in Chrome and even though it hasn't shipped in other browsers, they don't seem opposed to it in principle either.

With a text fragment you can link to and highlight a specific run of text using the URL alone, without a complex annotation selector. It obviously isn't as flexible or powerful as W3C Web Annotations but it's simple to use and comes with [a robust JavaScript implementation that's maintained by Google](https://www.npmjs.com/package/text-fragments-polyfill). The utilities provided by that module could be used to implement fragment matching for saving highlights using the micropub API.

Web Annotations is a powerful format. It offers a lot of features beyond text fragment matching, such as XPath selectors, CSS selectors, text position selectors, ranges, style customisation, and more. If you need the features, Web Annotations provide.

If, however, your annotation needs can be covered by links, using element id attributes or text fragments, then micropub might well be the strongest, simplest option.

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

## The two main alternatives in more detail (micropub and a simplified Web Annotations Protocol subset)
