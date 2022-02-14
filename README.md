# Research into a book—server communication

_(Below is the initial project outline)._

## Overview

- where a book is a standalone browser app served from static resources, provided at maximum with CORS headers that allow a connection to a 3rd party API (with those headers being possibly configurable by the reader in some way)
- where a server is an app running on a web server that provides an API in any way
- where communication is for now limited to login and one-way sync (push) of a part of the state (the reading position in a book)
- all this in a browser setting, using anything supported in current versions of major browsers and on any server platform

## Objectives

- discover a user-friendly way to login into a server from a book; if the definition is not workable on the server (using only CORS), possibly explore other minimal solutions that might be easily implemented (the goal is to log in from any book served over HTTPS with minimal overhead on the server side, but I’m not sure it’s possible)
- define the data transport parameters (API, data structure, possible problems) that enable that simple data push in the context of a book (web annotations)

## Desired result

- an article/docs and a prototype (pseudocode, working code examples, or any combination) that enable us to quickly check or test the proposed solution, parts of it, or its variations
- a solution built upon previous standards or experiments is welcome :)
- any kind of bibliography is welcome (hyperlinks in the text are, of course, enough of a bibliography, but any notes on the sources or their context are welcome)
- we’d like to publish the result on the next-book.info blog, so HTML compatible result is the best

## Roadmap and timeline

Most of the work on this research project should take place over the next few weeks. The overall work plan is very flexible so expect new additions to come in spurts and corrections and polish to continue even after the main work has ended.

1. Outline at a high level the various possible approaches and strategies while highlighting the most likely candidate for implementation.
2. Go into specific detail on the most promising approaches, including notes on implementation.
3. Detail a proposed approach, with pseudocode or code samples, in such a way that can be used to help test the viability of the approach.
4. If...
   1. ...it doesn't work: go back to _2_ and come up with something else
   2. ...it _does work_: polish the documents from this repository into HTML or markdown for publishing on the blog
