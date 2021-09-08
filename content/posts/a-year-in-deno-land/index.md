---
title: "A Year in Deno Land"
description: Reflections on working full-time on Deno for a year.
date: 2021-09-05T12:00:00+10:00
images:
- /posts/deno-skypack-and-types/hero.png
comments: true
summary: I started working full time on Deno on the 21st of September 2020 and wanted to share some of my thoughts.
tags:
- deno
- work
- work culture
author:
  name: Kitson Kelly
  image: /images/kpk_512.jpeg
menu:
  sidebar:
    name: A Year in Deno Land
    identifier: a-year-in-deno-land
    weight: 99
---

I started working full time on Deno on the 21st of September 2020 and wanted to
share some of my thoughts on doing so.

## My privilege

It was a bit of a leap of faith to go work on Deno full time. I was very
comfortable working at ThoughtWorks in Australia. The _only_ reason to make the
jump, really, was to see what could happen, working on a _passion project_ full
time. Lots of people will throw out there "follow your dreams", but in reality,
for a lot of people that is total BS.

Why? Because it is totally a position of privilege that I was able to do it.
Early in my career I wasn't conscious of my privilege, but being a white-male
from the middle class of America is a whole lot of privilege, and made a lot
possible for me, effectively serving me on a silver platter. I was just lucky
enough to be able to successfully ride the bow-wave.

The birth of my son in August 2018, and being able to take 7 weeks off work to
look after him, was another bit of privilege that allowed me to work on my
passion project of Deno that I had discovered as soon as Ry released the
prototype in May of that year.

Sitting in Melbourne, in the midst of another hard lock-down due to COVID-19,
working on Deno always makes me reflect on my privilege of how lucky I am to
have a wonderful family, being financially secure, doing work I enjoy in a very
flexible way that allows me to help look after our 3yo. I must be in the 1% of
1% of people that are that lucky.

## Diversity

My one frustration/regret/challenge with Deno and the commercial company Deno
Land Inc. is that in the company we have a significant lack of diversity in
certain ways. We have decent multi-cultural diversity. There is only one
American who lives in America. The other American, me, hasn't lived in the US
since 2006. We have Dutch, Polish, Japanese, Indian, Canadian and German folks.
We have a spectrum of ages, with I believe me being the oldest at 48 with the
youngest not being old enough to drink if they were in the US. We have at least
two of us that identify as part of the LGBTQ+ community.

All that being said, we are all males and of the ethnic majority of the country
we reside in. Deno - though - has been a self organizing community. Ry and Bert
had a previous work relationship and convinced Ben to join us, outside of that,
the remaining 7 of us (soon to be 8) have come from the Deno community itself.
It is great that people with a passion are joining a company to work on their
passion, but communities are communities, and unconscious bias easily works its
way in.

The good thing is that it is something we have talked about as a company, and
also how to help diversity in the community we are caretakers of. There are no
clear answers. A lack of gender diversity and ethnic minorities in the community
that is the source of what is still a relatively small commercial company makes
it hard to have diversity across lots of dimensions. I have voiced it internally
though, having more than 10 people and only one gender is a problem. We have to
consider something proactive to break the _echo chamber_ we have fallen into.

## Working remotely

I was lucky that I had worked previously for a 100% remote company, SitePen, for
two years. While a lot of us find ourselves working remotely for the first time
due to COVID-19, I had the experience. There are great things about it, and
there are some really yucky things about it.

The great thing is that it is a lot easier to get work-life balance right. I
have never had a big delta in the motivation/challenge between local and remote
work. Local does seem to get you dragged into endless meetings, though that is
more likely a challenge of scale for larger organizations than really a
local/remote thing.

The biggest drawback for me is the isolation. My nearest co-worker is in Tokyo.
I am isolated from everyone else's time zone as well as spatially. When I worked
for SitePen, we ended up with a few of us in London, which did mean it was easy
once a week to meet up, let alone having a handful of people at least in the
same time zone. Coupled with extended lock-downs in Melbourne, life is just
super isolated right now. It is hard to divest how much of it is the isolation
of lock-downs versus isolation of remote working at the moment.

