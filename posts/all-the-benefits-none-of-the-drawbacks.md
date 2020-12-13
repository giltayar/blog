---
title: All the benefits of TypeScript, with none of the drawbacks
description: How you can get all the benefits of TypeScript, without needing to transpile to TypeScript
date: 2020-11-27
tags:
  - TypeScript
  - JavaScript
layout: layouts/post.njk
---
<!-- markdownlint-disable MD029 -->
TypeScript. For me, it's a love-hate relationship.

I started my developer life in static typing
(well, if we ignore Basic, which is dynamically typed): C, C++, Java. All, statically typed.
Then, somewhere around 2010, I had the good fortune to start working in Python.
And I saw the light! I was liberated! Suddenly all those Java and C++ design sessions, where
we fretted and worried and analyzed our class hierarchy to death. We split hairs about
this design or that design, instead of just, well, working.

And then you come to Python, and all those discussions just end. You just work. And it's not
that the design doesn't happen, but somehow, once you stop talking about class hierarchy,
you start dealing with the **data** flow, dealing with what data you have in the system,
and how it changes over time, and what the algorithms are that change it. And because
you're dealing with _data_, and not trying to abstract it away, then the design becomes
simpler. Much easier to reason and deal with. And much less hair splitting. And when
disagreement happens, it was on a much more practical level.

The disappearance of types was a good thing for my profressional career. And it continued
when I started working in JavaScript. One of the more interesting benefits of dynamic typing
is that I found myself writing more tests than I usually write. This was _because_ dynamic
typing doesn't give the safety net that static typing does. So you write more tests. Which
is a good thing.

Another good thing about dynamic typing: my code was "clean". In some ways, type information
is noise that you usually don't need to understand the code. And with dynamic typing,
that noise goes away, and your code is "clean": pure intent.

But I have to admit: While I loved having my code clean of types, I sometimes
missed code completion, and the ability to know what a function accepts or returns. No worries,
because if I missed something, my tests would tell me, but that happens _after_ coding, so
some amount of time was to deal with problems my tests found that I would have found out
during coding time if I had static typing.

But when I weighed the drawbacks of static typing (as I knew it), with the benefits of dynamic
typing, I knew that I should stick with dynamic typing. "Never go back!", said I. And I didn't like
TypeScript's rise in popularity. It was like finally I found a nice comfortable dynamic language,
and, well, just when I thought I was out, they pull me back in!

![just when I thought I was out, they pull me back in](/img/they-pull-me-back-in.gif)

TypeScript's gaining popularity was an affront to my belief that static typing is a scourge
on the world of development and that everybody should move to dynamic typing, and quickly! How
can developers choose to go back to static typing after they saw the horrors of static typing?
I really couldn't understand it. And I fought back. In any forum I could—I warned of the dangers
of static typing, and reminded people that if we weren't careful, Java-isms would come and
take us back to the dark ages of static typing.

But after a while I calmed down, and I started thinking a bit. And I noticed two things. The first
was that the war I was fighting was a religious war, a binary war, a holy war. And I do not like
religious wars. And here I was, fighting one, feverishly, passionately, arguing **against something
that I had no experience about**. And that was bad.

The second thing I noticed was that a lot of people that I admired were starting to use TypeScript,
and were liking it. So it couldn't be _all_ bad could it? Then again, a lot of those people
did not have _any_ experience with static typing, so maybe they were lured by the temptation of
static typings, but once there, they would start to convert it to Java?

Maybe. But maybe maybe I should do what I should have done from the beginning, instead of starting
a holy war? I should just...

Listen.

So I talked to a lot of them. And read blog posts. And realized something I never realized: \
TypeScript's type system is NOT Java's. TypeScript's spirit is much closer to JavaScript's than
to Java's. How so? TypeScript's type system is just a formal definition of something
that was never really defined: JavaScript's type system. It does not try to bend JavaScript to its
type system. Rather, it tries to bend _itself_ to JavaScript's. I believe it's goal is to be able
to define a type signature for every NPM package out there, however weird that NPM package is. And
it's succeeding marvelously!

