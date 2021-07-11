---
title: "Deno is a Browser for Code"
description: Explaining the different mental model Deno has about managing
  dependant code.
date: 2020-05-28T12:00:17+10:00
comments: true
summary: Explaining the different mental model Deno has about managing
  dependant code.
tags:
  - deno
  - package manager
  - registry
author:
  name: Kitson Kelly
  image: /images/kpk_512.jpeg
menu:
  sidebar:
    name: Deno is a Browser for Code
    identifier: deno-is-a-browser-for-code
    parent: Deno
    weight: 100
---

I started contributing to Deno soon after Ry made the prototype visible in
May 2018. The most frequent question that people have is "where is the package
manager?" which often times isn't even in the form of a question. It is
statements like "I thought Deno took security seriously, and just downloading
resources off the internet is insecure." or "How can I possibly manage my
dependencies?"

In my opinion, we need to shift our mental model. Lots of folks take the
ubiquity of package managers and centralized code registries as a requirement to
have a package manager and a centralized code registries. Because they exist
doesn't mean they are required. They came into existence because they solved
problems in a particular way, and we have just accepted them as the only way to
solve that problem. I would argue that isn't true.

## Browsers

In order to publish a website, we don't login to a central Google server, and
upload our website to the registry. Then if someone wants to view our website,
they use a command line tool, which adds an entry to our `browser.json` file on
our local machine and goes and fetches the whole website, plus any other
websites that the one website links to to our local `websites` directory before
we then fire up our browser to actually look at the website. That would be
insane, right? So why accept that model for running code?

The Deno CLI works like a browser, but for code. You import a URL in the code
and Deno will go and fetch that code and cache it locally, just like a browser.
Also, like a browser, your code runs in a sandbox, which has zero trust of the
code you are running, irrespective of the source. You, the person invoking the
code, get to tell that code what it can and can't do, externally. Also, like a
browser, code can ask you permission to do things, which you can choose to grant
or deny.

The HTTP protocol provides everything that is needed to provide information
about the code, and Deno tries to fully leverage that protocol, without having
to create a new protocol.

## Discovering code

The first thing to think about is that, like a browser, the Deno CLI doesn't
want to have any opinions about what code you run. It lays out the rules of how
code is fetched, and how it sandboxes itself from the machine it runs on. In my
opinion, that is as much of an opinion a runtime should have.

In the Node.js/npm ecosystem, we have conflated the management of code on our
local machine, with a centralized registry of code to help facilitate discovery.
In my opinion, both have really bad flaws.

Back in the early days of the internet, we experimented with npm type of
discoverability. You would go add your website to Yahoo! under the right
categorization and people would come along, maybe use the search function, but
it was all structured based on the opinions of those providing the content, not
really based on optimizing for the needs of the consumer. Eventually along came
Google. Why did Google win? Because it was useful. It indexed websites in a way
that matched simple expressions of need (search terms) with the most relevant
web pages that met that need, looking at multiple factors, including meta data
provided the content provider as one factor in the mix.

While we don't have that model quite yet for code for Deno, it is a model that
works. In addition, we use Google because it solves problems for us, instead of
being told "you must use Google", as well as there are also other viable
alternatives to Google.

I got into a bit of a debate with Laurie Voss on twitter, someone who knows a
fair deal about the npm ecosystem I would say. He argued that Deno needed a
package manager, and this blog post is a longer winded version of the thoughts I
wanted to express, but Laurie raised a very valid point.

{{< tweet 1261140647056076801 >}}

GitHub has become the home for open source code, because it was useful and
solved problems, and built on top of the _de facto_ source code versioning tool,
git. From the Deno CLI perspective, there should be no technical restrictions to
where you source code from, it is up to the wider eco-system to create and
evolve ways to make code for Deno discoverable, probably in innovative ways that
could never have been conceived by those of us creating the CLI.

## Repeatable builds

In the npm eco-system, this became a problem. Because of the heavy reliance on
semantic versioning, and the complex dependency graphs that tend to come from
the Node.js/npm eco-system, having a repeatable build became a real problem.
Yarn introduced the concept of lock files, of which npm followed suit.

My personal feeling is it was a bit of the tail wagging the dog, in that the
behaviours of developers in the eco-system created a problem that then needed an
imperfect solution to fix it. Any of us that have lived with the eco-system for
a long time know that the fix to a lot of issues is
`rm -rf node_modules package-lock.json && npm install`.