## The company

I am really glad Ry and Bert decided to take on some investment and form a
commercial company. When the company was announced, there was a lot of noise
about the commercial nature of things. There is a big dichotomy in open source
software. Large enterprises have embraced it, with most wanting their cake and
eating it too, secure well maintained software that doesn't cost them anything
to use. The selfishness of large enterprises means they will only spend money
when "forced" to, and the open source community doesn't do a good job of forcing
enterprises to pay their fair share.

At the end of the day, unless you are paying software engineers to work on open
source software, you are left with taking advantage of people's privilege and
kindness to contribute to the project. That does not result in a sustainable
model. People need to get paid for their time.

I generally see four traditional models in this space:

- The _rounding-error_ model, where a large commercial enterprise can pay people
  a day job that is either full-time or part-time to maintain an open source
  project. Usually the commercial organization then has an outsize influence on
  the direction of the open source project, or the open-source project is used
  as a calling card for how "generous" the company is but under-invests in the
  open source project.
- The _pan-handling_ model, where an open source project spends a good amount of
  its time trying to convince individuals and commercial organizations to
  subsidize the development of an open source project, hopefully without too
  many strings attached.
- The _freemium_ model, where the commercial organization leverages an
  open-source project to build a commercial product, where the "good" features
  go into the commercial project. This often compromises the open source project
  and features greatly.
- The _consulting_ model, where an organization tries to use the profits from
  consulting around the open source project as an investment vehicle in the open
  source project. This I thought was the most sustainable model and thought that
  is where Ry and Bert might end up, but they didn't...

The cloud/SAAS/PAAS/serverless etc. have opened up another venue that wasn't
really there before, and Ry and Bert decided to try to build a business on that,
where the open source product is the open source product, un-compromised by
_freemium_ features, while along side it is a complimentary commercial project
that provides a pay-by-usage revenue stream. Deno Deploy shares a lot of code
with the Deno CLI, but it isn't the Deno CLI, and it won't ever be the Deno CLI.
It is a serverless web platform at the edge. (gotta love buzz words) Deploy is
totally separate from the CLI, though they compliment each other (and trust me,
there are some features in Deno CLI that would have never been there if Deploy
needed to leverage them).

Trying to build a sustainable open-source project is hard. Deno Land might not
have it right, but we gotta try to find ways to do so, because people need to be
compensated for their effort, if people want secure well maintained software.

## What I have learned

I have done more coding the past year than I have ever done in my life. Every
other role I have done, coding was only ever a minority part of my role. Most of
my career was consulting and then senior leadership, and most of my hard core
_coding_ was a side project or even just a hobby.

Having most of my work day be coding is strange for me. I think my experiences
in other roles _higher up the foodchain_ have made me a better software engineer
though, understanding the wider context of what producing code impacts.

Like a lot of my previous jobs, some of the best parts are the people you get to
work with.

I like working with Ry. I often find it frustrating, as he comes up with some
crazy bat-shit ideas, or throws out ideas I am diametrically opposed to, but
time and time again, those often get iterated upon and molded until we all have
something better than what we started with. I really appreciate his ability to
self-iterate. I like Bert. I like his style, I like his way of thinking. He
clearly has domains of experience that I don't have, where I learn a lot from,
and I almost always feel more energized and excited after talking to him 1-2-1.

Ben is awesome, he knows a hell of a lot more than I do about programming, and
he never talks down to me when I put forward some infantile code that he
reviews. Luca has great energy, and is very bright. We don't always agree, but
again, his ability to iterate on a problem usually ends up in a better place.
Bartek is awesome, while appears to be somewhat contrarian at times, he also
loves to collaborate and iterate on a problem. Yoshia-san is a quiet giant, he
has great contributions. We tend to work on different things, but everything I
see him do is quality and well considered. Like Yoshia, Will and I don't cross
work paths much, but we tend to share similar viewpoints on work culture which I
really appreciate. Satya is relatively new and I have had some good experiences
working on some things together and look forward to doing more with him. David
and I have _known_ of each other for a few years now. Hearing him speak at the
first TSConf in Seattle made me pay attention to him and now that he is working
on Deno with us, it is awesome. We are just starting our work relationship, but
he comes in the door with a lot of respect from me. We have another joining
soon, which will be another set of adventures I am sure!

