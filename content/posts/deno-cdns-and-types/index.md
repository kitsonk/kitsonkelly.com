---
title: "Deno, CDNs and Types"
description: How types work in Deno and how certain CDNs help.
date: 2021-07-11T11:57:17+10:00
images:
- /posts/deno-skypack-and-types/hero.png
comments: true
summary: Deno and some CDNs offer an easy and powerful way to develop.
tags:
- deno
- editors
- TypeScript
- SkyPack
- esm.sh
- CDNs
author:
  name: Kitson Kelly
  image: /images/kpk_512.jpeg
menu:
  sidebar:
    name: Deno, CDNs and Types
    identifier: deno-cdns-and-types
    parent: Deno
    weight: 99
---

[Deno](https://deno.land/) attempts to make TypeScript a first class language,
as well as well as be akin to a
[browser for code](../deno-is-a-browser-for-code). It allows you to just import
modules from the web, in a similar fashion to navigating to a website in your
browser.

Of course the websites you browser vary greatly, and so do CDNs dedicated to
hosting modules. While [deno.land/x](https://deno.land/x/) aspires to be a
registry of projects specifically tailored to Deno, there are whole classes of
packages and modules not written with Deno in mind.

This is where CDNs like [esm.sh](https://esm.sh) and
[Skypack](https://skypack.dev) come into play. They provide existing packages on
npm as bundled JavaScript with also the ability to provide types as well.

## TypeScript and types

TypeScript has two _worlds_, the _code_ world and the _type_ world. When we
write code in TypeScript those worlds are combined, and most of the time when
authoring TypeScript, we don't see where they join.

Of course when we transpile TypeScript to JavaScript so it can be executed by a
JavaScript engine, all those types are erased, though if we are using `tsc` we
can instruct TypeScript to put those types in a _declaration_ file (`.d.ts`).
This is where the code and the type world are visibly separate.

For example the following `.ts` file might look like this:

```ts
export function a(b: string, c: number = 1): string {
  return b + c;
}
```

And would result in a JavaScript file like this:

```js
export function a(b, c = 1) {
  return b + c;
}
```

And a declaration (`.d.ts`) file like this:

```ts
export function a(b: string, c?: number): string;
```

Since browsers and Node.js don't run TypeScript out of the box, most authors
writing packages in TypeScript distribute them as JavaScript with the
declaration files as part of the package.

If authoring JavaScript directly, a way to inform the TypeScript compiler about
the type world that cannot be inferred from usage, authors can annotate the code
with
[JSDoc](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html)
which will provide _most_ of the same type information that can be provided when
writing TypeScript directly. These comments need to be maintained with the code
to allow the TypeScript to enforce the types.

When dealing with existing code in JavaScript, it is also possible to "hand
craft" type definitions. The largest community project is
[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) which
publishes type only packages for existing npm packages in the `@types`
namespace.

## Deno module graph

When you run your programme under Deno, it has to build up a representation of
all the modules that are part of the programme. Deno needs to figure out what
code needs to be transpiled from TypeScript to JavaScript, but also what code is
required to be type checked.

As mentioned above, there are two worlds for TypeScript, and so Deno has two
_slots_ for every dependency, a _code_ slot and a _type_ slot. When resolving a
dependency, Deno fills in these slots. When the whole graph of all the
dependencies has been built up, the graph can be type checked. Deno uses the
built in TypeScript engine to do this (basically `tsc`) and when a module is
requested, Deno provides it with the module in the _type_ slot if available
otherwise the _code_ slot.

Before the programme is ready to be run, any modules in the _code_ slot that
aren't JavaScript get transpiled to JavaScript. Then the programme is ready to
be executed.

You can see how this manifests itself by using `deno info` on a programme.
Modules that are in a _type_ slot get displayed in _italics_. You can see it
more clearly if you use `deno info --json` where a dependency is listed with
both its `"code"` and `"type"` module.

## Resolving a dependency

When importing (or re-exporting) a module in Deno, Deno analyses the dependency
trying to "fill out" the slots. There are a few things that influence this.

One of the first things Deno tries to do is ensure it has access to the module.
If it is a local file system module, it checks to ensure it exists. If it is a
remote module, it looks in the cache to see if it exists and if not, goes and
fetches the remote module.

Once it has the module, Deno determines what sort of media type the module is,
in order to handle it properly. For local files, it relies upon the extension of
the file to determine the media type. For remote files, it is a bit more
complex. It looks at the `Content-Type` header to see if it can determine the
media type. The challenge is that there is no generally acceptable media type of
a TypeScript declaration file (`.d.ts`), as well as some web servers serve
content up as `text/plain`, but we still want people to be able to use code
served up in that way. Deno then has to "guess" at what the media type is in
certain situations.

Once the media type is determined, Deno then figures out which slot the module
should sit in in the dependency graph. There are a few things that impact this:

### Using `@deno-types`

When importing a module, you can instruct Deno that you want to use the import
dependency in the _code_ slot and put something else in the _type_ slot,
irrespective of what the dependency actually is. This can be done by using the
`@deno-types` pragma.

This is used in situations where you, as the importer knows that you have types
that you want to use with the import, which will be provided to the TypeScript
compiler when type checking.

For example, if I have a remote JavaScript module I want to use, but I have
create a declaration file locally with the APIs I am using out of it, I could do
something like this:

```ts
// @deno-types=./api.d.ts
import * as api from "https://example.com/api.js";
```

The `@deno-types` pragma affects only the next import (or re-export statement).

### Using triple-slash reference directive

When you are in a situation where you are providing a JavaScript module, and you
as the provider know where or are providing the types, there are ways of
informing Deno of that, so that the importer doesn't have to provide the
information or know about it.

The first way to do this is by using the re-purposed TypeScript triple-slash
reference directive. Putting this near the top of the file will instruct Deno to
add a dependency and put it in the _type_ slot for this module. For example, if
you are hosting a JavaScript file with the type definitions along-side it could
could do something like this in your JavaScript file:

```js
/// <reference types="./api.d.ts" />
```

### Using the `X-TypeScript-Types` header

Again, if you are hosting a JavaScript module on a web server, like in a package
registry, and you have control over the headers you send a client, you can use
the custom header `X-TypeScript-Types` to inform Deno where the types are for
your JavaScript file, and Deno will add what is provided there as a dependency
and put it in the _type_ slot. So headers from an example above might look
something like this:

```
HTTP/1.1 200 OK
Content-Type: application/javascript; charset=utf-8
Content-Length: 1234
X-TypeScript-Types: ./api.d.ts
```

_NOTE_ This is covered in the Deno manual under
[Types and Type Declarations](https://deno.land/manual@v1.11.5/typescript/types).

## Hosting code for Deno

There are a couple CDNs that make it really easy to consume code in Deno, where
the original author might not have targetted Deno specifically. Especially if
you want to consume code out of the npm ecosystem, it can be challenging, as a
lot of code is still CommonJS modules which Deno doesn't support. In addition,
an author may want to author the code in TypeScript but then emit to JavaScript
with a declaration file, so it can be easily consumed in browsers or Node.js.
[`deno.land/x/`](https://deno.land/x/) is great if your code is specifically
tailored to Deno and potentially written in TypeScript directly, but other CDNs
make it easier to consume from the npm space.

### esm.sh

This CDN was built with Deno in mind (also helping ensure code is available in
mainland China). It uses the Deno `std` library Node.js polyfills for Deno when
bundling up Node.js specific code and by default it provides
`X-TypeScript-Types` header when importing modules/packages. It allows you to
create URLs that very concisely define how to build the bundle of code,
including versions of dependent packages.

You can see more one the [homepage for esm.sh](https://esm.sh).

### Skypack

This CDN is designed to make consuming packages easy in browsers and Deno.
Skypack specifically detects the Deno user agent and will do its best to serve
up a bundle of code that is designed to work under Deno (it also detects the
user agent of a browser and adjust the bundle for that browser as well).

If you add `?dts` to the end of a package URL, Skypack will set the
`X-TypeScript-Types` to allow Deno to automatically discover the types
associated with the package. Skypack not only looks at the packaging meta data
for the package, but also tries to marry up types from DefinitelyTyped/`@types`
to the package if it doesn't provide them itself. This means there is generally
really good coverage of types for use by Deno when importing from Skypack with
`?dts`.

Currently, Skypack uses the browserify polyfills for Node.js packages, which
means that Node.js modules dependent on `fs`, `crypto`, `http` and `https`
built-in modules won't work under Deno. (I didn't specifically realise this and
will try to chat to them about how the Deno team might be able to provide
specific polyfills for them to use.)

Skypack also includes a quality score for packages, which is really useful when
you are searching for some code.

You can get started with Skypack at [skypack.dev](https://skypack.dev) as well
as there is some
[Deno specific documentation](https://docs.skypack.dev/skypack-cdn/code/deno).

## The Deno Language Server

The module resolution logic and other features described above are built into
the Deno Language Server which is part of the `deno` executable. This means that
when you import a module from one of the CDNs you automatically get type
information which then can provide auto-completion and hover support in your
editor.

In addition, the language server supports
["import completions"](https://github.com/denoland/vscode_deno/blob/main/docs/ImportCompletions.md)
which is a protocol that was created for Deno to make it possible to provide
auto-completions when completing an import in an editor. This means with the
likes of `deno.land/x/` you effectively "browse" the package registry from your
editor.

We need to [enhance the protocol](https://github.com/denoland/deno/issues/10051)
so that it can support very large registries/CDNs like Skypack as well as allow
a CDN to provide more rich information about a package in the IDE, making it
even easier to discover code for use in Deno.

There is the official
[vscode extension for Deno](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno)
which leverages the Deno Language Server as well as
[availability in other editors](https://deno.land/manual@v1.11.5/getting_started/setup_your_environment#lsp-clients).
