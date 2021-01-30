---
title: "Using ES Modules in Node.js: A Practical Guide"
description:
ESM is ready for use in Node.js. This guide shows you how, and how to avoid all the small gotchas.
The guide covers the basics, but also discusses how to write packages that
can be dual-mode (ESM and CJS), how to configure ESLint, Mocha, and Testdouble, and how
to use TypeScript with ESM.

date: 2021-02-01
tags:
  - ESM
  - JavaScript
  - Node.js
layout: layouts/post.njk
---
<!-- markdownlint-disable MD029 -->

(Hey, if you want to come work with me at Roundforest, and try out JSDoc typing, feel free to
find me on [LinkedIn](https://www.linkedin.com/in/giltayar/) or on Twitter (@giltayar))

ES Modules are the future of modules in JavaScript. They already are the rule in the frontend,
but till now they couldn't have been used in Node.js. Well, it seems that now they can.
Moreover, the Node.js community is fast at work at adding support for Node.js ESM. This includes
tools like Mocha, Ava, and even Jest (although in Jest the support is incremental). Moreover,
ESlint and TypeScript work nicely with ESM, albeit with a few gotchas.

This guide shows you how to use ESM in Node.js, detailing the basics, and also the gotchas
that you need to be careful with. You can find all the code in the
[companion code repository](https://github.com/giltayar/esm-in-nodejs-guide). It's a monorepo
where each package displays a certain facet of Node.js ESM support. This post goes through
each of the packages, explaining what was done there, and what are the gotchas.

> Note: this guide covers **Node.js ESM** and does not cover Browser ESM.

## What do I mean by ESM in Node.js? Don't we have that already?

ESM is the standard JavaScript module system (ESM is a shorcut for
EcmaScript Modules, whereby "EcmaScript" is the official name for the JavaScript language).
This is in constrast to the regular Node.js module system, which is CommonJS (CJS for short). It's
this syntax:

```js
// add.js
export function add(a, b) {
  return a + b
}

// main.js
import {add} from './add.js'
```

It was standardized in 2015, but it took awhile for browsers to support this, and it took even longer
for Node.js to support it (the final stable version in Node.js was finalized only in 2020!).
If you want more information, you can see my [talk at Node.TLV](https://www.youtube.com/watch?v=kK_3OP0uJ0Y).
In the talk, at the end, I discuss whether ESM is ready for work, and I say that it's not
quite, and people should start migrating in a year or two. Well, that year is NOW, and it's READY,
and this guide will prepare you for that.

Some of you may be nodding your head and asking themselves: aren't we already using that? Well,
if you are, you're transpiling your code using Babel or TypeScript, which support ESM
out of the box, and transpile it to CJS. The ESM this post is talking about is _native_ ESM
that is supported by Node.js without transpiling. While syntactically it is the same, there
are small differences between it and Babel/TypeScript ESM, differences which are discussed
in my Node.TLV talk above. More importantly, native ESM in Node.js does not need transpiling,
with all the problems that come with it.

## Give it to me straight. Can I start using ESM in Node.js?

Yes. Mostly yes. All the tooling I use supports it, but there are two BIG gotchas that are probably
hard to swallow for some people, gotchas that are hard to workaround:

* Jest support for ESM in Node.js is [experimental](https://jestjs.io/docs/en/ecmascript-modules)
* Jest experimental support does not support yet mocking modules
  (regular function/object mocking is supported)
* `proxyquire` and other popular module mockers do not yet support ESM

The biggest problem is the lack of support in module mockers for ESM. There _is_ one mocking
library that does support ESM, and we use that in this guide.

So can you live with this? If you can, then going all in with ESM in Node.js is now totally
possible. I've been using it for four months now, with zero problems. Actually, it feels
like VSCode support for ESM is much better than for CJS, so I suddenly get auto imports of modules,
which I didn't get before in the CJS world.

## Finally, the guide

So let's start. Each section is a separate package/folder in the
[companion repo](https://github.com/giltayar/esm-in-nodejs-guide).

## The simplest Node.js ESM project (`01-simplest-mjs`)

Companion code: <https://github.com/giltayar/esm-in-nodejs-guide/tree/main/01-simplest-mjs>.

This is the simplest package. Let's start by exploring `package.json`, and the new
`exports` field.

### `main` and `.mjs` (in `package.json`)

```json
{
  "name": "01-simplest-mjs",
  "version": "1.0.0",
  "description": "",
  "main": "src/main.mjs",
}
```

The main entry point is `src/main.mjs`. Why do we use the `.mjs` extension? Because in Node.s
ESM, the `.js` extension is reserved for CJS and `.mjs` means that this is an ES module
(in the next section, we'll see how to change that). We'll takl a bit more about that in the next
part.

Let's continue exploring with `main.mjs`:

### Importing using extensions (`main.mjs`)

```js
// main.mjs
import {bannerInColor} from './banner-in-color.mjs'

export function banner() {
  return bannerInColor('white')
}
```

Node.js ESM _forces_ you to specify the full relative path to the file, _including the extension_.
The reason they did this is to be compatible with Browser ESM that also forces you do to this. So
don't forget that extension!

Unfortunately, VSCode doesn't like the `.mjs` extension and so Ctrl/Cmd+Clicking it won't work,
and its built in intellisense doesn't work on it.

> **Gotcha**: VSCode doesn't like the `.mjs` extension and ignores files with that extension. In
  the next section we'll see how to deal with that, so it's not a _real_ gotcha.

The `main.mjs` exports the `banner` function, which will be tested in `test/tryout.mjs`. But
first, let's explore `banner-in-color.mjs`:

### Importing ESM and CJS paclages (`banner-in-color.mjs`)

We've seen how we can import ESM modules. Let's see how to import other packages:

```js
// banner-in-color.mjs
import {join} from 'path'
import chalk from 'chalk'
const {underline} = chalk
```

We first see that we can import Node.js internal packages like `path`, because Node.js
exposes them as "dual-mode": they are exposed as both CJS and ESM modules.

And if we had an ESM package in NPM, the same could have been used to import the ESM package.
But most of the pacakges NPM has are still CJS packages. As you can see in the second line, where
we import `chalk`,  CJS packages can also be imported using `import`. But for the most part,
you can only use "default" importing, and not "named" imports. So if you could do this in CJS:

```js
const {underline} = require('chalk')
```

You _cannot_ do this with CJS packages:

```js
import {underline} from 'chalk`
```

And you can only import the default (non-named) import, and use destructuring later:

```js
import chalk from 'chalk'
const {underline} = chalk
```

Why is this? Because ESM does not allow _executing_ a module to determine what the exports are,
and executing a module is the only reliable way of determining what the exports are.
Node.js actually tries very hard to figure out what the named exports are (using a very
fast parsing of the module), but my experience is that most packages I've tried this with
don't really work, and I need to fall back to default importing.

> **Gotcha**: importing a CJS module is easy, but you can't use named imports and need
  to add a second line to destructure out the named imports.

I believe that in 2021, more and more packages will have ESM entry points that export
themselves as ESM with the proper named exports. But for now, you may need the double lines
to use CJS pacakges.

### Top-level await (in `banner-in-color.mjs`)

Continuning our exploration of `banner-in-color.mjs` we find this extraordinary line
that reads a file from the disk:

```js
// banner-in-color.mjs
const text = await fs.readFile(join(__dirname, 'text.txt'), 'utf8')
```

Why so extraordinary? Because of that `await`. This is an `await` that is outside an `async`
function, and is at the top level of the code. This kind of `await` is called "top-level await"
and is supported since Node v14. It is extraordinary because it is the only feature in Node.js
that is available _only_ in ESM modules (i.e. not available in CJS). Why is that? Because
ESM is an async module system, and so supports async operations, while CJS is loaded synchronously
and so cannot support `await`.

Great feature, and only in ESM!

But notice the use of `__dirname`. Let's discuss that.

### `__dirname` (in `banner-in-color.mjs`)

If you try to use `__dirname` in ESM, you will find that it is not available
(just like `__filename`). But if you need it, you can quickly find it using these lines:

```js
// banner-in-color.mjs
import url from 'url'
const __dirname = url.fileURLToPath(new URL('.', import.meta.url))
```

Let's deconstruct this code. First off, `import.meta.url` is similar to `__filename`, except
that it's a URL and not a file path. Why URL's? Because ESM is defined in terms of URLs and not
file paths.

Now that we have the URL of the current file, we need th parent URL, and we use
`new URL('.', import.meta.url)` to get at it (why this works is out of scope of this guide).
Finally, to get at the file path and not the URL, we need a function that converts between the two
and the internal `url` module gives us this via the `ur.fileURLToPath` function.

Finally, we put it in a variable called `__dirname`, named so out of deference to the Node.js
tradition 😀.

### Testing this module

* To test this module, cd to the directory, and then:

```sh
npm install
npm test
```

The test will run `test/tryout.mjs`, which will `import` the `src/main.mjs`, which will
use (as we saw above) various CJS and ESM imports,
to export a function with the colored banner with the answer
(to life, the universe, and everything) `42`. It will assert that the answer is such,
and `console.log` it so we can see it with all its glory.

Done!

## Using the `.js` extension for ESM (`02-simplest-js`)

As we saw in the previous section, the `.mjs` extension is problematic.
We want our `.js` extension back, and that is what we will do in this section, with a very
simple change to `package.json`:

### `type: module` in `package.json`

```json
{
  "name": "02-simplest-js",
  "version": "1.0.0",
  "description": "",
  "type": "module",
  "main": "src/main.js",
```

There is a very simple way to make all your `.js` files be interpreted as ESM and not as CJS:
just add `"type": "module"` to your `package.json`, like above. That's it. From that point on,
all `.js` files will be interpreted as ESM, and it means that your whole code can use the `.js`
extension.

You can still use `.mjs` and it wil always be ESM, _and_ if you want a CJS module in your code,
you can use the new `.cjs` extension (we'll see how we use that in the "dual mode library" section).

That's it. The rest of the code in this director uses `.js`, and when importing, also uses the
`.js` extension:

```js
// src/main.js
import {bannerInColor} from './banner-in-color.js'
```

On to the next section, where we'll learn about an important feature for ESM:

## The `exports` field (`03-exports`)

Let's look at the new directory, `03-exports`, and see this new `exports` field:

```json
{
  "name": "03-exports",
  "type": "module",
  "main": "./src/main.js",
  "exports": {
    ".": "./src/main.js"
  }
```

The main entry point, just like in the previous section, is `src/main.js`.

While you can use `main` to define the entry point, the correct ESM way to define the package
entry point is `exports`. Note that in `exports`, the path MUST start with a `.` (for reasons
we'll see in the section on multiple entry points).

Using `exports`, you define the entry point (in the above case `.`), and what that entry point
points to: `./src/main.js`. It looks very verbose, but we'll see why this syntax is this way
shortly.

Why use `exports` if `main` is enough? Because once you define `exports`, then no `import`
can deep-link into the `packages`, i.e. if the package was published, another
package could import it by using

```js
import {banner} from '01-simplest-mjs'
```

But can't do this

```js
import {banner} from '01-simplest-mjs/src/main.mjs'
```

Note that `exports` is not just used in ESM, but is also available in CJS. So if
`exports` is defined in a CJS package, then no deep linking is allowed in CJS too.

Another import benefit can be seen in `test/tryout.mjs` in this package:

```js
import {banner} from '01-simplest-mjs'
```

This is code from the existing package, yet it can self-reference itself using the name of
the package!

> **Gotcha**: unfortunately, none of the toolings (eslint, typescript, etc) understand that this is
  is a self-reference that is allowed in Node.js and flag this as an error. But you can always
  disable the error on this specific line, as it most definitely works in runtime.

So `exports` gives us a nice way of "hiding" our inner modules and not exposing them ourside the
package. This is great, but has a nasty side-effect: some tools need access to all the
packages `package.json`, and using `exports` bars them from accessing it. To enable
them to access the `package.json`, we add another line to the `package.json` "exports":

```json
  "exports": {
    ".": "./src/main.js",
    "./package.json": "./package.json"
  }
```

The second line (`"./package.json": "./package.json"`) tells Node.js that
using `import '03-exports/package.json'` or `require('03-exports/package.json')` is OK.

> **Gotcha**: you can start using `exports`,
  but please do include the `./package.json` escape hatch and also
  continue to include `main` as some tooling (e.g. TypeScript) does not yet understand `export`.

## Multiple exports (`04-exports`)
