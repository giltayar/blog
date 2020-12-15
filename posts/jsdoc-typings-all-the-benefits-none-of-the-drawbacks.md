---
title: "JSDoc typings: all the benefits of TypeScript, with none of the drawbacks"
description: How you can use JSDoc typings to get all the benefits of TypeScript, without needing to transpile to TypeScript
date: 2020-11-27
tags:
  - TypeScript
  - JavaScript
layout: layouts/post.njk
---
<!-- markdownlint-disable MD029 -->
TypeScript. For me, it's a love-hate relationship.

I started my developer life in statically typed languages
(well, if we ignore Basic, which is dynamically typed): C, C++, and Java.
Then somewhere around 2010, I had the good fortune to start working in Python.
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

## Implicit and explicit typing

Let's define a variable that accepts the value of the call to `add`:

```js
const addition = add(7, 8)
```

What will be the type of this variable? TypeScript does it's thing, and the type will
be `addition`, because TypeScript knows how to automatically infer the type of the variable.
To prove that it does, let's hover above the new variable:

![implicit typing of variable](img/jsdoc-implicit-typing-of-variable.png)

This has nothing to do with JSDoc: it's TypeScript implicitly typing the variable to the correct
type. But what if we wanted the type to be explicit? What if we wanted the equivalent of this
TypeScript code?

```ts
const addition: number = add(7, 8)
```

We still can. The equivalent JSDoc is this:

```js
/**@type {number} */
const addition = add(7, 8)
```

`@type` in the line before types the variable to whatever you want it to be, explicitly.
The nice thing about TypeScript is that you usually don't need to type things explicitly as it
can infer the type of the variable automatically.

## Running typechecking as a test

To add real typechecking, one that will fail the build, and not just show up in Visual Studio Code,
we need TypeScript in our package. Let's do that:

```sh
npm install --save-dev typescript
```

Great, we've installed TypeScript. Let's now run typechecking on our files:

```log
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

If we want to be serious about TypeScript, we need to configure it like the Pros. Let's do that,
by adding a `tsconfig.json` to our project root. We'll build this file slowly, understanding
each step:

```json
{
  "include": ["src/**/*.js", "src/**/*.d.ts"],
  "exclude": ["node_modules"]
}
```

First, we define which files we want to type check, and which we don't. This is pretty
self-explanatory, and you may need to change this to conform to where your JS files are, but
notice that we're adding `*.d.ts` files. We'll be using this later. Let's just ignore this for now.
And, yes, you do need to tell TypeScript to ignore `node_modules`, because otherwise it won't and
you will get i) long typechecking times, and ii) lots of errors in packages you don't care about.

Let's continue:

```json
{
  "compilerOptions": {
    "lib": ["es2020"],
    "moduleResolution": "node",
    "module": "CommonJS",
  },
  "include": ["src/**/*.js", "src/**/*.d.ts"],
  "exclude": ["node_modules"]
}
```

These new options defines the source code that it will get. We're telling it that it's going
to use the latest JavaScript defitions, and that we're using CommonJS.
If you're using ESM, use `esnext`.

Let's continue with the options that allow us to use JS files:

```json
{
  "compilerOptions": {
    "...": "...",
    "allowJs": true,
    "checkJs": true,
    "resolveJsonModule": true,
    "noEmit": true.
  },
  "...": "..."
}
```

* `allowJs` allows JS files to be typeChecked
* `checkJs` tells TypeScript to check _all_ JavaScript files, and not just those with `//@ts-check`
* `resolveJsonModule` tells Typescript that `require`-ing `.json` files is OK. It will even generate
  a type for the JSON in the file, so it will be typechecked correctly. Because we're using
  JavaScript, and `require`-ing JSON in JavaScript is fine, we turn this on.
* `noEmit` tells TypeScript not to emit transpiled files, which we don't really want because
  we already have our JavaScript files

Last, but not least, we have our regular TypeScript configurations. Let's go all in, and use the
strictest option, which tells TypeScript to go all in and do the strictest checks:

```json
{
  "compilerOptions": {
    "...": "...",
    "strict": true
  },
  "...": "..."
}
```

