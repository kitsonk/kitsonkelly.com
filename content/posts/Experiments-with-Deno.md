---
title: Experiments with deno
date: 2018-05-31 09:18:03
tags:
- v8
- typescript
- golang
comments: true
---

I saw the following tweet from Mohsen:

{{< tweet 1001901925015719938 >}}

And I got excited.  While a TypeScript runtime was never an objective or goal of TypeScript, there is an argument that having a runtime, especially for a server side application, is better.

Also, I was recently having a debate with some of my co-workers about how Ryan had stopped contributing to NodeJS because he felt that Go was a better language for creating server side applications.  While I can honestly respect that, seeing him provide a runtime for TypeScript indicates that the ubiquity of JavaScript and TypeScript is hard to deny.

So I was really excited to see if I could get it running.

## Installation on OSX

This is essentially take from the current version of the [readme](https://github.com/ry/deno/blob/c0cc240810f9280ca458c54b4cb69acc30e47e27/README.md#compile-instructions) and modified for common OSX environments.  I have assumed that you have [Homebrew](https://brew.sh/) installed and available as well as the latest full version of XCode with current XCode command line tools.

First you will need Yarn, Go and Protobuf v3:

```
$ brew install yarn
$ brew install go
$ brew install protobuf
```

You should setup a `$GOPATH` environment variable.  go will default to `$HOME/go` if there is no path specified, but that is what I would recommend setting it to anyways.  You also need `$GOPATH/bin` as part of your `$PATH`.

Then you need `protoc-gen-go` and `go-bindata`:

```
$ go get -u github.com/golang/protobuf/protoc-gen-go
$ go get -u github.com/jteeuwen/go-bindata/...
```

You need to get and build `v8worker2`. It takes about 30 minutes to build:

```
$ go get -u github.com/ry/v8worker2
```

I got the following error when doing the above, which is expected, because the build is not able to be done by `go get`:

```
# pkg-config --cflags v8.pc
Failed to open 'v8.pc': No such file or directory
No package 'v8.pc' found
pkg-config: exit status 1
```

You then can do the following:

```
$ cd $GOPATH/src/github.com/ry/v8worker2
$ ./build.py --use_ccache
```

Finally you can get `deno` and its other Go deps.

```
$ go get -u github.com/ry/deno/...
```

```
$ cd $GOPATH/src/github.com/ry/deno
$ make
```

Once it has built, you can validate it is working:

```
$ ./deno testdata/001_hello.js
Hello World
```

I then created a symbolic link of `deno` to `$GOPATH/bin` so that it was then available in my path.

## Experimenting

Once I got it running, I was like _WOAH!_ this is crazy.  I fired up [Visual Studio Code](https://code.visualstudio.com/) and started with a minimal `tsconfig.json` just to ensure that the editor was doing the right thing:

```json
{
    "compilerOptions": {
        "target": "es2017",
        "types": [
            "./deno"
        ]
    },
    "files": [
        "./index.ts"
    ]
}
```

There is a very limited set of APIs above what v8 provides.  But to ensure vscode understood what was available, I copied over the [deno.d.ts](https://github.com/ry/deno/blob/master/deno.d.ts) and added it to the `types` in the config.

To access this minimal API, you simply import it then as a module, all in ES6 format:

```ts
import * as deno from 'deno';
```

I wanted to try these APIs, just to see if I could do something interesting.  There is the `readFileSync` API, which returns a `Uint8Array`.  So I created two little functions to provide reading text and JSON into code:

```ts
export function readTextSync(filename: string): string {
    return new TextDecoder('utf-8').decode(deno.readFileSync(filename));
}

export function readJsonSync<T>(filename: string): T {
    return JSON.parse(readTextSync(filename));
}
```

The simple fact that the code I was editing, in TypeScript syntax, was actually going to be run, was a little mind boggling, but I liked it.  If you do have a TypeScript syntax error, `deno` will pretty print out the error message for you:

```
/deno-experiments/index.ts:7:43 - error TS2304: Cannot find name 'Foo'.

7 export function readJsonSync<T>(filename: Foo): T {
                                            ~~~
```

And if you are using vscode, Cmd/Ctrl clicking on the position in the console will take you right to the code (which was also red in vscode).

So I created a `test.json` file and read it in and logged it to the console.  Worked like a charm.  I added a few other TypeScript only things, like interfaces, and was loving that there was no transpile steps.

Anyways, I made my experiments available on GitHub at [kitsonk/deno-experiments](https://github.com/kitsonk/deno-experiments).

## Conclusions

I am going to keep a close eye on `deno`.  It could be really innovative, providing a runtime environment with first-tier TypeScript support.  There is a lot more I would want to do/understand, like how `deno` feeds TypeScript its configuration and what the plan is for local package management for things like `@types`.  Overall though, it is super interesting.
