# What if there is no server?

If there is anything we've learned from the past few months, it's that we can't always take infrastructure and resources for granted. A few years ago, before the climate crisis became more apparent, before the pandemic pushed our supply networks to a breaking point, before we had armed conflict on our doorsteps, it looked safe to assume that we'd always be able to buy web services from Amazon Web Services, and that we'd always have the time, manpower, and resources to create flexible and integrated server-client architectures.

The world isn't always safe. But we can make it as safe as we can. One approach we can use is to push as much as we can towards the edges of the networks, to the devices people read on, and make sure they work without a reliable network, at least for the most part.

One way to do this, suggested by Jan Martinek, would be to make a browser extension that leverages existing browser features to let the next-book save highlights and positions.

There are a few different approaches possible here:

## Content Script extension

Inject a content script that takes provides an annotation UI and saves the annotations to annotation storage. That storage is then synced to a storage service whenever possible. This is the most involved.

While this is the most involved of the extension approaches, it might not be _as_ complicated as it looks at first because the goal isn't to create a general purpose annotation system. Instead, the content script would only be injected into web pages that are make with next-book or in a next-book compatible way. Additionally, because extensions can add entries to the native context menu and toolbars, this solution might end up requiring a lot less User Interface work than implementing the same in the web page itself. No need to build an annotation menu or highlight button if you can just add menu items to the context menu.

## Authentication extension

The extension only provides authentication. You add the extension, it lets you sign in to the annotations storage server once and saves an authentication token.

Then it uses the [declarativeNetRequest](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/#type-RuleAction) API to upgrade all API requests to that storage server from next-books to add an Auth Bearer header with an auth token.

This lets you write the annotations code on the page as if it were already authenticated _and_ same origin. This might not save on _that_ much work for a fully-fledge annotations solution but it would solve a few core problems:

- You no longer need to worry about cross-origin state, which is increasingly a complication for authentication as browsers tighten their privacy measures.
- This would _massively_ simplify a simple position-tracking and -saving implementation. If the goal is to save the reader's current position, then that can be done with little UI code.

## Native bookmarking extension

The extension enhances the browser's existing bookmarking system. The browser's existing bookmarking system has the advantage of being automatically synced and saved to the cloud. There's little additional work to be done in terms of saving the reader's annotation beyond, maybe, a feature that lets them export them to a service connected with next-book.

What would make this work is the [text fragments](https://wicg.github.io/scroll-to-text-fragment/#context-terms) specification which is already shipped in Chrome, implemented and about to ship in Safari, and partially underway in Firefox. The extension wouldn't need native support to ship as it would use the text fragments to save the annotation locations to the native bookmarking system but would take care of the highlighting/annotation aspect itself. Native support has the added bonus, though, that if your bookmarked annotations sync to a browser that doesn't have the extension installed, the browser will still highlight the targeted text if you navigate to the bookmark.

So, the extension would save highlights and positions to the browser's existing bookmarks storage with the exact positions saved as [text fragments](https://wicg.github.io/scroll-to-text-fragment/#context-terms). The bookmarks otherwise do not allow complex or strucctured data but you could set annotation types by adding custom suffixes to the bookmark title. Something like "[Yellow Highlight]" or "\[Position\]". When a next-book is loaded or when a highlight is added, the extension searches for all bookmarks for the next-book that contain a text fragment and renders them as highlights. The browser takes care of syncing the bookmarks between devices and saving it to the cloud.

The UI would be provided as context menu items and toolbar items with very little additional effort.

## The Extension experience

Using a browser extension to simplify the annotation experience makes sense as most readers will expect annotations to integrate with the context menu somehow, but in normal web page scripts overriding the context menu is an all-or-nothing affair. You either disable the native menu entirely and replace it with your own, which is usually a pretty bad user experience, or you try to create an extra floating menu somewhere that you try to position in such a way that it doesn't interfere with the native browser menu. Neither approach is optimal.

The extension approach could solve this by providing a more natural method for interacting with and creating annotations.

That it could simplify the storage and synchronisation of annotation data is an added benefit that makes the approach very compelling on the whole, especially now that web extensions seem to be getting cross-browser and cross-platform support.