If you want less strict options, or any other TypeScript configuration, feel free to add them
here.

This produces our final `tsconfig.json`:

```json
{
  "compilerOptions": {
    "lib": ["es2020", "DOM"],
    "moduleResolution": "node",
    "module": "esnext",
    "resolveJsonModule": true,
    "allowJs": true,
    "checkJs": true,
    "noEmit": true,
    "strict": true
  },
  "include": ["src/**/*.js"],
  "exclude": ["node_modules"]
}
```

Now we can have TypeScript be part of our tests by adding it to our `scripts` in `package.json`:

```json
{
  "scripts": {
    "test": "... && tsc"
  }
}
```

Now, when we run `npm test`, Typescript (`tsc`) will run, look at our JSDoc typings,
and typecheck all our JS files.

We've got TypeScript, but without the transpilation.

## Typedefs, classes, and importing types

Sure, but how much of TypeScript do we have? Do we have `interface Foo {}`? `type Foo = ...`?
`class`? We do! We'll get to `interface` later, but let's start with `type Foo = ...`.

We'll define a function, `breakName`, that breaks a full name to the "first name" and "last name".
Please don't use this code in production, as it's an extremely naïve implementation:

```js
function breakName(name) {
  const [first, ...rest] = name.split(' ')

  return {firstName: first, lastName: rest.join(' ')}
}
```

Since I'm using Visual Studio Code, and you write the above code, I'll get a squiggly red line
on the `name` parameter:

![error when untyped parameter](img/jsdoc-error-on-unknown-type-in-breakname.png)

This is because I use `strict: true` in the `tsconfig.json`, and that tells TypeScript that
implicitly typing variables to `any` is not allowed, and thus forces you to define all function
parameters. So let's fix that, and as long as we're there, let's define the return value too:

```js
/**
 * @param {string} name
 * @returns {{firstName: string, lastName: string}}
 */
function breakName(name) {
  const [first, ...rest] = name.split(' ')

  return {firstName: first, lastName: rest.join(' ')}
}
```

We defined the `{firstName: string, lastName: string}` as a return value. As I said:
we have the full power of Typescript at your disposal. Notice the double curly braces `{{...}}`
when defininig the return type. The outer curly braces are needed by JSDoc, who's syntax
forces us to surround all types with curly braces, and the inner curly braces are for the TypeScript
type definition for an object.

Let's make it nicer, to make that return value a `type`? We can do that, using
the `@typedef` JSDoc:

```js
/**
 *
 * @typedef {{firstName: string, lastName: string}} BrokenName
 */
```

This is equivalent to the TypeScript code:

```ts
type BrokenName = {firstName: string, lastName: string}
```

To use it in the function, we change it a bit:

```js
/**
 * @param {string} name
 * @returns {BrokenName}
 */
function breakName(name) {
  const [first, ...rest] = name.split(' ')

  return {firstName: first, lastName: rest.join(' ')}
}
```

And to use it, we just:

```js
const brokenName = breakName('Gil Tayar')
console.log(brokenName) // => { firstName: 'Gil', lastName: 'Tayar' }
```

## Importing types from other modules

What if we wanted to do explicit typings in the above example? As above, we can just do this:

```js
/**@type {BrokenName} */
const brokenName = breakName('Gil Tayar')
```

But what happens if `breakName` and `BrokenName` come from another module? In TypeScript we can use:

```ts
import {BrokenName} from './names'

const brokenName: BrokenName = breakName('Gil Tayar')
```

But this is not possible in JavaScript, because JavaScript modules don't export types. So how
do we do the same in JSDoc? How do we use a type (or interface or class) that was defined in another
module? The answer is "use `import(type)`":

```js
/**@type {import('./names').BrokenName} */
const brokenName = breakName('Gil Tayar')
```

The `import` in JSDoc allows you to import "types stuff" from other modules and packages. And
if you get tired of writing `import('./names')` everywhere, you can always write:

```js
/**@typedef {import('./names').BrokenName} BrokenName */
```

to alias it in your other module.

## Using types from other NPM packages

Would this `import` work with other libraries? Definitely! Let's talk about using types from
other NPM package. We have three different types of libraries:

