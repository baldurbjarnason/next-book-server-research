# Authentication

There are, roughly, two ways of going about this: the simplest thing and the not quite as simple.

## The simplest thing (for both dev and end user) that could possibly work: server-hosted JS libraries

The first tool that everybody reaches for these days is for a OAuth-based API server. It might be RESTful. It might use GraphQL. Either way, it almost always uses Oauth2. That's the problem it's designed to solve, right? The problem is that Oauth isn't exactly straightforward. It isn't _hard_, per se. Just not straightforward. So it shouldn't surprise you that Oauth isn't the simplest thing that could possibly work.

The simplest thing is that the server that provides the API also provides _and hosts_ a frontend JS component for accessing that API.

A fairly typical example is the [Chooser component provided by Dropbox](https://www.dropbox.com/developers/chooser).

1. The developer of the frontend app creates an app in Dropbox's developer platform.
2. Gets an embed `<script>` tag that is placed in the frontend, rendering a component.
3. The end user interacts with the component, which can call Dropbox API endpoints because it's hosted on a Dropbox domain.
4. The developer's page gets the response without ever having to worry about authentication.

You could even use Dropbox's Chooser and Saver components for the intitial prototype work, provided it makes sense to store all of the data in a flat file somewhere.

But even if we need APIs that are annotation-specific, you could still use the embedded component approach. You need a button somewhere in the UI that the end user presses to indicate that you should use that component to save the annotations. At which point it probably makes sense for it to transform from a button to a small widget indicating how much data or how many annotations have been saved from the current book.

### Pros

1. This approach has the benefit of simplicity. People have been using components like these since the dawn of the web. It's a known quantity. Fastest way to get up and running.
2. No need to worry about auth on the front end. The server needs to handle it in some way but that's a given for all of the approaches.
3. Lets you wrap the annotation HTTP API in a nice JS library and abstract away the actual protocol and data model on the wire.

### Cons

1. This approach works best if each web book only plans to work with a small set of potential annotation servers. Unlike some Oauth approaches, this won't let the end user type in an API URL of their choosing. An ideal situation for this approach would be something like a library-hosted web book that works with a short list of annotation servers that are likely to represent their end users.
2. The more annotation servers you want to support, the less appropriate this approach becomes.
3. The annotation server _should_ be hosted on its own domain, separate from other services. Even though this approach is fairly secure, it is a way to bypass some of the CORS security features of the web platform so you'd want the annotations server siloed if something goes wrong.
4. Some testing needs to be done to make sure that the handover of annotation data from the book to the annotation server conforms to the current privacy protections that browsers have in place.

## The not quite as simple, but industry standard: CORS-based APIs using access tokens for authentication

All of the other approaches use tokens for authentication in some way.

## Personal Access Tokens (PAT)

The access token option would be to use pre-generated Personal Access Tokens for auth, much like [GitHub offers](https://github.blog/2021-04-05-behind-githubs-new-authentication-token-formats/).

The user would log in to a server by filling a form that somehow indicated the target server URL along with a pasted access token. Subsequently, the web-book would send annotations to the target server with the access token in the "Authorization: Bearer ..." http header.

The biggest technical issue is the need to store the token securely but there are a few ways that can be done safely.

It's relatively simple to implement both on the server and client.

However, it's usability is... poor. It requires the user to manage and copy-paste a number of access token which, although safer than requiring password authentication, is error-prone and often confusing.

The PAT approach could be used for the initial implementation as almost all of the auth alternatives require you to figure out token storage and token auth for requests anyway. But you'd have to switch to a more user-friendly way to sign in before you start to test with regular users. (Unless your core audience is the dev crowd who are used to this sort of bare-boned approach.)

### Pros

1. Let's you support arbitrary API URLs. You don't really care what server you are sending the annotations to, just so long as it speaks the right protocol and accepts the access token that the user entered with it.
2. Probably the simplest strategy to implement both on the server and on the front end.
3. Great for the dev crowd.

### Cons

1. Hard to use. The UX isn't great. Even if you combine the server URL and the PAT into a single string, that still leaves you with a long opaque string that the end user needs to correctly copy and paste.

## Oauth 2.0 (Build Your Own Auth)

The good thing about our requirements is that we don't need to take care of actually authenticating the end-user. We only care about getting an auth token to let us push data to the target server, letting the reader choose where their data is stored. Oauth should be tailor-made to solve this problem.

The problem is that it's more of a tailor's rough cut: they've measured our general height and weight but haven't actually cut the material to measure, let alone started sowing it together.

What I'm trying to say is that Oauth is a standard for making standards. All Oauth systems work _mostly_ the same way, but almost never _exactly_ the same way. It can get tricky but there are a few libraries out there that would let the target server implement Oauth and there are services, such as Auth0, that aim to implement most of it for you. It can get tricky, though.

The client-side can be done with little to no server involvement, which is one of our goals. However, it is generally recommended that you save the Refresh Token (the long lived token that represents the grant of access to the target server to your client app) somewhere safe, which usually means serverside storage. That means that to use Oauth 2.0 securely, you generally need a server endpoint that saves the refresh token and will request a new access token for the front end whenever it needs it.

### Pros

1. This is what almost everybody else is using. Granted, most of them have VC funding or are huge tech cos. But still, lots of documentation and libraries.
2. It's _flexible_. It works for web apps, native apps, browser extensions, whatever.
3. It can be wrapped in a nice UX.

### Cons

1. Implementing it can be a lot of work.
2. Not simple.
3. It kind of _has to_ be wrapped in a nice UX to work.

### Oauth (IndieAuth)

One option would be to use a more narrowly specified profile of Oauth 2.0, one that covers the most common use cases you're likely to need. [IndieAuth](https://indieauth.spec.indieweb.org/) is one such profile. It makes a set of assumptions about your use case&mdash;which algorithms to use, how you plan on preventing Man-In-The-Middle attacks, how you plan on using tokens, how you identify apps and users&mdash;and in doing so it simplifies implementation somewhat. Instead of having to, essentially, design your particular Oath service, you build it according to the specific IndieAuth recipe.

The biggest limitation in IndieAuth is that everything is identified as a URL. The biggest strength of IndieAuth is that everything is identified as a URL.

It's a strength because it standardises the various kinds of endpoint discovery on using `<link>` elements in the HTML of the identifying URL. It's a weakness because end-user understanding of "URLs" is often quite vague.

That weakness could be mitigated with UX design. For example, if you know that most of your readers will sign in to push their annotation data to a target sever hosted by a library and that their identifying URL is always of the form "https://library.example.com/user/email%40gmail.com" then you could add a "Sign in With Example Library" button that asks the user for their email address before starting the IndieAuth flow.

So, IndieAuth might be a way to simplify the implentation of an Oauth 2.0 system if Oauth gets chosen.

### Pros

1. This is as close to elegant as Oauth 2.0 will ever get.
2. Has most of the advantages of the more general approaches to Oauth 2.0.

### Cons

1. Has most of the disadvantages of the more general approaches to Oauth 2.0.
2. Having the URL front and center is, unfortunately, not particularly user-friendly.