I have learned a lot from my co-workers in the past year (yet to mention the
wider Deno community as a whole), and for me that is one of the biggest gauges
of "success" in my mind.

## The _product_

Most of the past year has been a focus on developer experience, and trying to
make the developer experience with the CLI even better, and touching on trying
to solve problems with Deploy in the context of _dog fooding_ Deploy and helping
work out the kinks.

It has been an interesting focus, and I have enjoyed it. We have been able to
really make progress in making the development experience in Deno a lot better,
building a language server right into the Deno binary itself. There is still
more to do, and I guess that is one of the biggest lessons I learned, my desire
to make things better outstrips my ability to make things better. People finding
issues or gaps, or being confused or frustrated by a feature really sort of gets
to you on a fundamental level.

The "nice" thing about my focus on Deploy, is that I am the one trying to
identify the gaps for others on the team to try to address. I am the source of
their angst. I have done a lot of fun stuff with Deploy. I am really becoming a
fan of the power of Web Assembly.

## JavaScript, TypeScript and Rust

My language journey has been Turbo Pascal, Delphi, PHP, JavaScript, TypeScript
and now Rust. While Turbo Pascal and Delphi were memory allocated, they weren't
the C-esque type of memory allocation and so while I understood it, having spent
the last 20 years in garbage collected languages, Rust was a steep learning
curve for me. I suspect even those coming from C, Rust is a steep learning
curve, because of its memory management model shift, but now that I am on the
other side of it, and at least I feel "productive" with Rust, I think it allows
me to evaluate it.

I have been doing TypeScript since 2014, heavily since 2015, and JavaScript
heavily since 2008. I would say I "love" all three languages, and all three
languages have their sweet spot.

Rust only reinforces my love of static typing though. Even being proficient at a
language, static types not only save you from _run-it-and-pray_ behavior, they
are the easiest way to discover and consume APIs.
[rust-analyzer](https://rust-analyzer.github.io/) has made me so productive in
Rust, that I don't think I ever want to use another language without an
intelligent IDE. It was one of the compelling reasons we invested in the Deno
language server. If you take static typing out of a language/system, then you
are really flying blind. Well not only are you flying blind, everyone who
consumes your code is flying blind too. They have to read your code to figure
out what you intended to do.

Yeah, I write JavaScript sometimes still, but I always write it with JSDoc type
annotations. I just can't stand that in my IDE that it is trying its best to
guess at what I am trying to do. I like the safety of type safety. TypeScript
has improved since 2014 though to such a great extent I find myself writing
TypeScript by default, because it is still easier to do the type annotations
inline when needed, but so much less needed than before.

What is a lot easier in TypeScript/JavaScript than Rust is scaffolding out a new
API. Because of the strictness of Rust, it is really hard to "stub" stuff out,
whereas you can often still run your code in TypeScript/JavaScript without
crossing every T and dotting every I. This makes prototyping in
TypeScript/JavaScript a lot easier. If you know what you are building, Rust is
great. I really love Rust enums and iterators, plus also the patterns of traits
like `From` and `Display` are superpowers that make you code super clean and
super consumable. Every time I drop back into TypeScript/JavaScript, I miss the
power of `match` and `.into()` and `if let` and returns from blocks to do
variable assignment.

## Final thoughts

I don't think anyone knows what the future holds. I think most people would say
their 2019 selves would barely recognize their 2021 selves and that certainly
holds true for me, but for me in a work dimension as well. I am so lucky to be
privileged though to be able to work on something that I really am emotionally
invested in. In other roles, I have grown to love what we built, but Deno is
something I loved and got the opportunity to build.

Where will I and Deno be in a year, or two years, or three? Who knows. We are
trying to build a platform where scripting in TypeScript and JavaScript is fun
and productive. Hopefully we will succeed and make a difference. If we do,
awesome, if we don't, I think I will have few if any regrets.