* Libraries that have type information, especially libraries that were written in TypeScript
* Libraries that have external type information from a `@types/...` package
* Libraries that don't have type information

Let's talk about each one, and how to use each.

## Using types from libraries that have type information

This one's easy: just `npm install` the package and use it. If it has type information,
TypeScript will find it, don't worry. Just use the stuff that are exported, and it will have
type information in it. Let's take an example:

```js
const slugify = require('@sindresorhus/slugify')

console.log(slugify('i ❤️ slugs')) // => i-slugs
```

We can even use the types that the library exports:

```js
/**@type {import('@sindresorhus/slugify').Options} */
const slugifyOptions = {separator: '_'}

console.log(slugify('i ❤️ slugs', slugifyOptions)) // => i_slugs
```

And if there's a typechecking error, TypeScript will fail:

```log
$ npm test

src/jsdoc-typing.js:39:23 - error TS2345: Argument of type '{ badOption: boolean; }' is not assignable to parameter of type 'Options'.
  Object literal may only specify known properties, and 'badOption' does not exist in type 'Options'.

39 slugify('something', {badOption: true})
                         ~~~~~~~~~~~~~~~
```

It works! We can use any TypeScript compatible library and use it and get the same typechecking
that TypeScript programs use.

## Using types from libraries that have external type information from a `@types/...` package

Let's try Lodash:

```js
const {map} = require('lodash')
```

If we typecheck this, we get:

```log
$ npm test

src/jsdoc-typing.js:5:23 - error TS7016: Could not find a declaration file for module 'lodash'. '/Users/giltayar/code/jsdoc-typing/node_modules/lodash/lodash.js' implicitly has an 'any' type.
  Try `npm i --save-dev @types/lodash` if it exists or add a new declaration (.d.ts) file containing `declare module 'lodash';`

5 const {map} = require('lodash')
```

Weird error. It says (if I may interpret), that it couldn't find type information for `lodash`.
And it won't continue without types, it doesn't want to continue. What can we do?

Theoretically, we can write the type declarations ourselves, but that's a lot of work, and we'll
see how to do it below. But another option is to use an existing, community-owned, database of
type definitions for lots of NPM packages. Let's see if Lodash has type definitions in this
database. To find out, just add a package that has these type definitions:

```sh
npm install --save-dev @types/lodash
```

It works! If a package has type information that is community-owned, it is probably in
`@types/<package-name>`, and if you install it, you will get typechecking on that library.
I've found that most of the type definitions in `@types/...` are of high quality. But not all
of them... Just remember that those type definitions are used in the same way by the TypeScript
community itself.

Now if we typecheck the file, we find we have type definitions for the `map` function we imported.
Yay!

### Using types from libraries that don't have type information