And that is the main argument against my "you're going to turn JavaScript into Java" point. It's
not trying to emulate an OOP type system's like Java. Not at all. It's trying to formalize
JavaScript's informal way of implementing APIs and package interfaces. And since most
JavaScript packages interface are more concerned with _data_, and less with abstractions, then
so does TypeScript's. Yes, you can abuse TypeScript (and JavaScript!) to create Java-ish
abstractions, but the _spirit_ of TypeScript doesn't really want you to do that. Instead, it
encourage's a JavaScript like simplicity, that is more around functions and data than it is
around classes and abstractions.

But what about my second problem with TypeScript? Transpilation. I dislike transpilation. It...
complicates everything: debugging, stack traces, tooling, comprehension, scripting. Everything.
So while I was curious TypeScript, and could see the benefits, I didn't really want to deal
with _that_ problem.

But, still, one _should_ try, right? Right. So, because at work we work with Microservices,
I decided to write my next Microservice in TypeScript. Just to try it out. Which I did.

And it was wonderful and horrible at the same time. Wonderful, because types really do help the
coding phase. They don't really reduce bugs, because I have tests for that, and any typing
mistakes I make will be caught by those tests. But they _do_ reduce the coding time, becausse
no more typing mistakes (mostly), and they _do_ increase the readability of the code.

Horrible? Yes. TypeScript is the only language I know that changes its behavior based on a
config file. In actuality, TypeScript is not _one_ language. It's an infinity of languages,
each one _slightly_ different, and each one "generated" by a specific configuration file. And
to figure out the language variation you want, you start playing with that arcane config file.
Documentation? It's there, and much better than it was a year ago, but it's still arcane.

And it was horrible because of the transpilation. I had to start figuring out the tooling. And
how to work with Docker. And debugging. And stack traces. Not a nice environment. Workable,
but less than perfect.

## The dilemma

