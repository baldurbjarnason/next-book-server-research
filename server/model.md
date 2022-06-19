# The Data Model

Having a nice API and a flexible auth strategy doesn't matter if you don't have any data to share over the wire and it matters even less if you can't _agree_ on what that data should look like.

And if you agree on the data model, you can shuttle it over practically anything. You could build an entire system around shipping USB thumb drives through the post if you wanted to. It'd be impractical but you could do it if both ends agree on the data format.

Or, if you want a marginally less ridiculous example: you could build a document-based annotation system where the end user actually _downloaded_ their annotations and uploaded it themselves to the annotation or bookmarking server.

This is why everybody tries to push their business's favoured data model through standardisation. If you get everybody to agree to a standard, costs go down, quality goes up, and business flexibility increases.

One of the examples that the economist Paul Romer uses to illustrate this is the disposable coffee cup. Before the disposable coffee cup was a standardised commodity, any coffee shop that wanted to offer coffee to go either had to convince their clientele to bring their own travel mugs or they had to invest a _ton_ of money to have a large order of disposable coffee cups made for them especially, with printed branding and everything. More importantly, you had to figure out a way to prevent people from burning their fingers. Turns out there are many different approaches you can use to make a hot paper (or plastic) cup safe to hold. Some of them work well. Some of them are hard to implement. Some of them are very fiddly. The market was filled with solutions ranging from attachable handles, to plastic collars, to plastic handles that wrapped around the paper cup, to paper cups with folded cardboard handles. There was variety and with it came expense.

However, once you standardise the approach, things begin to open up. First it was the styrofoam cup in the 1960s. Once it arrived on the scene, the hacky and awkward alternatives were quickly driven out of the market. It wasn't until years later, when environmental concerns began to rise, that the standard switched almost wholesale to a plastic-coated paper cup with an insulating cardboard collar.

With the styrofoam cup, the costs of providing disposable cups dropped to the floor. The cups became more reliable as everybody standardised on an approach that worked and that the consumers to learn. Now _everybody_ could offer coffee to go. Costs go down. Quality goes up. And because costs go down for _everybody_, even the biggest players in the market will eventually switch.

All of which is to say that we have a choice here. The same way that a coffee shop owner pre-standardisation had a multitude of options for disposable paper cups, we have a variety of options when it comes to annotation data models. The first one being the ostensible styrofoam cup of the web annotations world: the [W3C Web Annotation Data Model](https://www.w3.org/TR/annotation-model/).

## Selector-based approaches

Stepping back a bit, what the W3C approach provides is the ability to target specified ranges or selections of a document using a selector object that has been defined using [JSON-LD](https://json-ld.org/). This means that you could, in theory, use that object in any given data model that has been defined in JSON-LD, such as Activity Streams or [schema.org](https://schema.org/). But, in principle, you'd still be using W3C Web Annotations. You'd just be embedding the critical bit in another format.

### W3C Web Annotations (Data Model)

Web Annotations are the W3C's attempt at delivering a feature-complete annotations format. And feature complete it is. It lets you define:

- Multiple target ranges in multiple documents. You can, in effect, add the same note to a dozen different highlights across a dozen different formats.
- Multiple bodies (notes, for example) both embedded and external (URLs).
- Format-agnostic highlights. You can use a text-quote selector to create a highlight that works on a book _no matter what format it is_. PDF, EPUB, HTML, plain text---in theory your annotation could be applied to them all.
- Apply a custom stylesheet or CSS class to an annotation to customise its rendering.
- Capture archival-related time state so that you can tie the annotation to a specific version of the resource.
- DRM-compatible highlights: you can use Xpath or text position selectors to save an annotation to a document that the annotations software can't actually access itself.
- A variety of predefined semantic types of annotations: bookmarking, assessing, commenting, describing, editing, highlighting, linking, and more.

It offers _a lot_. It should, in theory, work for everybody. If you have a need, W3C Web Annotations have a feature to address that need. You need to identify the creator of an annotation using a SHA1 hash of their email? The W3C has you covered (from the spec, linked above):

```
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "http://example.org/anno12",
  "type": "Annotation",
  "creator": {
    "id": "http://example.org/user1",
    "type": "Person",
    "name": "My Pseudonym",
    "nickname": "pseudo",
    "email_sha1": "58bad08927902ff9307b621c54716dcc5083e339"
  },
  "generator": {
    "id": "http://example.org/client1",
    "type": "Software",
    "name": "Code v2.1",
    "homepage": "http://example.org/client1/homepage1"
  },
  "body": "http://example.net/review1",
  "target": "http://example.com/restaurant1"
}
```

Which brings us to the issue with standards like these. Remember the styrofoam cup? It didn't just take over because it was standardised and worked well for mass-production. It was also _simpler_ than most of the solutions it was replacing.

(It also, in economic terms created a massive negative externality in the form of a shit-tonne (technical term) of plastic waste, but that's a topic for a different day.)

Web Annotations? Not simple.

The problem with standards like these that are featureful and support every use case is that if you only care about the simple use case and only implement a subset, you lose many of the benefits of using a standard.

Let's say your needs are simple and your engineering resources are limited. You choose to only support a subset of the standard. Let's say your annotations look something like this (again, from the spec):

```
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "http://example.org/anno23",
  "type": "Annotation",
  "body": "http://example.org/comment1",
  "target": {
    "source": "http://example.org/page1",
    "selector": {
      "type": "TextQuoteSelector",
      "exact": "anotation",
      "prefix": "this is an ",
      "suffix": " that has some"
    }
  }
}
```

This looks fairly doable. There's even an npm module that handles [converting the Text Quote Selector to a DOM Range for you](https://www.npmjs.com/package/dom-anchor-text-quote).

But subsetting inevitably means you lose compatibility. You end up working exclusively within your own space because interoperability will be error prone. So, you have a choice:

1. Stay in your silo
2. Expand your support to cover the entire standard.
3. Or, just pick and choose whichever bit of it is useful, like a scavenger, but ultimately you do your own thing.

Almost every player in the web annotations field has chosen door number three. The annotations formats and API developed by [Hypothesis](https://web.hypothes.is/)---one of the major drivers of the Web Annotations specifications--[bear no more than a passing resemblence to the standards defined by the W3C](https://h.readthedocs.io/en/latest/api-reference/#tag/annotations).

The format is different. The API is different.

Turns out the Web Annotations specifications are standards in name only. They have their users, yes. But they are neither simpler than rolling your own data model nor are they in broad use.

They are neither a styrofoam cup nor are they a paper cup with insulating cardboard collar.

It's still valuable work. You could pick a subset of it and use that as the foundation of your own format, which is essentially what Hypothesis has done. But you wouldn't be using it as a standardised format.

## URL-based approaches

### Activity Streams (Data Model)

### Micropub and microformats

## What's the simplest thing that could work?
