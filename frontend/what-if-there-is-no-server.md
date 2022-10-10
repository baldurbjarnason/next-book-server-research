# What if there is no server?

If there is anything we've learned from the past few months, it's that we can't always take infrastructure and resources for granted. A few years ago, before the climate crisis became more apparent, before the pandemic pushed our supply networks to a breaking point, before we had armed conflict on our doorsteps, it looked safe to assume that we'd always be able to buy web services from Amazon Web Services, and that we'd always have the time, manpower, and resources to create flexible and integrated server-client architectures.

The world isn't always safe. But we can make it as safe as we can. One approach we can use is to push as much as we can towards the edges of the networks, to the devices people read on, and make sure they work without a reliable network, at least for the most part.

One way to do this, suggested by Jan Martinek, would be to make a browser extension that leverages existing browser features to let the next-book save highlights and positions.

There are a few different approaches possible here:

- Inject a content script that takes provides an annotation UI and saves the annotations to annotation storage. That storage is then synced to a storage service whenever possible. This is the most involved.
- The extension only provides authentication. You add the extension, it lets you sign in to the annotations storage server once. Then it uses the [declarativeNetRequest](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/#type-RuleAction) API to upgrade API requests to that storage server from next-books to add an Auth Bearer header with an auth token.
- The extension enhances the browser's existing bookmarking system. Highlights and positions are saved to the browser's existing bookmarks storage with added [text fragments](https://wicg.github.io/scroll-to-text-fragment/#context-terms) for specific locations. Annotation types can be set by adding custom suffixes to the bookmark title. Something like "[Yellow Highlight]" or "[Position]". When a next-book is loaded or when a highlight is added, the extension searches for all bookmarks for the next-book that contain a text fragment and renders them as highlights. The browser takes care of syncing the bookmarks between devices and saving it to the cloud.