Let's try another NPM package. This time, this package doesn't have type information
embedded in it, _and_ doesn't have a community-built type package for it. As an example, we'll
use the package... [flowers](https://www.npmjs.com/package/flowers): It's a package that
lists about 400 different types of flowers, and is a proof that you can find _anything_ in the NPM
registry!

And it doesn't have a `@types` type package:

```sh
$ npm i @types/flowers
npm ERR! code E404
npm ERR! 404 Not Found - GET https://registry.npmjs.org/@types%2fflowers - Not found
...
```

So what happens when we add it to our code:

```js
const flowers = require('flowers')
```

Let's typecheck it: as expected, we get an error because TypeScript wants type definitions for
the module:

```log
$ npm test

src/jsdoc-typing.js:6:25 - error TS7016: Could not find a declaration file for module 'flowers'. '/Users/giltayar/code/jsdoc-typing/node_modules/flowers/index.js' implicitly has an 'any' type.
  Try `npm i --save-dev @types/flowers` if it exists or add a new declaration (.d.ts) file containing `declare module 'flowers';`

6 const flowers = require('flowers')
                          ~~~~~~~~~
```

If we were working in TypeScript, the solution would be simple: add a `declare module 'flowers'`
somewhere in our TypeScript code. This signals to TypeScript that the module exists,
and there are no type definitions for it, so anything coming out of it is just `any`. Of
course, we could go the extra mile, and even define the types for this module using:

```ts
declare module 'flowers' {
  // ... type definitions
}
```

But we don't have TypeScript code. And we don't TypeScript code, because we don't want to do
translation. And there's no equivalent to `declare module...` in JSDoc. But there is another
solution: `.d.ts` files. These are TypeScript "declaration files" that describe the types and API
of a module, and are actually what is exported by packages when they have type definitions,
and is what is exported by those type definition packages in the `@types/...` repository. They
include _only_ type definitions, and have no executable code in them.

And we can use them by having a few of those in our project. So we create a file, let's call it
`global.d.ts`, and add the `declare module 'flowers'` there:

```ts
// src/global.d.ts
declare module 'flowers'
```

That is not enough, though. We need the TypeScript type checker to know about this file, so
we need to `include` it in the `tsconfig.json`:

```json
{
  "...": "",
  "include": ["src/**/*.js", "src/**/*.d.ts"],
  "exclude": "..."
}
```

By adding `src/**/*.d.ts`, we tell TypeScript to add `global.d.ts` to the type checking, and now
if we type check our file using `npm test`, it completes with no error.

`global.d.ts` is a nice escape hatch because you can use any type script type definitions in there.
We'll be using it later for some advanced TypeScript stuff.

## Advanced Typescript

Can we do other, more advanced stuff that we can do in TypeScript? Yes! Almost everything
you can do in TypeScript, you can do with JSDoc. Let's start with `class`

### Class type definitions

```js
class Person {
  /**@type {string}*/
  firstName
  /**@type {string}*/
  lastName

  /**
   * @param {string} firstName
   * @param {string} lastName
   */
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }

  /**
   * @returns {string}
   */
  fullName() {
    return `${this.firstName} ${this.lastName}`
  }
}
```

The above is a class `Person`, with two properties (`firstName` and `lastName`) and a method
(`fullName()`), that are all typechecked. We didn't really use any new JSDoc capabilities,
just the regular `@type`, `@param`, and `@return`. But since it's inside a class, TypeScript
knows how to deal with it, and can typecheck it. So the following bad code...

```js
const p1 = new Person('Gil', 'Tayar')
p1.someName
p1.fullName(4)
```

...will fail typechecking:

```log
$ npm t

src/jsdoc-typing.js:50:4 - error TS2339: Property 'someName' does not exist on type 'Person'.

50 p1.someName
      ~~~~~~~~

src/jsdoc-typing.js:51:13 - error TS2554: Expected 0 arguments, but got 1.

51 p1.fullName(4)
               ~
```

### Type Casting

In TypeScript, we sometimes need to cast something, to tell the typechecker that we _know_ that
the thing is of a certain type. As an example, take this example:

```js
const numberOrString = Math.random() <= 1 ? "This is a string" : 100;
console.log(numberOrString.toUpperCase())
```

The typechecker will fail on `numberOrString.toUpperCase()` because TypeScript
correcrly infers the type of `numberOrString` to be `number | string`. And yet, we _know_
that the type of `numberOrString` is always `string` because `Math.random()` always returns a
number in the range 0…1. If we were in TypeScript, we could write:

```ts
const numberOrString = Math.random() <= 1 ? "This is a string" : 100;
console.log(numberOrString.toUpperCase() as string)
```

Does JSDoc have a comparable mechanism? The answer is yes:

```js
const numberOrString = Math.random() <= 1 ? "This is a string" : 100;
console.log(/**@type {string}*/(numberOrString.toUpperCase()))
```

We've already seen that `@type` can define the

> This is the ugliest part of JSDoc, and I truly wish there was a nicer way of typecasting, but
  there isn't unfortunately. It's the only place in JSDoc land that I really dislike.

### Templates

Now for the craziest thing in JSDoc: you can even do templates. Let's go wild. Let's write
a function, `mapValue`, that receives an object, and a map function and returns an object
where the keys are the same, but the values are mapped:

```js
function mapValues(object, mapFunction) {
  return Object.fromEntries(Object.entries(object).map(([key, value]) => [key, mapFunction(value)]))
}
```

Nice and simple. Let's add types, without templates, using `any`:

```js
/**
 * @param {Record<any, any>} object
 * @param {(t: any) => any} mapFunction
 *
 * @returns {Record<any, any>}
 */
function mapValues(object, mapFunction) {
  return Object.fromEntries(Object.entries(object).map(([key, value]) => [key, mapFunction(value)]))
}
```

Still nice (`Record` is a builtin type in TypeScript,
and defines an object with key type and value tyep). Better than before, because we know that
the first parameter needs to be an object, and the second one a mapping function.

But if we write this code:

```js
const result = mapValue({a: 4}, x => x + 1)
result.x
```

TypeScript won't catch us on the second line, even though it's obvious that `result` doesn't have
an `x` property.

If we were in TypeScript land, we would write something like this:

```ts
<K extends string|number|symbol, T, W> function (obj: Record<K, T>, mapFunction: (t: T) => W): Record<K, W> {
  //...
}
```

Is there an equivalent in JSDoc? Incredibly enough, there is:

```js
/**
 * @template {string|number|symbol} K
 * @template T
 * @template W
 * @param {Record<K, T>} object
 * @param {(t: T) => W} mapFunction
 *
 * @returns {Record<K, W>}
 */
function mapValues(object, mapFunction) {
  return Object.fromEntries(Object.entries(object).map(([key, value]) => [key, mapFunction(value)]))
}
```

This is the equivalent to the above TypeScript template code: three template variables
(`K`, `T`, and `W`), where `K` also `extends string|number|symbol`, and the parameters
are defined according to those template parameters. And now, if we use the function incorrectly...

```js
const result = mapValue({a: 4}, x => x + 1)
result.x
    //~~
   //Property 'x' does not exist on type 'Record<"a", number>'
```

...we get the correct error.

There's just one small problem: `mapValues` itself doesn't typecheck. `Object.fromEntries/entries`
doesn't return the correct types and resets them to a generic (`Record<any, any>`)
which makes TypeScript (rightfully) fail the typechecking because the return type of `mapValue`
is more specific. Two options: typecast it, or just ignore the error. This time, I'll ignore the
error:

```js
function mapValues(object, mapFunction) {
  //@ts-ignore-error
  return Object.fromEntries(Object.entries(object).map(([key, value]) => [key, mapFunction(value)]))
}
```

The `//@ts-ignore` tells TypeScript to ignore the TypeScript errors in the line following it.
Use it, but use it sparingly. An even better option, in my opinion, is the companion
`@ts-expect-errpr`:

```js
function mapValues(object, mapFunction) {
  //@ts-expect-error
  return Object.fromEntries(Object.entries(object).map(([key, value]) => [key, mapFunction(value)]))
}
```

This will ignore any typechecking failures in the next line, but **will fail the typecheck** if
there are no errors. It's a good way to ensure that that ignoring is really needed.

## Using a bit of TypeScript typings

We've seen that most everything you can do in TypeScript, you can do in JavaScript with JSDocs,
and get full typechecking. But there are some things we can do in TypeScript that are
not (yet?) possible using JSDoc typings. One example is, incredibly enough, `interface`: you can
define a type alias to an interface, but you can't define an interface. So the equivalent of

```ts
type Point = {x: number, y: number}
```

Would be

```js
/**
 * @typedef {{x: number, y: number}} Point
*/
```

But surprisingly enough, there is no equivalent in JSDoc to

```ts
interface Point {
  x: number,
  y: number
}
```

So what do we? Well, we _can_ get by with only type aliases, but it would be nice
to be able to declare an interface. And it turns out that we can: we'll use a new
`point-type.d.ts` file to define the `interface Point`, and then use `import` to use it. Let's
start with the type definition:

```ts
// point-type.d.ts
export interface Point {
  x: number
  y: number
}
```

And now, since we've already told TypeScript to include `*.d.ts` files in the typechecking, we
can use it in our JavaScript code:

```js
/**
 * @param {import('./point-type').Point} point
 * @param {number} dx
 * @param {number} dy
 *
 * @returns {import('./point-type').Point}
 */
function move(point, dx, dy) {
  return {x: point.x + dx, y: point.y + dy}
}

console.log(move({x: 2, y: 4}, 1, 1))
```

We've used `import('./point-type).Point` to "get" the exported type, and used it in our regular
JSDoc code. Mission accomplished!

> Note: you cannot use `declare module` and `export` in the same `.d.ts` file
  (for some arcane reason I can't understand),
  so you need at least two separate `.d.ts` files if you
  have both `declare module ...` and `export ...`.

## Exporting `.d.ts` files

The last functionality that we have in TypeScript, and which we would like to duplicate using
JSDoc typing, is the ability to generate an NPM package that also exposes its type definitions.
Just like `slugify` does above. It's actually possible, and even easy. Let' start on the journey.

First, we need to generate the `.d.ts` file that will contain all the type definitions we have.
We'll run it in a `build` script, because that is what it is: building the definition files
from source. This build script doesn't need to be run in development, but it does need to be
run before we publish the package to NPM. This is how it will look like:

```json
  "scripts": {
    "build": "tsc  --noEmit false --emitDeclarationOnly true",
    "test": "tsc",
    "start": "node src/jsdoc-typing.js"
  }
