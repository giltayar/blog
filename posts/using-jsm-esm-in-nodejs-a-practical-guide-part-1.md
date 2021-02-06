---
title: "Using JS Modules (ESM) in Node.js: A Practical Guide (Part 1)"
description: |
  JSM (formerly ESM) is ready for use in Node.js.
  This guide shows you how, and how to avoid all the small gotchas.
  The guide covers the basics, but also discusses how to write packages that
  can be dual-mode (JSM and CJS), how to configure ESLint, Mocha, and Testdouble, and how
  to use TypeScript with JSM.
date: 2021-02-05
tags:
  - JSM
  - JavaScript
  - Node.js
layout: layouts/post.njk
---

<!-- markdownlint-disable MD029 -->
<!-- markdownlint-disable MD033 -->

![JSM Logo](/img/jsm-logo.png)

(Hey, if you want to come work with me at Roundforest, and try out JSM on Node.js, feel free to
find me on [LinkedIn](https://www.linkedin.com/in/giltayar/) or on Twitter (@giltayar))

In the text below, I use JS Modules (JSM) as the new nomenclature for ESM.

JS Modules are the future of modules in JavaScript. They already are the rule in the frontend,
but till now they couldn't have been used in Node.js. Well, it seems that now they can.
Moreover, the Node.js community is fast at work at adding support for Node.js JSM. This includes
tools like Mocha, Ava, and even Jest (although in Jest the support is incremental). Moreover,
ESlint and TypeScript work nicely with JSM, albeit with a few gotchas.

This guide shows you how to use JSM in Node.js, detailing the basics, and also the gotchas
that you need to be careful with. You can find all the code in the
[companion code repository](https://github.com/giltayar/jsm-in-nodejs-guide). It's a monorepo
where each package displays a certain facet of Node.js JSM support. This post goes through
each of the packages, explaining what was done there, and what are the gotchas.

This guide turned out to be pretty long, so I broke it into three parts:

1. The basics (this document)
1. ["exports" and its uses (including dual-mode libraries)](../using-jsm-esm-in-nodejs-a-practical-guide-part-2/)
1. [Tooling and Typescript](../using-jsm-esm-in-nodejs-a-practical-guide-part-3/)

> Note: this guide covers **Node.js JSM** and does _not_ cover Browser JSM.

## What do I mean by JSM in Node.js? Don't we have that already?

JSM is the standard JavaScript module system (JSM is a shorcut for
JavaScript Modules, and is also called ESM, or EcmaScript modules,
whereby "EcmaScript" is the official name for the JavaScript language).
JSM is the "newer" module system, and is supposed to be a replacement for the
regular Node.js module system, which is CommonJS (CJS for short), altought CommonJS will probaby
stil be with us for a very very long time. The module syntax is this:

```js
// add.js
export function add(a, b) {
  return a + b
}

// main.js
import { add } from "./add.js"
```

(An intro to JSM is out of scope of this guide, but you can find it _everywhere_ today on the
internet)

JSM was standardized in 2015, but it took awhile for browsers to support this,
and it took even longer for Node.js to support it
(the final stable version in Node.js was finalized only in 2020!).
If you want more information, you can see my
[talk at Node.TLV](https://www.youtube.com/watch?v=kK_3OP0uJ0Y). In the talk, at the end,
I discuss whether JSM is ready for work, and I say that it's not quite there yet,
and people should start migrating to it in a year or two. Well, that year is NOW, and it's READY,
and this guide will prepare you for that.

Some of you may be nodding your head and asking themselves: aren't we already using that? Well,
if you are, then you're transpiling your code using Babel or TypeScript, which support JSM
out of the box and transpile it to CJS. The JSM this post is talking about is _native_ JSM
that is supported by Node.js without transpiling. While syntactically it is the same, there
are small differences between it and Babel/TypeScript JSM, differences which are discussed
in my Node.TLV talk above. Most importantly, native JSM in Node.js does not need transpiling,
and so doesn't come with the baggage of problems transpiling comes with.

## Give it to me straight. Can I start using JSM in Node.js?

Yes. Mostly yes. All the tooling I use supports it, but there are two BIG gotchas that are probably
hard to swallow for some people, gotchas that are hard to workaround:

- Jest support for JSM in Node.js is [experimental](https://jestjs.io/docs/en/ecmascript-modules)
- Jest experimental support does not support yet mocking modules
  (regular function/object mocking is supported)
- `proxyquire` and other popular module mockers do not yet support JSM (although `testdouble`
  fully support it)

The biggest problem is the lack of support for module mockers. There _is_ one mocking
library that does support JSM (`testdouble`[https://www.npmjs.com/package/testdouble]),
and we use that in this guide.

So can you live with this? If you can, then going all in with JSM in Node.js is now totally
possible. I've been using it for four months now, with zero problems. Actually, it feels
like VSCode support for JSM is much better than for CJS, so I suddenly get auto imports of modules,
and other goodies, which I didn't get before in the CJS world.

## The guide to Node.js JSM

- Part 1 (this document): the basics of JSM

1. [The simplest Node.js JSM project](#section-01)
2. [Using the `.js` extension for JSM](#section-02)

- Part 2: "exports" and its uses (including dual-mode libraries)

3. [The `exports` field](../using-jsm-esm-in-nodejs-a-practical-guide-part-2/#section-03)
4. [Multiple exports](../using-jsm-esm-in-nodejs-a-practical-guide-part-2/#section-04)
5. [Dual-mode libraries](../using-jsm-esm-in-nodejs-a-practical-guide-part-2/#section-05)

- Part 3: Tooling and Typescript

6. [Tooling](../using-jsm-esm-in-nodejs-a-practical-guide-part-3/#section-06)
7. [TypeScript](../using-jsm-esm-in-nodejs-a-practical-guide-part-3/#section-07)

This guide comes with a monorepo that has 7 directories, each directory being a package
that demonstrates the above sections of the Node.js support for JSM. You can find the monorepo
[here](https://github.com/giltayar/jsm-in-nodejs-guide).

## <a id="section-01"></a>The simplest Node.js JSM project

Companion code: <https://github.com/giltayar/jsm-in-nodejs-guide/tree/main/01-simplest-mjs>

This is the simplest package, and demonstrates the basics of the basics.
Let's start by exploring `package.json`, and the new `exports` field.

### `main` and `.mjs`

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/01-simplest-mjs/package.json>

```json
{
  "name": "01-simplest-mjs",
  "version": "1.0.0",
  "description": "",
  "main": "src/main.mjs"
}
```

The main entry point is `src/main.mjs`. Why does the file use the `.mjs` extension? Because in
Node.js JSM, the `.js` extension is reserved for CJS and `.mjs` means that this is a JS module
(in the next section, we'll see how to change that). We'll talk a bit more about that in the next
part.

Let's continue exploring `main.mjs`.

### Importing using extensions

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/01-simplest-mjs/src/main.mjs>

```js
// src/main.mjs
import {bannerInColor} from "./banner-in-color.mjs"

export function banner() {
  return bannerInColor("white")
}
```

Look at the import statement that imports `banner-in-color`:
Node.js JSM _forces_ you to specify the full relative path to the file, _including the extension_.
The reason they did this is to be compatible with Browser JSM (when using JSM in browsers,
you always specify the full name of the file, including the extension). So
don't forget that extension! (You can understand more about this in my
[talk at Node.TLV](https://www.youtube.com/watch?v=kK_3OP0uJ0Y))

Unfortunately, VSCode doesn't like the `.mjs` extension and so Ctrl/Cmd+Clicking it won't work,
and its built in intellisense doesn't work on it.

> **Gotcha**: VSCode doesn't like the `.mjs` extension and ignores files with that extension. In
> the next section we'll see how to deal with that, so it's not a _real_ gotcha.

This `main.mjs` exports the `banner` function, which will be tested in `test/tryout.mjs`. But
first, let's explore `banner-in-color.mjs`, which contains most of the implementation of
the `banner()` function.

### Importing JSM and CJS paclages

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/01-simplest-mjs/src/banner-in-color.mjs>

We've seen how we can import JSM modules. Let's see how to import other packages:

```js
// src/banner-in-color.mjs
import {join} from "path"
import chalk from "chalk"
const {underline} = chalk
```

We can import Node.js internal packages like `path` easily, because Node.js
exposes them as JS modules.

And if we had a JSM package in NPM, the same could have been used to import the JSM package.
But most of the pacakges NPM has are still CJS packages. As you can see in the second line, where
we import `chalk`, CJS packages can also be imported using `import`. But for the most part,
when importing CJS modules, you can only use "default" importing, and not "named" imports. So
while you can import named imports in a CJS file:

```js
// -a-cjs-file.cjs
const {underline} = require("chalk")
```

You _cannot_ do this in a JSM file:

```js
// -a-jsm-file.mjs
import {underline} from 'chalk'
```

You can only import the default (non-named) import, and use destructuring later:

```js
import chalk from "chalk"
const {underline} = chalk
```

Why is this? It's complicated, but the gist of it is that when loading modules,
JSM does not allow _executing_ a module to determine what the exports are,
and so the exports need to be determined statically.
Unfortunately, in CJS, executing a module is the only reliable way of determining
what the exports are. Node.js actually tries very hard to figure out what the named exports are
(by parsing the module using a very fast parser),
but my experience is this method doesn't work for most packages that I've tried this with,
and I need to fall back to default importing.

> **Gotcha**: importing a CJS module is easy, but you mostly can't use named imports and need
  to add a second line to destructure out the named imports.

I believe that in 2021, more and more packages will have JSM entry points that export
themselves as JSM with the proper named exports. But for now, you may need the additional
destructuring to use named imports from CJS pacakges.

### Top-level await

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/01-simplest-mjs/src/banner-in-color.mjs>

Continuning our exploration of `banner-in-color.mjs` we find this extraordinary line
that reads a file from the disk:

```js
// src/banner-in-color.mjs
const text = await fs.readFile(join(__dirname, "text.txt"), "utf8")
```

Why so extraordinary? Because of that `await`. This is an `await` that is outside an `async`
function, and is at the top level of the code. This kind of `await` is called "top-level await"
and is supported since Node.js v14. It is extraordinary because it is the only feature in Node.js
that is available _only_ in JSM modules (i.e. not available in CJS). Why is that? Because
JSM is an async module system, and so supports async operations when loading the module,
while CJS is loaded synchronously and so cannot support `await`.

Great feature, and only in JSM! 🎉🎉🎉🎉

But notice the use of `__dirname` in the line above. Let's discuss that.

### `__dirname`

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/01-simplest-mjs/src/banner-in-color.mjs>

If you try to use `__dirname` in JSM, you will find that it is not available
(just like `__filename`). But if you need it, you can quickly determine it using these lines:

```js
// src/banner-in-color.mjs
import url from "url"

const __dirname = url.fileURLToPath(new URL(".", import.meta.url))
```

Complex? Yes. So let's deconstruct this code to understand it.

First off, the expression `import.meta.url` is part of the JSM spec, and its purpose
is the same as the CJS `__filename`, except that it's a _URL_ and not a file path.
Why URL-s? Because JSM is defined in terms of URLs and not file paths (to be browser-compatible).
BTW, the URL we get is not an HTTP URL. It's a "`file://...`" URL.

Now that we have the URL of the current file, we need the parent URL to get at the directory,
and we use `new URL('.', import.meta.url)` to get at it
(why this works is out of scope of this guide).
Finally, to get at the file path and not the URL, we need a function that converts between the two
and the Node.js `url` module gives us this via the `url.fileURLToPath` function.

Finally, we put directory path in a variable called `__dirname`,
named so out of deference to Node.js traditions 😀.

### Testing this module

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/01-simplest-mjs/test/tryout.mjs>

```js
// test/tryout.mjs
import assert from 'assert'
import {banner} from '../src/main.mjs'

assert.strict.match(banner(), /The answer is.*42/)

console.log(banner())
```

The test will run `test/tryout.mjs`, which will `import` the `src/main.mjs` module, which will
use (as we saw above) various CJS and JSM imports, to export a function
with the colored banner with the answer (to life, the universe, and everything) `42`.
It will assert that the answer is such, and `console.log` it so we can see it with all its glory.

To run the test, cd to `01-simplest-js`, and run:

```shell
npm install
npm test
```

Yes! We've written our first JSM package! Now let's do the same, but with a `.js` extension!

## <a id="section-02"/></a>Using the `.js` extension for JSM

Companion code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/02-simplest-js>

As we saw in the previous section, the `.mjs` extension is problematic, because tooling still
doesn't fully support it well. We want our `.js` extension back,
and that is what we will do in this section, with a very simple change to `package.json`.

### `type: module`

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/02-simplest-js/package.json>

```json
{
  // package.json
  "name": "02-simplest-js",
  "version": "1.0.0",
  "description": "",
  "type": "module",
  "main": "src/main.js",
```

There is a very simple way to make all your `.js` files be interpreted as JSM and not as CJS:
just add `"type": "module"` to your `package.json`, like above. That's it. From that point on,
all `.js` files will be interpreted as JSM, so your whole code can now use the `.js` extension.

You can still use `.mjs` and it wil always be JSM. Moreover, if you need a CJS module in your code,
you can use the new `.cjs` extension (we'll see how we use that in the "dual mode library" section).

That's it. The rest of the code in this directory uses `.js`, and when importing, also uses the
`.js` extension:

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/02-simplest-js/src/main.js>

```js
// src/main.js
import {bannerInColor} from "./banner-in-color.js"
```

That's it for the basics. On to the
[next part](../using-jsm-esm-in-nodejs-a-practical-guide-part-2/) of this guide,
where we learn about an important feature of JSM: `exports`.