![](https://memegenerator.net/img/instances/75583685/have-you-tried-rm-rf-node-modules-npm-install.jpg)

That being said, Deno has two solutions for that. First, is that Deno caches
modules. That cache can be checked into your source control, and the
`--cached-only` flag will ensure that there is not attempts to retrieve remote
modules. The `DENO_DIR` environment variable can be used to specify where the
cache is located to provide further flexibility.

Second, Deno supports lock files. `--lock lock.json --lock-write` would write
out a lock file with hashes of all the dependencies for a given workload. This
would be used to validate future runs when the `--lock lock.json` is used.

There are also a couple other commands that make managing repeatable builds.
`deno cache` would resolve all the dependencies for a supplied module and
populate the Deno cache. `deno bundle` can be used to generate a single file
"build" of a workload which all the dependencies are resolved and included in
that file, so only that single file is needed for future `deno run` commands.

## Trusting code

This is another area where I think we have a skewed mental model. For whatever
reason, we put trust in code that is in a centralized registry. We don't even
think about it. Not only that, we trust that that code has fully vetted all of
its dependencies and that those are to be trusted to. We do a quick search and
type in `npm install some-random-package` and think "This is Fine!" I argue the
rich npm package eco-system has lulled is into a sense of complacency.

To compensate for this laxness and complacency, we implement security monitoring
software in our tool chains, to analyse our dependencies and the thousands upon
thousand lines of code to let us know that maybe some of the code is
exploitable. Corporations setup private registries to host packages that might
be vetted slightly more than the single public registry.

It feels like there is an elephant in the room here. The best strategy is we
shouldn't trust any code. Once we have that established, then opening it back up
becomes a little be easier. But we are lying to ourselves if we think a package
manager and a centralised registry solve this problem, or even substantially
help with this problem. In fact, I argue they make use let our guards down.
"Well it is on npm, if it were bad for me, surely someone would take it down."

Deno in this aspect isn't quite as done as I think it should be, but it is
starting from a good position. It has zero trust at startup, and provides fairly
fine grained permissions. One of the things I personally dislike is that there
is the `-A` flag, which is basically saying "oh yeah allow everything" which is
such an easy thing for a frustrated developer to do instead of figuring out what
they really need.

It is also hard to break down those permissions, to say "this code can do this,
but this other code over here can't" or when code prompts to escalate privileges
where is that code coming from. Hopefully we can figure out an easy to use
mechanism coupled with something that would be effective and performant at
runtime to try to solve those challenges.

A recent change though, which is a good one, in my opinion, is that Deno no
longer allows you to downgrade your imports. If something is imported from
`https://` then it can only import from other `https://` locations. This follows
the browser model of not being able to downgrade transport. I still think longer
term it would be good to kill off any remote imports that aren't over
`https://`, much like Service Workers require HTTPS, so we will see what the
future holds.

## Dependency management

I think we need to talk frankly about dependencies in the npm ecosystem. To be
honest, it is broken. An ecosystem that enables
[5 lines of code](https://github.com/juliangruber/isarray/blob/master/index.js)
to be downloaded and installed
[_30 million_ times a week](https://www.npmjs.com/package/isarray) for code that
has been in every browser for the last 9 years and never was needed in Node.js
is a broken ecosystem. This one example, the actual code is 132 bytes, but the
package size is 3.4kb. The runnable code is 3.8% of the package size. "This is
Fine!"

My opinion is that there are several factors involved in this. A big part of it
is that we have the model inverted, which I talked about Deno being a browser
for code. The problem is that this backwards model has infected how we create
websites. While we don't have a central registry, when we build a website, we
download all the code we depend up and bake it into something that we load up on
a server, and then each user downloads a bunch of code to their local machine.
Some evidence is that only around 10% of that code that is downloaded is unique
to that site or web application, the rest is all that code we are downloading to
our development workstation and bundling up. This model being broken are some of
the problems solutions like [Snowpack](https://www.snowpack.dev/) are trying to
solve.

Another significant problem is that our dependencies are not coupled with our
code. We put dependencies in our `package.json` but if our code actually uses
that code or not is totally decoupled. While our code expresses what we are
using out of that other code, it is very loosely coupled to the version of that
code. That is contained in the `package.json`, though it has the biggest impact
on the code we write, because it is the code that is actually consuming the
dependent code.

This leads us to the Deno model, which I like to call _Deps-in-JS_, since all
the cool kids are doing _*-in-JS_ things. Explicitly stating our external
dependencies as URLs means that the code depends upon the other code is concise
and clear, and our code and dependencies are tightly coupled together. If you
want to see that dependency graph, you simply need to use `deno info` with a
local or remote module:

```shell
$ deno info https://deno.land/x/oak/examples/server.ts
local: $deno/deps/https/deno.land/d355242ae8430f3116c34165bdae5c156dca21aeef521e45acb51fcd21c9f724
type: TypeScript
compiled: $deno/gen/https/deno.land/x/oak/examples/server.ts.js
map: $deno/gen/https/deno.land/x/oak/examples/server.ts.js.map
deps:
https://deno.land/x/oak/examples/server.ts
  ├── https://deno.land/std@0.53.0/fmt/colors.ts
  └─┬ https://deno.land/x/oak/mod.ts
    ├─┬ https://deno.land/x/oak/application.ts
    │ ├─┬ https://deno.land/x/oak/context.ts
    │ │ ├── https://deno.land/x/oak/cookies.ts
    │ │ ├─┬ https://deno.land/x/oak/httpError.ts
    │ │ │ └─┬ https://deno.land/x/oak/deps.ts
    │ │ │   ├── https://deno.land/std@0.53.0/hash/sha256.ts
    │ │ │   ├─┬ https://deno.land/std@0.53.0/http/server.ts
    │ │ │   │ ├── https://deno.land/std@0.53.0/encoding/utf8.ts
    │ │ │   │ ├─┬ https://deno.land/std@0.53.0/io/bufio.ts
    │ │ │   │ │ ├─┬ https://deno.land/std@0.53.0/io/util.ts
--snip--
```

Deno has no strong opinions around "versions" of code. A URL is a URL is a URL.
While Deno requires an appropriate media type in order to understand how to
treat code, all the "opinions" about what code to serve up is left up to the web
server. A server can implement semantic versioning to its hearts content, or do
any sort of "magical" mapping of URLs to resources it wants. Deno doesn't care.
For example `https://deno.land/x/` is effectively nothing but a URL redirect
server, where it rewrites URLs to include a git commit-ish reference in the
redirected URL. So `https://deno.land/x/oak@v4.0.0/mod.ts` becomes
`https://raw.githubusercontent.com/oakserver/oak/v4.0.0/mod.ts`, which GitHub
serves up a nice versioned module.

Of course spreading "versioned" remote URLs throughout your codebase doesn't
make a lot of sense, so don't do that. The great thing about the dependencies
just being code though is that you can structure them any way you want to. A
common convention is to use a `deps.ts` which re-exports all the dependencies
you might want. Take a look at the one for
[oak server](https://deno.land/x/oak@v4.0.0/deps.ts):

```ts
// Copyright 2018-2020 the oak authors. All rights reserved. MIT license.

// This file contains the external dependencies that oak depends upon

// `std` dependencies

export { HmacSha256 } from "https://deno.land/std@0.51.0/hash/sha256.ts";
export {
  Response,
  serve,
  Server,
  ServerRequest,
  serveTLS,
} from "https://deno.land/std@0.51.0/http/server.ts";
export {
  Status,
  STATUS_TEXT,
} from "https://deno.land/std@0.51.0/http/http_status.ts";
export {
  Cookie,
  Cookies,
  delCookie,
  getCookies,
  setCookie,
} from "https://deno.land/std@0.51.0/http/cookie.ts";
export {
  basename,
  extname,
  isAbsolute,
  join,
  normalize,
  parse,
  resolve,
  sep,
} from "https://deno.land/std@0.51.0/path/mod.ts";
export { assert } from "https://deno.land/std@0.51.0/testing/asserts.ts";

// 3rd party dependencies

export {
  contentType,
  lookup,
} from "https://deno.land/x/media_types@v2.3.1/mod.ts";
```

I created oak server and maintained for 18 months through about 40 releases of
Deno and the Deno `std` library, including moving of `media_types` from internal
to oak, out to the `std` library, to only have it be "ejected" from the `std`
library to be its own thing. Not once did I think to myself "hey, I need a
package manager to manage this for me".

One of the benefits of TypeScript is that you can get comprehensive validation
of compatibility of your code with other code. If your dependencies are "raw"
TypeScript written for Deno, this is great, but let's say that you want to take
advantage of pre-processing of the TypeScript to JavaScript, but still have the
ability to consume that remote code safely. Deno supports a couple different
ways to allow that to happen, but the most seamless is the support for the
`X-TypeScript-Types` header. This header indicates to Deno where a types file is
located which can be used when type checking the JavaScript file that you are
depending upon. [Pika CDN](https://pika.dev/cdn) supports this. Any packages
that are available on the CDN that have types associated with them will serve up
that header and Deno will also fetch those types and use that when type checking
the file.

All this being said, there may still be a need to "remap" a remote (or local)
dependency to what is expressed in the code. In this case, the unstable
implementation of [import-maps](https://github.com/WICG/import-maps) can be
used. It is a proposal specification that is part of the W3C incubator where
browser standards come out of. It allows a map to be provided which will map a
particular dependency in code to another resource, be it a local file or a
remote module.

We had it implemented in Deno for an extended period of time, as we had really
hoped that it would become adopted widely. Sadly, it was only an
[origin trial in Chrome](https://chromestatus.com/feature/5315286962012160) and
hasn't gotten wider adoption yet. This led us to putting it behind the
`--unstable` flag for Deno 1.0. My personal opinion is that it is still a big
risk of being a dead end, and should be avoided.

## But, but, but...

I suspect a lot of people are still coming with a list of objections to the
model that Deno has. I think the strategy Deno has tried to take, which I am
very aligned to, is to deal with real problems when they arise. A lot of the
objections I hear are from people who are new to Deno, who haven't worked with
it, who haven't tried to understand that there might be a different way.

All that being said, if we collectively run into a problem and there is a
compelling need to change something in the Deno CLI, I am confident that it will
happen, but a lot of problems simply don't exist, or there are other ways to
solve them that don't require your runtime to have strong opinions or be coupled
to an external programme to manage your code.

So my challenge to you is, flirt a bit with not having a package manager or a
centralised package repository and see how it goes. You might never go back!