So on the one side, JavaScript: wonderful language (no, don't listen to those Java and Go snobs),
no build time, really great tooling. And dynamically typed, with all the pros and cons that come
with this. And on the other side, TypeScript: a variation on JavaScript, statically typed,
altough as dynamic as you want it to be. And transpiled, with all the cons that come with this
 (no, there are no pros to transpilation).

So what to choose for a new project? I've just joined [a new company](https://roundforest.com),
and I'd like to figure out what to use there! My heart went for JavaScript and it's buildless
simplicitiy, but I could not forget how _nice_ static typing works when coding, and how it helps
the readability of the code. But I couldn't help myself: I dislike transpiling.
I didn't want to lose the simplicity that comes with no build phase.

So can I get both? Can I get the buildless, no transpiling simplicity of JavaScript, along
with the ability to statically type functions and classes?

The answer it seems, is yes. And this blog post is all about that: how to enable this solution
in your code.

> Note: this solution works in Node.js. I haven't _yet_ tried it with frontend code, but I'm pretty
sure the results will be similar.

## The solution

The solution? Good ol' JSDocs. Ironically enough, JSDocs are an evolution of _JavaDocs_, which
were invented in Java land as a way to document Java code, and to make it available as structured
documentation.

But this time, we're not going to use them to document JavaScript _code_, but rather only
the _types_ in our code. And to do that, we're going to not simply write JSDoc comments, but
right them in a way that is **recognized by TypeScript**. Let's start.

By the way, you can find all the code in this blog post at the `jsdco-typing` repo
<https://github.com/giltayar/jsdoc-typing>, that includes an NPM package that has full typings
and can be used with any TypeScript (or JSDoc typing) code.

## Simple JSDocs

Let's start with a simple example of JavaScript code that is typed:

```js
function add(a, b) {
  return a + b
}

module.exports = {
  add,
}
```

(the above code uses CommonJS, but you can also use the new native ESM supprot in Node.js)

The above code is simple: it defines a function, `add`, that adds two numbers. Simple JavaScript,
no types. Now let's add some JSDocs.

```js
/**
 * @param {number} a
 * @param {number} b
 */
function add(a, b) {
  return a + b
}

module.exports = {
  add,
}
```

Same code, but we added a comment above the function `add` that defines the types of `a` and `b`.
Is this code regular JavaScript? Yes. We just added some comments. Those comments are JSDocs.
Let's look at tha parameters of the JSDoc:

1. It starts with `/**` to tell the world that this is a JSDoc.
2. It includes a line for each param in the format:

```js
/**
 * @param {<type>} <paramName>
 * /
```

3. The type MUST be inside `{...}` and anything inside it defines what type the parameter is.
4. If TypeScript is used to interpret the JSDoc, then the type inside it MUST be a TypeScript type,
   or else TypeScript will ignore the type and define the parameter to be `any`, which meanas
   that it is typeless.

This is _not_ going to be a tutorial on JSDoc. If you want more information, there'll be a link
to the JSDoc documentation in the TypeScript site. Because, yes, TypeScript supports JSDoc. How?
Well, once this function is JSDoc-ed, then for all sense and purpose, TypeScript will treat it as
typed. Let's add another file that _uses_ the `add` function, and see how it supports it.

```js
const {add} = require('./utils')

console.log(add(4, 5)) // => 9
```

The above code imports the `add` function and calls it. Nothing special here. But! If you're using
Visual Studio Code (and probably other IDEs as well), and hover above the `add` function, you'll
get this:

![hover showing JSDoc typing information](/img/jsdoc-typing-hover.png)

So Visual Studio Code, because it supports TypeScript out of the box, shows the JSDoc information.
But moreover! Because TypeScript is reading the JSDoc, it's also reading the JavaScript code
as if it was TypeScript, figuring out by itself that the return value is also a number, and
showing that information as well! Without us have to add anything.

## Adding type checking to the hover information

This is already great! We get nice autocompletion and hover information for our functions.
But it's not typechecking. It doesn't fail if the types are incorrect. It doesn't even show
up in Visual Studio Code as an error. Let's try it and see:

```js
const {add} = require('./utils')

console.log(add('sdfsdf', 5))

console.log(add(4, 5)) // => 9
```

If you try it in Visual Studio code, you won't get a squiggly red line on the second line where
we try a string argument. But there is a way to do that, a shortcut to the correct way. I'll show
you how to do it, but it's not going to be the final solution. We'll just add a `//@ts-check`
to the start of the code:

```js
//@ts-check
const {add} = require('./utils')

console.log(add('sdfsdf', 5))

console.log(add(4, 5)) // => 9
```

Let's now see it in Visual Studio Code:

![squiggly-red-line-on-bad-call](/img/jsdoc-squiggly-red-line-with-ts-check.png)

Wow! Types are working! I can now see my errors. But that isn't enough. We want typechecking!

## Adding typechecking to our tests

To add real typechecking, one that will fail the build, and not just show up in Visual Studio Code,
we need TypeScript in our package. Let's do that:

```sh
npm install --save-dev typescript
```

Great, we've installed TypeScript. Let's now run typechecking on our files:

```sh
$ tsc --noEmit --allowJS src/*.js

src/jsdoc-typing.js:4:17 - error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.

4 console.log(add('sdfsdf', 5))
                  ~~~~~~~~

Found 1 error.
```

Yup, we found the error using TypeScript. We've just completed step 1 of our JSDoc typing journey.

A few comments on the options we gave to TypeScript:

* `--noEmit`: this tells TypeScript not to transpile to "JavaScript". Obviously, we don't need
  that, because our files are _already_ JavaScript.
* `--allowJs`: this tells TypeScript that it's OK to check JavaScript files. If we don't add this,
  it will ignore the JS files, even if we specified them in the command line.

## Configuring TypeScript options correctly

## Exporting `.d.ts` files

## Using other libraries

* Libraries with TS support
* Libraries that need `@types/...`
* Libraries that need a `declare module...`

## Advanced Typescript

## Using a bit of TypeScript typings

## Summary
