# Goal: The Simplest Thing That Could Possibly Work

Would be to do nothing. But that's obviously not helpful. 🙂

The requirements for the initial version are:

> - discover a user-friendly way to login into a server from a book; if the definition is not workable on the server (using only CORS), possibly explore other minimal solutions that might be easily implemented (the goal is to log in from any book served over HTTPS with minimal overhead on the server side, but I’m not sure it’s possible)
>
> - define the data transport parameters (API, data structure, possible problems) that enable that simple data push in the context of a book (web annotations)

And you also don't want to paint yourself in a corner and make it impossible for you to add other features in the future.

So, what we want is a standard, not-too-controversial, way of signing in to your chosen annotations server from a book you're reading and a standard, not-too-controversial, definition of the data transport for said annotations. With a focus on implementation simplicity on both ends.

## Let's get the obvious out of the way&mdash;why ActivityPub isn't an option

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

## The other players

We have three problems we need to solve:

1. A reader needs to be able to authenticate themselves from a statically generated web-book, to a third party annotation server, with as little involvement from the web-book hosting server as it possible.
2. The annotations need to be in a sensible, preferably standard form.
3. Using an API that is sensible and standard.

And I'd add the fourth requirement:

4. Has the opportunity to grow to include other annotation features in the future (highlighting, sync, etc.)

Of these, the first requirement is the trickiest because auth is always hard.

### Personal Access Tokens (PAT)

The absolute simplest option would be to use pre-generated Personal Access Tokens for auth, much like [GitHub offers](https://github.blog/2021-04-05-behind-githubs-new-authentication-token-formats/).

The user would log in to a server by filling a form that somehow indicated the target server URL along with a pasted access token. Subsequently, the web-book would send annotations to the target server with the access token in the "Authorization: Bearer ..." http header.

The biggest technical issue is the need to store the token securely but there are a few ways that can be done safely.

It's relatively simple to implement both on the server and client.

However, it's usability is... poor. It requires the user to manage and copy-paste a number of access token which, although safer than requiring password authentication, is error-prone and often confusing.

The PAT approach could be used for the initial implementation as almost all of the auth alternatives require you to figure out token storage and token auth for requests anyway. But you'd have to switch to a more user-friendly way to sign in before you start to test with regular users. (Unless your core audience is the dev crowd who are used to this sort of bare-boned approach.)

### Oauth 2.0 (Build Your Own Auth)

The good thing about our requirements is that we don't need to take care of actually authenticating the end-user. We only care about getting an auth token to let us push data to the target server, letting the reader choose where their data is stored. Oauth should be tailor-made to solve this problem.

The problem is that it's more of a tailor's rough cut: they've measured our general height and weight but haven't actually cut the material to measure, let alone started sowing it together.

What I'm trying to say is that Oauth is a standard for making standards. All Oauth systems work _mostly_ the same way, but almost never _exactly_ the same way. It can get tricky but there are a few libraries out there that would let the target server implement Oauth and there are services, such as Auth0, that aim to implement most of it for you. It can get tricky, though.

The client-side can be done with little to no server involvement, which is one of our goals. However, it is generally recommended that you save the Refresh Token (the long lived token that represents the grant of access to the target server to your client app) somewhere safe, which usually means serverside storage. That means that to use Oauth 2.0 securely, you generally need a server endpoint that saves the refresh token and will request a new access token for the front end whenever it needs it.

### Oauth (IndieAuth)

One option would be to use a more narrowly specified profile of Oauth 2.0, one that covers the most common use cases you're likely to need. [IndieAuth](https://indieauth.spec.indieweb.org/) is one such profile. It makes a set of assumptions about your use case&mdash;which algorithms to use, how you plan on preventing Man-In-The-Middle attacks, how you plan on using tokens, how you identify apps and users&mdash;and in doing so it simplifies implementation somewhat. Instead of having to, essentially, design your particular Oath service, you build it according to the specific IndieAuth recipe.

The biggest limitation in IndieAuth is that everything is identified as a URL. The biggest strength of IndieAuth is that everything is identified as a URL.

It's a strength because it standardises the various kinds of endpoint discovery on using `<link>` elements in the HTML of the identifying URL. It's a weakness because end-user understanding of "URLs" is often quite vague.

That weakness could be mitigated with UX design. For example, if you know that most of your readers will sign in to push their annotation data to a target sever hosted by a library and that their identifying URL is always of the form "https://library.example.com/user/email%40gmail.com" then you could add a "Sign in With Example Library" button that asks the user for their email address before starting the IndieAuth flow.

So, IndieAuth might be a way to simplify the implentation of an Oauth 2.0 system if Oauth gets chosen.

### Other Login/Auth options?

### W3C Web Annotations (Data Model)

### W3C Web Annotations (API)

### Micropub

## A high level proposal for pushing reader position back to a reader-chosen API server
