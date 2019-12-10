---
title: TSConf 2018
date: 2018-03-13 14:28:59
tags:
- conference
- technology
- TypeScript
comments: true
---

I got the opportunity to attend [TSConf 2018](https://tsconf.io). It was the worlds first focused conference on TypeScript and the lead organiser was my former employer [SitePen](https://sitepen.com). We had been batting around a focused conference for an extended period of time, and when Dylan and I visit the TypeScript team about 9 months ago, we figured it was time to pull it together. I wasn't able to stay around to help make it a reality, but I was glad I could attend to see the outcome.

<!-- more -->

![Entrance to TSConf at Axis in Seattle](/images/TSConf_Sign.jpg)

## Keynote - Anders Hejlsberg

I suspect you can't really expect to have any first _official_ TypeScript conference without having Anders Hejlsberg ([@ahejlsberg](https://twitter.com/ahejlsberg)) as your keynote speaker. After having gotten my [fanboy experience](./Fanboy) out of the way when I visited Redmond last year, I was looking forward to Anders' talk and I wasn't disappointed. It was clear he was enjoying talking to a group of people who actually lived and breathed TypeScript. I suspect most of the time he has to go back to the basics of what is TypeScript instead of talking about the interesting aspects.

Anders started talking about the history of TypeScript, in particular how Microsoft saw that JavaScript was going to become important to the enterprise but that it wasn't going to be able to scale to enterprise applications. He indicated that even in the earliest days, in 2011 when it was code named _Strada_, that it was always envisioned that it would be a super set of JavaScript. One thing that was off the table in the early days was generics in TypeScript, as it was considered too hard of a problem to solve. Of course those familiar with the language now know that generics are a critical and powerful feature of TypeScript.

He gave insight into one areas of TypeScript that most users never even notice, which is good (and sometimes a little bad) in that there are actually two namespaces. There is the _type_ namespace and the _member_ namespace. The _type_ namespace is what doesn't exist at runtime in JavaScript and is purely the domain of TypeScript, while the _member_ namespace is where the variables and other identifiers which exist at runtime are. TypeScript manages both of those namespaces at the same time, and very seldom does the developer need to be aware they exist.

Also from the early design notes for TypeScript, Anders said that it was a big change for Microsoft to explicitly state that _winning in this space is not about protecting IP_. TypeScript was designed from the start to be an open language. Something that has continued to this day. Anders highlighted too that a side project, [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/) has the most reviews of any project on GitHub, has over 42k commits and almost 8k contributors. This project, along with TypeScript are providing type intelligence, not only to projects written in TypeScript but also just plain old JavaScript.

One of the interesting things that Anders talked about is that the TypeScript compiler, as it stands now, is actually the 3<sup>rd</sup> rewrite of the compiler. The first version really was an experiment, the second version struggled from a performance perspective and the current 3rd generation was something he felt they could maintain and live with in the long term. It is a heavily functional based compiler, which is something that surprised him. He indicated a couple times that sort of living in the Object Oriented world his whole career, he has found that functional programming has a lot to offer. It almost sounded like he was a convert. I know personally, having been raised with OOP, I too have found that sometimes it causes a huge amount of indirection and doesn't really deliver as well on the promises it makes. That in a lot of ways, some problems are significantly easier to solve when you start looking at things where your logic and data are cleanly separated. Hearing Anders say some of the things that I had thought for a long time was really interesting.

![Slide equating recursive types to an infinite stack of turtles supporting the Earth](/images/Turtles.jpg)

Probably one of the most fun and interesting moments of Anders is when he talked about recursive types. He had indicated that the team had struggled with it and felt that maybe they were missing something, that there has to be a solution to it. They went off to Microsoft Research and asked "ok, how do solve this problem?" the response "oh, yeah, that problem... well everyone cheats on that problem." So in TypeScript when using a recursive type, TypeScript only goes to about five _turtles_ deep before it cuts off. Anders then said that in practice there wasn't a single issue raised that that had caused a problem, until recently, and someone had some crazy JSON type that was about 10 _turtles_ deep where it changed and was wondering why TypeScript wasn't catching errors in it.

![A slide showing a nun with a ruler and talking about strict mode in TypeScript](/images/Strict_TypeScript.jpg)

Anders touched on a part of TypeScript I have always been a fan of, the strictness. Over the last several versions of TypeScript greater and greater levels of type checking and adherence have been introduced. The continue to do analysis of how the impacts of the strictness actually impact code and are finding that the a good third of the time, increased strictness is actually identifying issues with the code, while a third of the time, it is identifying code that _could_ have been a problem in the future.

Anders finished with two things in the future with TypeScript. With conditional types coming in TypeScript 2.8 (which is going to a release candidate this week) and then project references coming at some point in the future.

Conditional types essentially allow description of complex types where resolution of a generic type influences the final type at the usage site. This solves a whole class of problems, far more than I had thought when I first heard about it. For example, you can actually properly model the `typeof` operator with string literals in TypeScript:

```ts
type TypeName<T> = T extends string
  ? "string"
  : T extends number
    ? "number"
    : T extends boolean
      ? "boolean"
      : T extends undefined
        ? "undefined"
        : T extends Function ? "function" : "object";

function typeOf<T>(x: T) {
  return typeof x as TypeName<T>;
}
```

The thing that was most exciting to me about this is that it solves a whole set of challenges that I know I have struggled with but also loads of wider community have struggled with. In a lot of ways, this really gets as close as I think we need for _programmatic types_, where we would essentially have direct access to the type system from within the code. Here are some of the new pre-defined types that leverage the new conditional types:

```ts
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;
type NonNullable<T> = T extends null | undefined ? never : T;

type ReturnType<T extends (...args: any[]) => any> = T extends (
  ...args: any[]
) => infer R
  ? R
  : any;

type InstanceType<T extends new (...args: any[]) => any> = T extends new (
  ...args: any[]
) => infer R
  ? R
  : any;
```

You can even model things like _flatten_ (or _smoosh_ should I say?):

```ts
type Flatten<T> = T extends Array<infer U> ? U : T;

type T1 = Flatten<string>; // string
type T2 = Flatten<string[]>; // string
```

Ander's finished up his talk about some work that Ryan Cavanaugh ([@SeaRyanC](https://twitter.com/SeaRyanC)) has been leading which is to try to find a way to deal with _big_ projects and allow structuring of complex projects, where there might be different configurations and more importantly, don't require the compiler to re-build the entire project each time. This didn't make 2.8 and Anders didn't want to commit to a release that it will be in, but hopes in the relative near future.

While there was already the ability to extend TypeScript configuration files, this feature would introduce project references. Each `tsconfig.json` would be able to express other TypeScript projects that it exists with, essentially making mono-repos of large TypeScript projects a reality. The biggest impact was in the performance. In all good things there has been some _dog-fooding_ by the team by breaking up the TypeScript compiler into separate projects. Anders live demonstrated how not only does it make incremental change quicker (super quick) it actually improves the whole compile process because TypeScript is given information about when certain areas of code are going to be required in the compile time and that those _packages_ of TypeScript. I really can see a lot of application of this in the future. It was one of the things we struggled with in Dojo 2, how to realistic "build from source" for projects.

Overall the talk was engaging and exciting and as I was going to find out from the rest of the day, I learned a lot from the talk.

## Building StencilJS with TypeScript – Josh Thomas

Josh Thomas ([@jthoms1](https://twitter.com/jthoms1)), one of the core developers at Ionic, talked to us about StencilJS. I had heard of StencilJS and knew at a high level what it was attempting to do. In particular Matt Gadd came back from a conference totally impressed with it, and commented that if we were starting Dojo 2 again, we might have just embraced StencilJS. From a design ethos it was very synergistic with what we set out to achieve with the widgeting system for Dojo 2.

Ionic has, in my opinion, picked the right moment to fully embrace Web Components, and StencilJS seems like a solid bit of technology. They also have addressed some of the things I _dislike_ about Angular 2+ components, in that they have heavy use of decorators that perform a lot of _magic_ this it dangerous in that it doesn't really integrate well with the type system. They have created the necessary tooling to inject intelligence and safety into decorators at design and build time.

They also, like we found with Dojo 2, appreciate the _reactive_ model for widgets. Josh indicated that a lot of the life-cycle methods are directly adapted from React.

The beauty of course is that being Web Components, StencilJS components can be used with any framework you so choose, or even realistically with _VanillaJS_.

Overall, Josh's talk reinforced that I should take a closer look at StencilJS and certainly if I hear of people using it, it would not be one of those things that would raise alarm bells in my head, like some other technology which just worries me when I hear people using it.

## Escape the Office: Designing Interfaces for Other Developers – Sarah Highley

I of course have a soft spot for Sarah Highley ([@codingchaos](https://twitter.com/codingchaos)), having interviewed her to work at SitePen and been quite proud that we had her join the team. Now that I have left SitePen, it is really good to see that she is continuing to challenge herself and do more public speaking.

![Slide showing cartoons of the Caveman Coder, The Intern Horde and the Inventor](/images/Milestone_Monsters.jpg)

Sarah's talk focused on two main areas, first that TypeScript really allows us to design APIs not only for ourselves, but other developers. She adapted some of the _monsters_ from the [MileStone Mayhem](https://milestonemayhem.com/) game we developed at SitePen. I was also hugely proud that one of the monsters I gave a [talk on](https://www.sitepen.com/blog/2017/10/17/caveman-coders-and-how-to-defend-against-them/) and we added to the game, made an appearance. Sarah then applied these three types of programmers to how they would consume an API around creating accessible components, and how we create the interfaces in TypeScript directly increase usability and help keep these different types of programmers from making mistakes.

Sarah is hugely passionate about accessibility, and it was demonstrated again. I remarked to her and others afterwards, that while we need to continue to be proactive about diversity at conference, I am feeling like we also need to have at least one talk that reminds people about the need of accessibility. I don't think most developers _want_ to make applications that aren't accessible to certain people, but that is what we often accidentally do.

## End-to-End Type Safety with Swagger and TypeScript – Mohsen Azimi

![A slide showing the server and the client with a crying emoji in the middle, titled The Lost Types](/images/Lost_Types.jpg)

Mohsen ([@mohsen\_\_\_\_](https://twitter.com/mohsen____)) is from Lyft and was talking about the challenges of rapidly changing server side APIs, but then introducing breaking changes in the client code. One of the things Lyft did rather publicly was migrate their front end code to TypeScript, mostly because they got tired of people pointing out a lot of the mistakes they were making could be caught via TypeScript.

They still suffered from regressions though, where developers would make breaking API changes on the server side and not realise the impact they were having on the client side, because all the types get lost in the communication between the back end and the front end. What if the two could talk together, Mohsen pondered. Of course TypeScript support strong typing, and there are tools like [swagger-codegen](https://github.com/swagger-api/swagger-codegen/) which can generate TypeScript typings based on an API definition.

The challenge becomes how best to describe the APIs on the server side to ensure that the whole API is maintained. Well traditionally this could be accomplished by sever annotations that helped describe the full API. That of course can be a challenging, since those annotations are not always tied to the actual code.

What Mohsen proposed was to author the contract of your APIs as [Protocol Buffers](https://developers.google.com/protocol-buffers/) to describe your API in a language agnostic way, which can do both code gen on the server side language, but also be ingested by Swagger to define the contract for the API. This then can be used to keep your client side TypeScript code in sync with the APIs and help identify quite quickly if an API change is going to cause an impact on the client.

I have to say overall, that exploiting TypeScript to ensure that our communication with between the client and the server is an excellent way to help bring more sanity to the front-end and an ideal use of the technologies. It is great to see others coming to the same conclusions and helping move forward the community.

## TypeScript at Google – Rado Kirov and Bowen Ni

One of my fears about the conference was that it might be dominated by Angular, but I was really glad to find a huge diversity in the topics and these even came from those who were speaking from the Angular team. While Rado ([@radokirov](https://twitter.com/radokirov)) and Bowen ([bowenni](https://github.com/bowenni)) are both formally part of the Angular team, their day to day jobs are to help manage TypeScript usage at Google.

![A slide showing the current usage of TypeScript at Google](/images/Google_Usage_TypeScript.jpg)

Rado explained how that the way Google develop code has some advantages and disadvantages and that their job of maitaining the usage of the language in the organisation has some unique challenges. Google has a gigantic mono repo and the ~70k TypeScript files can only be used transpiled with a single version of TypeScript. That means that have to utilise particular tooling to achieve this. They have migrated the code base from every release of TypeScript from 1.4 to 2.6 and it takes about 3-4 engineering weeks of work to upgrade.

They use some automated tooling to perform the upgrades and the really interesting strategy is that when an upgrade causes a type error, what they do is cast the variable/assignment to a specific type alias of `any`. For example when upgrading to 2.6, they would do something like this:

```ts
type AnyDuringTS26Migration = any; // remove after 2.6 lands

someFunction(expr); // error with stricter checks in 2.6
// get change to...
someFunction(expr as AnyDuringTS26Migration);
```

They then use tooling to track that the teams that own the code actually get around to fixing whatever is required to make the code work with the updated version.

They also discussed what makes a good style guide, especially with TypeScript. They indicated that a good style guide should do the following:

* Local knowledge is important
* Global rules should be an exception
* A rule should be in the style guide if it removes a known problem
  * For example, discourage use of `any` or disallow namespaces
* Provides consistency across irrelevant variations
  * For example, `T[]` or `Array<T>`
  * For example, `expr as T` or `<T>expr` or `<T> expr`
* Helps maintain code long-term

The other think that they talked about is how they use specially built tooling that integrates with the TypeScript language service to make some of the refactorings of code easier, especially when dealing with things at scale. The application is called [tsetse](https://github.com/alexeagle/tsetse) which they use to fix or refactor known patterns of code and generate changes which are then submitted to the code owners for acceptance.

![Graph showing the utility of tsetse, users say that 21% of errors detected by tsetse would have been found in production](/images/tsetse.jpg)

The team did a survey of the users, to understand if the higher-order static analysis was actually adding value or just noise. While there was likely 24% of errors that wouldn't have gotten past code review, the majority of issues would not have been identified until testing or even worse, would have been found by users in production. This type of tooling highlights that the TypeScript compiler is designed to be open. While the language services APIs are not as well documented as we would like, they are still powerful tools we can leverage. It also highlights that static code analysis and finding bugs early is a really important pattern.

## Making Weird Physical Games with TypeScript – Mike Lazer-Walker

Making physical games with TypeScript might not be the fist thing that comes to peoples minds, but it is with Mike ([@lazarwalker](https://twitter.com/lazerwalker)). While a lot of his projects span the physical and logical worlds, he had found himself writing narrative stories and made an [open tool](https://github.com/lazerwalker/storyboard), based on TypeScript for writing those stories. He can then plumb those narratives into the physical displays he creates.

For example, one of his big installations was a working game using vintage 1920s telephone switching hardware:

<blockquote class="twitter-tweet tw-align-center" data-lang="en" data-theme="dark"><p lang="en" dir="ltr">I made a game played on an actual vintage 1920s telephone switchboard! <a href="https://t.co/tDB20dsu6T">https://t.co/tDB20dsu6T</a> <a href="https://t.co/ICrXegsYQq">pic.twitter.com/ICrXegsYQq</a></p>&mdash; mike (@lazerwalker) <a href="https://twitter.com/lazerwalker/status/890293234534031360?ref_src=twsrc%5Etfw">July 26, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

It was interesting hearing his stories of physical game installations as well as how TypeScript was helping him create innovative ideas as well as inspire others to create interesting art.

## Static Analysis and Source Code Manipulation in TypeScript – David Sherret

I remember when I was reading all the talks briefs submitted, I thought this was going to be the most interesting one technically, but I wasn't sure how well it would play in front of an audience. The interesting thing is that everyone in the room really seemed to like it and David ([@davidsherret](https://twitter.com/davidsherret)) delivered an excellent presentation. He is a quiet and reserved Canadian, but packed a fairly powerful punch.

The first bomb he dropped on us was an [AST viewer for TypeScript](http://ts-ast-viewer.com). While we have some excellent JavaScript AST viewers, and even [AST Explorer](https://astexplorer.net/) supports TypeScript now, having a focused dedicated AST viewer was nice to see.

He went on to provide was a primer on walking TypeScript AST, using his [handy API](https://github.com/dsherret/ts-simple-ast) which makes interfacing with the TypeScript AST a little bit easier, for example using the library to help enforce higher order design patterns:

```ts
// for each public class method in our source file...
for (const method of classDec.getInstanceMethods()) {
  if (method.getReturnType().isNullable()
      && classDec.getInstanceMethod(method.getName() + 'OrThrow') === undefined)
    {
      console.warn(`Expected method ${classDec.getName()}.${method.getName()} ` +
        `to have a corresponding OrThrow method.`);
    }
}
```

Again, like with the talk by Rado and Bowen, extending the TypeScript compiler to provide additional higher order static analysis seems to be a common use case for larger teams. I think it is a really interesting space and something I think I will consider in the future when faced with these sorts of problems.

## Webpack and Ethereum Smart Contract with TypeScript – Devon Zuegel

It was only after Devon ([@devonzuegel](https://twitter.com/devonzuegel)) gave her talk, I found out that it was her first conference talk. I would have never have known. I happened to be sitting next to Anders and I know he too was really impressed with her talk.

Devon first spoke about how [webpack](https://webpack.js.org/), while being one of the most powerful tools in the web application space, is a beast to master, and how applying TypeScript to the configuration of webpack has allowed her, and her team to start to master it. I think the whole room was nodding with her about this.

This led into talking about Ethereum smart contracts and how the company she works for, [Bloom](https://bloom.co/), has realised the power of static typing in relationship to smart contracts. She mentioned the case of nearly $6 million that has been lost forever in Ethereum because there was not static type checking on an Ethereum address that was addressed as a string instead of a hex address. She hypothesised that if static type checking had been used, it would have at least flagged that the address was not typed appropriately and could have saved someone losing millions of dollars. She continued to describe how they have been finding applicability of strong typing in every aspect at Bloom, and how dealing when dealing with smart contracts, it is essentially unacceptable to have it left up to the vagaries of a weakly typed language like JavaScript.

## Q&A with the TypeScript Team

Nick ([@nicknisi](https://twitter.com/nicknisi)), who had been hosting the event and Torrey ([@iTorrey](https://twitter.com/iTorrey)) who hosts [TalkScript](https://twitter.com/talkscript), a TypeScript podcast, asked questions of a panel from the TypeScript team. The panel included Mohamed ([@fdsmars](https://twitter.com/fdsmars)), Ryan ([@SeaRyanC](https://twitter.com/SeaRyanC)), Anders ([@ahejlsberg](https://twitter.com/ahejlsberg)) and Daniel ([@drosenwasser](https://twitter.com/drosenwasser)).

![A picture of Mohamed, Ryan, Anders and Daniel](/images/TypeScript_Team.jpg)

There was a fair amount of insight and ideas floating around on the panel. Some interesting things I picked up during the QA where:

* We are all frustrated with ES Modules. While there is little issue with the syntax, the implementation has been nothing but problems. I had a few offline conversations afterwards too, and the feeling was there was a lot of blame to go around. The quote though from Anders was _we can flag our way out of it_ indicating that one way or another, we can set flags which will adjust the emits for our targets. Realistically I think we are in a world where bundling using tools like webpack are here to stay, because even if we do _fix_ ES Modules, there are other things like HTML, CSS and static assets that we will need to manage.
* The team regrets implementing some things, like namespaces/modules, ahead of the ECMAScript/TC39 standards. They have learned from that and when it comes to things that are domains of the syntax of the language, they will only implement when it reaches [Stage 3](https://tc39.github.io/process-document/).
* Anders talked a bit more on how he has learned a lot from functional programming versus OOP and how it has made him think about approaches to problems differently.
* In talking about the perfect language, Anders said _show me the perfect language and I will show you one with no users_ and how TypeScript will continue to evolve and pick up some crud along the way, and that in a way is the sign of a healthy language, versus this idealistic one that doesn't actually solve real world problems.
* Torrey asked what each team member's spirit emoji was. Daniel of course went for 💩. Colour me surprised.
* There was further disclosure by Daniel that he had originally worked on the `--pretty` flag for TypeScript and so the team started referring to him as _Pretty Boy_.

There was a good combination of humour and insight into the language, and I think everyone took away a little bit more and at the very least understood a bit more about the some of those behind the language.

## Final Thoughts

While I have accidentally become a TypeScript fan-boy, and am still shocked that I have become a huge fan of a Microsoft product. At its core though, TypeScript was the start of something bigger at Microsoft. It was a situation exemplified by Satya, where Microsoft is continuing to transform into an engineering led organisation. I know talking to many on the TypeScript team over the past couple of years, the team fought hard and won a lot of battles to make TypeScript a truly open language. A model that is continuing to be adopted by others, like Erich Gamma and the [Visual Studio Code team](https://github.com/Microsoft/vscode) (which is an [Electron](https://electronjs.org/) application authored in TypeScript) and the [Chakra Core](https://github.com/Microsoft/ChakraCore) team.

Being a part of the first focused TypeScript conference therefore was a pleasure and to meet a wider community of people, as well as touch bases with many that I know, to see how healthy the community is. I am already looking forward to the next one.
