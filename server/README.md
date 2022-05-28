# Goal: The Simplest Thing That Could Possibly Work

Would be to do nothing. But that's obviously not helpful. 🙂

The requirements for the initial version are:

> - discover a user-friendly way to login into a server from a book; if the definition is not workable on the server (using only CORS), possibly explore other minimal solutions that might be easily implemented (the goal is to log in from any book served over HTTPS with minimal overhead on the server side, but I’m not sure it’s possible)
>
> - define the data transport parameters (API, data structure, possible problems) that enable that simple data push in the context of a book (web annotations)

And you also don't want to paint yourself in a corner and make it impossible for you to add other features in the future.

So, what we want is a standard, not-too-controversial, way of signing in to your chosen annotations server from a book you're reading and a standard, not-too-controversial, definition of the data transport for said annotations. With a focus on implementation simplicity on both ends.

You need to send data from the book to a third party server. It needs to be relatively user-friendly and, preferably, relatively simple to implement.

We have three problems we need to solve:

1. A reader needs to be able to authenticate themselves from a statically generated web-book, to a third party annotation server, with as little involvement from the web-book hosting server as it possible.
2. The annotations need to be in a sensible, preferably standard form.
3. Using an API that is sensible and standard.

And I'd add the fourth requirement:

4. Has the opportunity to grow to include other annotation features in the future (highlighting, sync, etc.)

Of these, the first requirement is the trickiest because auth is always hard.

## 1. [Auth](auth.md)

## 2. [Data Model](model.md)

## 3. [APIs](api.md)