```

We're adding two options when running `tsc` because we _do_ want to emit files (`--noEmit false`),
but want to emit only `.d.ts` files (`--emitDeclarationOnly true`)

But we need to tell it where to emit the `.d.ts` files, but we can do that in the `tsconfig.json`:

```json
{
  "...": "",
  "compilerOptions": {
    "...": "",
    "declarationDir": "types",
    "declaration": true
  }
}
```

This tells "TypeScript" to omit the declarations to the `types` directory. We also need to make
sure that when we publish this package, the `types` directory will also be published, so
if you're enable-listing the files you publish by using `files` in `package.json`, don't forget
to add the `types` directory there:

```json
{
  "files ": ["src", "types"]
}
```

And, while we're at it, we don't really want the `types` directory to be source controlled,
because it's a file that generated by source code, so we add it to `.gitignore`:

```txt
# .gitignore
node_modules
types
```

Last thing we need to do before we publish this package, is add a property in the `package.json`
that specifies where the root `.d.ts` file is, so that any package that `npm install`-s this package
will know how to get at it's type definitions. So let's add the `"types"` field:

```json
{
  "name": "jsdoc-typing-example",
  "main": "./src/jsdoc-typing.js",
  "types": "./types/jsdoc-typing.d.ts",
```

Just like you define the package's entry point using `main`, so you define your package's type
definition entry point using `types`. And this `.d.ts` was generated using the `npm run build`
script we created earlier, and with the help of the `tsconfig.json` properties we added
which told it where to generate those type definition files.

### One last fine tune

Once we `npm publish` this package, we can use it in any JS (or TS) file, and it will enable
code completion and typeching of the available exports:

```js
//@ts-check
const example = require('jsdoc-typing-example')

example.add('wrong-type', 4)
        //  ~~~~~~~~~~~~
 ```

Mission accomplished!

## Drawbacks

Are there any drawbacks? Yes, of course there are. The four I can think of are:

* The type definitions in JSDoc comments are sometimes clumsy and too verbose at times
* You can't do everything with those type definitions, and while there is a solution for that
  (using `.d.ts` files as described above), it feels like a kludge
* The type casting syntax is *very* clumsy and is a wart on the code. And unlike
  the JSDoc comments, it sits _right_ in your code
* The community is

## Summary

But, weighing the drawbacks, I still feel that this is a better solution than TypeScript
transpiling.

You can use JSDoc typing to do _everything_ you can do with TypeScript, but with pure JavaScript,
using the ability of TypeScript to read type definitions encoded in JSDoc comments,
and thus enable embedding type information in your JavaScript files,
and using that type information to typecheck your JS files, as if they were TypeScript.

So go ahead and use JSDoc typings: **all the benefits of TypeScript, without the drawbacks!**

## References

* Great documentation on JSDoc comments on the Typescript site: <https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html>
* The Github repo with the sample code above, and a complete example of such a JSDoc package: <https://github.com/giltayar/jsdoc-typing>


