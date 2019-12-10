---
title: Bundling in Deno
date: 2019-12-09 11:21:54
tags:
  - deno
---

I've been working on [Deno](https://deno.land/) for a while now, and one of the features I am proud of, which I wanted to share a bit more detail on, is how we do bundling.

Deno typically does all the heavy lifting for you.  Modules are just URLs, so you don't need any special tooling to include modules like a package manager. For example if you wanted to run a static web server, you would only need to do something like this:

```sh
> deno run --allow-read --allow-net https://deno.land/x/oak/examples/staticServer.ts
```

Then all the dependencies will be fetched for you, cached, the program compiled, and executed for you.  The next time you run the programme, Deno will realise that all the code, including any remote modules are already compiled and cached locally.

But let's say you are building a server, where you want to distribute the programme?  Well, you could "cache" your Deno cache.  If it is present, then Deno won't try to go and fetch anything remote.  Wouldn't it be better though if you resolved all those dependencies and output a single JavaScript file that wouldn't need anything external to run?

That was the motivation around `deno bundle`, to provide a single JavaScript file with all the dependencies resolved and included in the file.

# ESM

Deno works totally off of ES Modules.  Node.js has a main module format of CommonJS, and only recently allowed loading of ESM without a command line flag.  Evergreen browsers support ESM when you load a JavaScript file via `<script type="module">`, otherwise they are simply treated as global scripts.

So, for Deno, we needed the output of `deno bundle` to be a valid ESM file, but we needed to include all the modules, but effectively ensure that the scope of each module is preserved.  ES Modules cannot be concatenated together in a single file, you end up with an invalid module.  This is true of CommonJS as well.  Bundlers like webpack and rollup do static analysis on code, and figure out how to re-scope a module so that it can be inlined safely.  They use an internal format to manage the instantiation of the different module scopes.

So given the need to generate a valid ES module, and the need to be able to easily ensure the modules behaved in the same way as far as their scope, we needed a way to effectively concatenate all the dependencies.  We could build all the static analysis in, or we could look at a different approach.  The TypeScript compiler supports multiple module output formats, including two that can be concatenated: AMD and SystemJS.  Given that we would effectively have the whole analysis and bundling "for free" it made sense for us to use the TypeScript compiler to output the program to one of those module formats as a single file, which we chose AMD.

Quite a few folks have questioned (and some even (╯°□°)╯︵ ┻━┻) the choice of AMD, as it is an "antiquated technology" holding us back.  My argument is that it is an implementation detail that users are abstracted from and shouldn't worry about it.  The alternative is to re-invent the wheel and create some sort of "module scoping", re-writing each of the modules in a way that they can be inlined into a single file.  All of which is code that doesn't exist, when there is a well-known and well-tested solution that "just works" already built into Deno.

So we built and released the MVP for bundling, which required that you utilise some sort of loader script to bootstrap the bundle.  And it worked, and it was something that we could build on.

# Dog fooding

In Deno, there is a lot of JavaScript and TypeScript code that defines the compiler, the runtime environment, and workers.  In order to speed up boot times we use a feature from V8 called "snapshots", where we essentially run JavaScript code into V8, V8 does all the parsing of the code and loads it into memory.  We then take that memory and save it out to disk, and then build that into the `deno` binary.  So instead of having to read a lot of JavaScript, parse it, and allocate objects in memory, Deno just loads that directly into V8.

In order to create these snapshots, we need to create a single JavaScript script file which can be loaded into the isolate.  Sound familiar?  Up until this point we had been using rollup to transpile the internal TypeScript and generate the single bundle.  But rollup requires Node.js, and we wanted to remove Node.js from our build pipeline.

So we effectively ported the `deno bundle` logic into the `deno_typescript` part of Deno, which transpiles our TypeScript and outputs a single bundle for both the compiler and the runtime.  We load these bundles into V8 and then take a snapshot. ([@ry](https://github.com/ry/) did most of this work, while I have done most of the rest of the bundling work)

# Integrating the loader

The next steps we wanted to take was to integrate a loader directly into the output.  Up to this point, you had to `run` an external loader script, which causes some problems (like messing up arguments available to the workload).  It also isn't as UX friendly.  We wanted to be able to do something like this:

```sh
> deno bundle https://deno.land/x/oak/examples/staticServer.ts staticServer.bundle.js
> deno run --allow-net --allow-read staticServer.bundle.js
```

So we integrated the loader into Deno, so when the bundle is output, the loader is inlined and the internalised modules will be instantiated in the same way if we were loading them directly.

# Named exports

We had a couple users of Deno trying to generate bundles that weren't fully standalone programmes, but were in fact libraries that would be imported into another program.  This meant that not only did we need to generate a file that could be loaded as a valid ES module, but we also needed to ensure it exported the same things the original root module exported.  For example, if I wanted to create a single file library of a web server, which I can distribute and import into my programme, I want to be able to do something like this:

```sh
> deno bundle https://deno.land/x/oak/mod.ts oak.bundle.js
```

So I can do the following in my program:

```ts
import * as oak from "./oak.bundle.js";
```

In order to do this, we needed a way to detect what is exported from that root module.  So a bit of experimenting with the TypeScript compiler allowed us to determine that information.  Here is a partial example of how we detected the named exports.  We already have our `ts.Program` which will be used to emit the files, and we know the module name of our `rootModule`:

```ts
const rootSourceFile = program.getSourceFile(rootModule);
const checker = program.getTypeChecker();
const rootSymbol = checker.getSymbolAtLocation(rootSourceFile);
const rootExports = checker
  .getExportsOfModule(rootSymbol)
  .map(sym => sym.getName());
```

At the end of this, `rootExports` will contain an array of strings which are the named exports (including potentially `default`) from the root ES Module.

The one challenge is that the symbols returned with this process contain both things that will be emitted as well as type only exports (like interfaces) which are erased during the emit.  So we need to refine the list a bit further:

```ts
const rootExports = checker
  .getExportsOfModule(rootSymbol)
  .filter(
    sym =>
      !(
        sym.flags & ts.SymbolFlags.Interface ||
        sym.flags & ts.SymbolFlags.TypeLiteral ||
        sym.flags & ts.SymbolFlags.Signature ||
        sym.flags & ts.SymbolFlags.TypeParameter ||
        sym.flags & ts.SymbolFlags.TypeAlias ||
        sym.flags & ts.SymbolFlags.Type ||
        sym.flags & ts.SymbolFlags.Namespace ||
        sym.flags & ts.SymbolFlags.InterfaceExcludes ||
        sym.flags & ts.SymbolFlags.TypeParameterExcludes ||
        sym.flags & ts.SymbolFlags.TypeAliasExcludes
      )
  )
  .map(sym => sym.getName());
```

Then all we have to do, is with the bundle returned from the TypeScript compiler on emit, is to re-export all the named exports from the root module.  We add this to the bundle, and that might look something like this:

```ts
const __rootExports = instantiate("oak");
export const Application = __rootExports["Application"];
export const Context = __rootExports["Context"];
export const HttpError = __rootExports["HttpError"];
export const composeMiddleware = __rootExports["composeMiddleware"];
```
