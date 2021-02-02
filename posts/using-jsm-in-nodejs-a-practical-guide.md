---
title: "Using JS Modules in Node.js: A Practical Guide"
description: |
  JSM is ready for use in Node.js. This guide shows you how, and how to avoid all the small gotchas.
  The guide covers the basics, but also discusses how to write packages that
  can be dual-mode (JSM and CJS), how to configure ESLint, Mocha, and Testdouble, and how
  to use TypeScript with JSM.
date: 2021-02-01
tags:
  - JSM
  - JavaScript
  - Node.js
layout: layouts/post.njk
---

<!-- markdownlint-disable MD029 -->
<!-- markdownlint-disable MD033 -->

(Hey, if you want to come work with me at Roundforest, and try out JSDoc typing, feel free to
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

> Note: this guide covers **Node.js JSM** and does not cover Browser JSM.

## What do I mean by JSM in Node.js? Don't we have that already?

JSM is the standard JavaScript module system (JSM is a shorcut for
EcmaScript Modules, whereby "EcmaScript" is the official name for the JavaScript language).
This is in constrast to the regular Node.js module system, which is CommonJS (CJS for short). It's
this syntax:

```js
// add.js
export function add(a, b) {
  return a + b;
}

// main.js
import { add } from "./add.js";
```

It was standardized in 2015, but it took awhile for browsers to support this, and it took even longer
for Node.js to support it (the final stable version in Node.js was finalized only in 2020!).
If you want more information, you can see my [talk at Node.TLV](https://www.youtube.com/watch?v=kK_3OP0uJ0Y).
In the talk, at the end, I discuss whether JSM is ready for work, and I say that it's not
quite, and people should start migrating in a year or two. Well, that year is NOW, and it's READY,
and this guide will prepare you for that.

Some of you may be nodding your head and asking themselves: aren't we already using that? Well,
if you are, you're transpiling your code using Babel or TypeScript, which support JSM
out of the box, and transpile it to CJS. The JSM this post is talking about is _native_ JSM
that is supported by Node.js without transpiling. While syntactically it is the same, there
are small differences between it and Babel/TypeScript JSM, differences which are discussed
in my Node.TLV talk above. More importantly, native JSM in Node.js does not need transpiling,
with all the problems that come with it.

## Give it to me straight. Can I start using JSM in Node.js?

Yes. Mostly yes. All the tooling I use supports it, but there are two BIG gotchas that are probably
hard to swallow for some people, gotchas that are hard to workaround:

- Jest support for JSM in Node.js is [experimental](https://jestjs.io/docs/en/ecmascript-modules)
- Jest experimental support does not support yet mocking modules
  (regular function/object mocking is supported)
- `proxyquire` and other popular module mockers do not yet support JSM

The biggest problem is the lack of support in module mockers for JSM. There _is_ one mocking
library that does support JSM, and we use that in this guide.

So can you live with this? If you can, then going all in with JSM in Node.js is now totally
possible. I've been using it for four months now, with zero problems. Actually, it feels
like VSCode support for JSM is much better than for CJS, so I suddenly get auto imports of modules,
which I didn't get before in the CJS world.

## The guide to Node.js JSM

Each section is a separate package/folder in the
[companion repo](https://github.com/giltayar/jsm-in-nodejs-guide). You can skip to a specific section with these links:

1. [The simplest Node.js JSM project](#section-01)
1. [Using the `.js` extension for JSM](#section-02)
1. [The `exports` field](#section-03)
1. [Multiple exports](#section-04)
1. [Dual-mode libraries](#section-05)
1. [Tooling](#section-06)
1. [TypeScript](#section-07)

Let's start!

## <a id="section-01"></a>The simplest Node.js JSM project (`01-simplest-mjs`)

Companion code: <https://github.com/giltayar/jsm-in-nodejs-guide/tree/main/01-simplest-mjs>.

This is the simplest package. Let's start by exploring `package.json`, and the new
`exports` field.

### `main` and `.mjs` (in `package.json`)

```json
{
  "name": "01-simplest-mjs",
  "version": "1.0.0",
  "description": "",
  "main": "src/main.mjs"
}
```

The main entry point is `src/main.mjs`. Why do we use the `.mjs` extension? Because in Node.s
JSM, the `.js` extension is reserved for CJS and `.mjs` means that this is an JS module
(in the next section, we'll see how to change that). We'll takl a bit more about that in the next
part.

Let's continue exploring with `main.mjs`:

### Importing using extensions (`main.mjs`)

```js
// main.mjs
import { bannerInColor } from "./banner-in-color.mjs";

export function banner() {
  return bannerInColor("white");
}
```

Node.js JSM _forces_ you to specify the full relative path to the file, _including the extension_.
The reason they did this is to be compatible with Browser JSM that also forces you do to this. So
don't forget that extension!

Unfortunately, VSCode doesn't like the `.mjs` extension and so Ctrl/Cmd+Clicking it won't work,
and its built in intellisense doesn't work on it.

> **Gotcha**: VSCode doesn't like the `.mjs` extension and ignores files with that extension. In
> the next section we'll see how to deal with that, so it's not a _real_ gotcha.

The `main.mjs` exports the `banner` function, which will be tested in `test/tryout.mjs`. But
first, let's explore `banner-in-color.mjs`:

### Importing JSM and CJS paclages (`banner-in-color.mjs`)

We've seen how we can import JSM modules. Let's see how to import other packages:

```js
// banner-in-color.mjs
import { join } from "path";
import chalk from "chalk";
const { underline } = chalk;
```

We first see that we can import Node.js internal packages like `path`, because Node.js
exposes them as "dual-mode": they are exposed as both CJS and JSM modules.

And if we had an JSM package in NPM, the same could have been used to import the JSM package.
But most of the pacakges NPM has are still CJS packages. As you can see in the second line, where
we import `chalk`, CJS packages can also be imported using `import`. But for the most part,
you can only use "default" importing, and not "named" imports. So if you could do this in CJS:

```js
const { underline } = require("chalk");
```

You _cannot_ do this with CJS packages:

```js
import {underline} from 'chalk`
```

And you can only import the default (non-named) import, and use destructuring later:

```js
import chalk from "chalk";
const { underline } = chalk;
```

Why is this? Because JSM does not allow _executing_ a module to determine what the exports are,
and executing a module is the only reliable way of determining what the exports are.
Node.js actually tries very hard to figure out what the named exports are (using a very
fast parsing of the module), but my experience is that most packages I've tried this with
don't really work, and I need to fall back to default importing.

> **Gotcha**: importing a CJS module is easy, but you can't use named imports and need
> to add a second line to destructure out the named imports.

I believe that in 2021, more and more packages will have JSM entry points that export
themselves as JSM with the proper named exports. But for now, you may need the double lines
to use CJS pacakges.

### Top-level await (in `banner-in-color.mjs`)

Continuning our exploration of `banner-in-color.mjs` we find this extraordinary line
that reads a file from the disk:

```js
// banner-in-color.mjs
const text = await fs.readFile(join(__dirname, "text.txt"), "utf8");
```

Why so extraordinary? Because of that `await`. This is an `await` that is outside an `async`
function, and is at the top level of the code. This kind of `await` is called "top-level await"
and is supported since Node v14. It is extraordinary because it is the only feature in Node.js
that is available _only_ in JSM modules (i.e. not available in CJS). Why is that? Because
JSM is an async module system, and so supports async operations, while CJS is loaded synchronously
and so cannot support `await`.

Great feature, and only in JSM!

But notice the use of `__dirname`. Let's discuss that.

### `__dirname` (in `banner-in-color.mjs`)

If you try to use `__dirname` in JSM, you will find that it is not available
(just like `__filename`). But if you need it, you can quickly find it using these lines:

```js
// banner-in-color.mjs
import url from "url";
const __dirname = url.fileURLToPath(new URL(".", import.meta.url));
```

Let's deconstruct this code. First off, `import.meta.url` is similar to `__filename`, except
that it's a URL and not a file path. Why URL's? Because JSM is defined in terms of URLs and not
file paths.

Now that we have the URL of the current file, we need th parent URL, and we use
`new URL('.', import.meta.url)` to get at it (why this works is out of scope of this guide).
Finally, to get at the file path and not the URL, we need a function that converts between the two
and the internal `url` module gives us this via the `ur.fileURLToPath` function.

Finally, we put it in a variable called `__dirname`, named so out of deference to the Node.js
tradition 😀.

### Testing this module

- To test this module, cd to the directory, and then:

```shell
npm install
npm test
```

The test will run `test/tryout.mjs`, which will `import` the `src/main.mjs`, which will
use (as we saw above) various CJS and JSM imports,
to export a function with the colored banner with the answer
(to life, the universe, and everything) `42`. It will assert that the answer is such,
and `console.log` it so we can see it with all its glory.

Done!

## <a id="section-02"/></a>Using the `.js` extension for JSM (`02-simplest-js`)

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

There is a very simple way to make all your `.js` files be interpreted as JSM and not as CJS:
just add `"type": "module"` to your `package.json`, like above. That's it. From that point on,
all `.js` files will be interpreted as JSM, and it means that your whole code can use the `.js`
extension.

You can still use `.mjs` and it wil always be JSM, _and_ if you want a CJS module in your code,
you can use the new `.cjs` extension (we'll see how we use that in the "dual mode library" section).

That's it. The rest of the code in this director uses `.js`, and when importing, also uses the
`.js` extension:

```js
// src/main.js
import { bannerInColor } from "./banner-in-color.js";
```

On to the next section, where we'll learn about an important feature for JSM:

## <a id="section-03"></a>The `exports` field (`03-exports`)

Let's look at the new directory, `03-exports`, and see this new `exports` field:

### The `exports` field in `package.json`

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

While you can use `main` to define the entry point, the correct JSM way to define the package
entry point is `exports`. Note that in `exports`, the path MUST start with a `.` (for reasons
we'll see in the section on multiple entry points).

Using `exports`, you define the entry point (in the above case `.`), and what that entry point
points to: `./src/main.js`. It looks very verbose, but we'll see why this syntax is this way
shortly.

Why use `exports` if `main` is enough? Because once you define `exports`, then no `import`
can deep-link into the `packages`, i.e. if the package was published, another
package could import it by using

```js
import { banner } from "01-simplest-mjs";
```

But can't do this

```js
import { banner } from "01-simplest-mjs/src/main.mjs";
```

Note that `exports` is not just used in JSM, but is also available in CJS. So if
`exports` is defined in a CJS package, then no deep linking is allowed in CJS too.

### Self-referencing the package in `test/tryout.js`

Another import benefit can be seen in `test/tryout.js` in this package:

```js
import { banner } from "03-exports";
```

This is code from the existing package, yet it can self-reference itself using the name of
the package!

> **Gotcha**: unfortunately, none of the toolings (eslint, typescript, etc) understand that this is
> is a self-reference that is allowed in Node.js and flag this as an error. But you can always
> disable the error on this specific line, as it most definitely works in runtime.

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
> but please do include the `./package.json` escape hatch and also
> continue to include `main` as some tooling (e.g. TypeScript) does not yet understand `export`.

## <a id="section-04"></a>Multiple exports (`04-multiple-exports`)

We've already seen the `exports` field in `package.json`. Let's see what else it can do:

### Multiple entry points in `package.json`

```json
{
  "name": "04-multiple-exports",
  "exports": {
    ".": "./src/main.js",
    "./red": "./src/main-red.js",
    "./blue": "./src/main-blue.js",
    "./package.json": "./package.json"
  }
}
```

In the above, we can see four entry points: the main one (`04-mutiple-exports`),
the colored ones (`04-multiple-exports/red` and `04-multiple-exports/blue`), and the fallback
one (`04-multiple-exports/package.json`).

This is a nice way to define multiple entry points to a package, and as we've already said,
also "hides" the implementation modules from the outside.

Let's see how we test this using the "self-referencing" feature of JSM:

### Self-referencing the multiple entry points in `test/tryout.js`

```js
import { banner as whiteBanner } from "04-multiple-exports";
import { banner as redBanner } from "04-multiple-exports/red";
import { banner as blueBanner } from "04-multiple-exports/blue";
```

As you can see, we can self-reference the entry points in the module itself, and thus
are able to test the module entry points.

> **Gotcha**: (already mentioned above, repeated here for reference)
> unfortunately, none of the toolings (eslint, typescript, etc) understand that this is
> is a self-reference that is allowed in Node.js and flag this as an error. But you can always
> disable the error on this specific line, as it most definitely works in runtime.

## <a id="section-05"></a>Dual-mode libraries (`05-dual-mode-library`)

Up to now, we've created JSM code that was exported as an JSM package. So if another JSM
package `npm install`-ed this one, then it could use it without a problem (using `import`).
But if a CJS package `npm install`-ed it, it would be difficult to use, because we are not allowed
to `require` an JSM package, and the only way to use it would be using `await import(...)`.

Why does this limitation exist? To make it simple: because CJS is a synchronous module system,
and JSM is asyncronous, so the only way to import JSM from CJS is using an asynchronous operation,
`await import(...)`.

But what if we could create a module that can be both `import`-ed and `require`-ed. If it is
`import`-ed, it uses JSM code, and if it is `require`-ed, it uses CJS code.

We can! And again, we use the incredible `exports` field, and its ability to do
"conditional exports". Let's look at the `package.json`:

### Conditional exports in `package.json`

```json
{
  "name": "05-dual-mode-library",
  "exports": {
    ".": {
      "require": "./lib/main.cjs",
      "import": "./src/main.js"
    }
  }
}
```

The main entry point (`.`), instead of being a _path_ to the entry point module, is
an object that has two "conditions": `require` and `import`, and it's those conditions that have
paths to entry point modules. The conditions define _when_ each entry point will be used. If
you're `require`-ing the package, Node.js uses `./lib/main.cjs`, and when you're `import`-ing
it, Node.js uses `./src/main.js`.

Perfect!

Just one small detail. Are we now going to write two implementations of our package, one for
CJS (using `require` everywhere) and one for JSM (using `import` everywhere)? That's asking
a bit too much.

One simple way out of this conundrum, is to write the package using CJS, and then write
a small JSM wrapper. That's a great solution, but this is a guide on how to write JSM code,
so it won't work for this guide. So what do we do?

We do what our ancestors did: we transpile. 😎 For this,
I'm going to use [Rollup](https://rollupjs.org/). Rollup is great because it understands
JSM, understands CJS, and can convert from one to another, so as you'll see, the solution is easy.

### Dev Dependencies needed for transpiling (`package.json`)

```json
  "devDependencies": {
    "cpr": "^3.0.1",
    "rollup": "^2.38.1"
  },
```

We `npm install` both `rollup` and `cpr`, which we're going to use as a cross platform way to
copy our non-js assets (in our case, it's the `text.txt` file used by the code).

Now let's see how to configure Rollup to transpile our code:

### Transpiling in `rollup.config.mjs`

Remember, the code is in `src/*.js` and we want to transpile to `lib`

```js
const srcFiles = await fs.promises.readdir(new URL('./src', import.meta.url))

export default {
  input: srcFiles
    .filter((file) => file.endsWith('.js'))
    .map((x) => `src/${x}`),
  output: {
    dir: 'lib',
    format: 'cjs',
    entryFileNames: '[name].cjs',
    preserveModules: true,
  }
}
```

> **Gotcha**: theoretically, we should be able to name the rollup config file `rollup.config.js`,
  but Rollup assumes that `.js` is CJS, and so forces us to name the file `rollup.config.mjs`.

Let's deconstruct this code. First, the `input`, which is basically the list of `.js` files
in `src`. Now for the output: The output `dir` is `lib`, and we want to format it as `cjs`.
This makes sense.

But why do we want the output filename end with `.cjs`? Because if we ended them with `.js` in this
package, they will be treated as JSM files. So we use the `.cjs` extension to make Node.js treat
them as CJS files.

And what is that `preserveModules: true`? This forcs Rollup to create a module for each module
in the input, instead of trying to chunk them together.

And that's it, we just need to run Rollup, and not to forget to copy the other asserts also
to lib:

### Build script in `package.json`

```json
  "scripts": {
    "build": "rollup -c && cpr src/text.txt lib/ --overwrite",
  },
```

### Testing the code

To test the code, we first `npm run build` to generate the CJS code,
and then run `test/tryout.js`, that executes

```js
import {banner} from '05-dual-mode-library'
//...
```

And then we also run `test/tryout.cjs`, that executes

```js
const {banner} = require('05-dual-mode-library')
//...
```

Each one will find the correct entry point using the conditional export, and all will be well.

OK, cool. We're done with the basics of JSM in Node.js. Now for the tooling: using things
like ESLint, test runners, and mocks. Let's dig deep.

## <a id="section-06"></a>Tooling (`06-tooling`)

All the above directories were "toy" directories. `06-tooling` is a more complete package,
with all the modern necessities needed for JavaScript development. Included in the package is
support for:

- ESLint for linting, including the Node ESLint plugin
- Mocha and Chai for testing
- [Testdouble](https://www.npmjs.com/package/testdouble) for module mocking

> **Gotcha**: as I said in the introduction, all major test runners today support JSM,
  except for Jest that only has experimental support.
> **Gotcha**: the only mocking library that supports mocking JSM modules is currently
  TestDouble.

Let's start exploring the package, tool by tool. We'll start with the most important of them,
ESLint.

### ESlint configuration for Node.js JSM

ESlint already supports JSM well because a lot of code is already using JSM via Babel or TypeScript.
But there is one aspect it doesn't support yet, and that is top-level await. If you're not using
top-level await, then just go ahead and use whatever ESLint configuration you already have. Just
don't forget to use `sourceType: "module"`:

```json
// .eslintrc.json
{
  "parserOptions": {
    "sourceType": "module"
  },
}
```

But if you _are_ using top-level await (and I heartily recommend it!), then ESLint will
say that it cannot parse it. The solution to this, at least until ESLint supports it, is
to use the babel parser with the correct plugin:

```json
// .eslintrc.json
{
  "parser": "@babel/eslint-parser",
  "parserOptions": {
    "sourceType": "module"
  }
}
```

But that is not enough. You need to install the babel parser (with babel core),
and the babel plugin that supports top-level await:

```shell
npm install --save-dev @babel/eslint-parser @babel/core @babel/plugin-syntax-top-level-await
```

And to create a small .babelrc file that points to the plugin:

```json
// .babelrc.json
{
  "plugins": ["@babel/plugin-syntax-top-level-await"]
}
```

That's it! Now when we run `eslint`, it will use the babel parser, which will use the babel
plugin that knows how to parse top-level await. And all is well.

Well, almost. There is a much used plugin for ESlint, the
[Node.js ESLint plugin](https://www.npmjs.com/package/eslint-plugin-node) that supports
linting Node code. And it supports JSM out of the box! And pretty well. It does
currently (February 2021) have a bug in that it doesn't support `await import(...)`, and
the only workaround I could find was to disable the error in the eslint configuration:

```json
// .eslintrc.json
  "rules": {
    "node/no-unsupported-features/es-syntax": [
      "error", {"ignores": ["dynamicImport", "modules"]}
    ]
  }
```

And that's it for ESLint. On to the next tools, Mocha and Chai.

## Mocha and Chai

Mocha was actually the first test runner to support JSM (support written by yours truly 😊),
and it now also supports `--require` that loads JSM code too. So write your test files using JSM
with no problem. The _only_ problem is a minor one, for both Mocha and Chai: they are still
CommonJS packages, and so importing them is a two step process:

```js
// test/test.js
import mocha from 'mocha'
const {describe, it} = mocha
import chai from 'chai'
const {expect} = chai
```

There are pull requests in both projects to add JSM wrappers, so it's just a matter of a couple
of more months till this problem is solved.

## Mocking imports using Testdouble

Currently, the only module mocking library that supports JSM is
[testdouble.js](https://www.npmjs.com/package/testdouble)
(whose support was also written by yours truly 😊), so if you do module mocking
using libraries such as [proxyquire](https://www.npmjs.com/package/proxyquire) then you're out
of luck.

> **Gotcha**: currently, the only library that supports mocking JSM modules is testdouble.
  Note that regular function mocking has no problems in any library, as it is not done with
  modules.

Testdouble (and other mocking libraries in the future) uses a JSM-only feature called
[loaders](https://nodejs.org/api/esm.html#esm_loaders) that enables a module to hook into the
JS module loading process. Unfortunately, for a module to be a loader, it needs to be declared
as such in the command line. Which is why we run Mocha thus:

```sh
mocha --loader=testdouble test/test.js
```

(and get a scary looking "loaders are experimental" kind of a warning. Because, well,
they _are_ experimental!)

Now, to use mocking, you can just:

```js
// test/test.js
td.replaceEsm('../src/add.js', {add: () => 44})
```

And it will replace the `add` function in `add.js` with the above mock. Simple and efficient.

But we haven't covered one of the most important tools to come out of the previous decade:

## <a id="section-07"></a>TypeScript (`07-typescript`)

I gave a talk last year, and said that TypeScript wasn't yet ready for transpiling to Node.js
JSM. Not sure what changed, but it's definitely ready now. Let's first look at the `tsconfig.json`:

```json
// tsconfig.json
{
  "compilerOptions": {
    "lib": ["es2020", "DOM"],
    "target": "ES2020",
    "module": "esnext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    // ...
  },
  //...
}
```

All the above are important to make TypeScript transpile the code and keep the `import` so that
they work with JSM. There's just one gotcha, and it's not even a gotcha, _as this is by design_:
because JSM in Node.js needs to write relative paths with extensions, you must write the extensions
in TypeScript too, _and they have to be `.mjs` or `.js`! So in our code:

```js
// banner-in-color.js
import {add} from './add.js'
```

Even though `add.ts` is a TypeScript file, we still write `./add.js` and not `./add.ts`. This is
by design of TypeScript and is not a bug or anything.

So just remember this weirdness, modify your `tsconfig.json` and start transpiling your TypeScript
code to JSM!

Given that I wrote a whole
[blog post on JSDoc typings](./jsdoc-typings-all-the-benefits-none-of-the-drawbacks),
I would be amiss if I didn't say that JSDoc typings also work well, with the same `tsconfig.json`
(just don't forget to turn on `allowJS` and `checkJS`). You can see an example project that
uses JSDoc typings and JSM in `08-jsdoc-typing`.

## Summary

The trip is over. I've shown how to use JSM in Node.js, how to
configure your projects and tools to support JSM, and covered some of the gotchas around those.
I'm sure you're using tools that I haven't covered.
I'd love to hear about them and if there were any JSM gotchas or tuning to do to support it.
Drop me a Twitter DM at @giltayar: I'd love to add the information to this document.
å