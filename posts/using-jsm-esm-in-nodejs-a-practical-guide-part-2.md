---
title: "Using ES Modules (ESM) in Node.js: A Practical Guide (Part 2)"
description: |
  ESM is ready for use in Node.js.
  This guide shows you how, and how to avoid all the small gotchas.
  The guide covers the basics, but also discusses how to write packages that
  can be dual-mode (ESM and CJS), how to configure ESLint, Mocha, and Testdouble, and how
  to use TypeScript with ESM.
date: 2021-02-06
tags:
  - ESM
  - JavaScript
  - Node.js
layout: layouts/post.njk
---

<!-- markdownlint-disable MD029 -->
<!-- markdownlint-disable MD033 -->

![ESM Logo](/img/jsm-logo.png)

(Hey, if you want to come work with me at Roundforest, and try out ESM on Node.js, feel free to
find me on [LinkedIn](https://www.linkedin.com/in/giltayar/) or on Twitter (@giltayar))

This is part 2 of the guide for using ES modules in Node.js.

- Part 1: the basics of ESM

1. [The simplest Node.js ESM project](../using-jsm-esm-in-nodejs-a-practical-guide-part-1/#section-01)
2. [Using the `.js` extension for ESM](../using-jsm-esm-in-nodejs-a-practical-guide-part-1/#section-02)

- Part 2 (this document): "exports" and its uses (including dual-mode libraries)

3. [The `exports` field](#section-03)
4. [Multiple exports](#section-04)
5. [Dual-mode libraries](#section-05)

- Part 3: Tooling and Typescript

6. [Tooling](../using-jsm-esm-in-nodejs-a-practical-guide-part-3/#section-06)
7. [TypeScript](../using-jsm-esm-in-nodejs-a-practical-guide-part-3/#section-07)

This guide comes with a monorepo that has 7 directories, each directory being a package
that demonstrates the above sections. You can find the monorepo
[here](https://github.com/giltayar/jsm-in-nodejs-guide).

On to the next section, where we'll start looking at a new feature in Node.js: exports.

## <a id="section-03"></a>The `exports` field

Companion code: <https://github.com/giltayar/jsm-in-nodejs-guide/tree/main/03-exports>

The `exports` field is a new field in `package.json` which controls what entry points a package
has. Let's look at the simplest case.

### The `exports` field

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/03-exports/package.json>

```json
{
  // package.json
  "name": "03-exports",
  "type": "module",
  "main": "src/main.js",
  "exports": {
    ".": "./src/main.js"
  }
```

The entry point to the package, which is the file that loads when your `import` or `require`
a package, is `src/main.js` (just like in the previous parts of the guide), and is usually
defined by the field `main`.

While you can use `main` to define the entry point, the correct ESM way to define the package
entry point is a new field: `exports`. Note that in `exports`, the path MUST start with a "`.`".
This makes sense, as it is the same path you would use to `import` the file. If you're using
`main`, you don't need the "`.`", but the exports field mandates it.

Using `exports`, you define the entry point (in the above case "`.`"), and what that entry point
points to: `./src/main.js`. It looks very verbose, but we'll see why this syntax is this way
shortly.

Why use `exports` if `main` is enough? Looks the same, right? Well, it's close, but if you use
`exports`, then **no `import` can deep-link into the package**,
i.e. if the package was published, another package could import it by using

```js
import {banner} from "01-simplest-mjs"
```

But wouldn't be able to do this (they'll get an error)

```js
import {banner} from "01-simplest-mjs/src/main.mjs"
```

An interesting thing to note about `exports`, is that this is actually not a ESM feature.
It is a Node.js feature and is also available for CJS! So `exports` works in CJS packages,
and if `exports` is defined in a CJS package, then no deep linking is allowed in CJS too.

### Self-referencing the package

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/03-exports/test/tryout.js>

Another importan benefit can be seen in `test/tryout.js`.

```js
// test/tryout.js
import {banner} from '03-exports'
```

This is code from the existing package, yet it can self-reference itself using the name of
the package, instead of using `../src/main.js`.

> **Gotcha**: unfortunately, none of the toolings (eslint, typescript, etc) understand that this is
  is a self-reference that is allowed in Node.js and flag this as an error. But you can always
  disable the error on this specific line, as it most definitely works in runtime.

So `exports` gives us a nice way of "hiding" our inner modules and not exposing them outside the
package. This is great, but has a nasty side-effect: some tools need access to all the
packages `package.json`, and using `exports` bars them from accessing it. To enable
them to access the `package.json`, we add another line to the `package.json` "exports":

```json
  // package.json
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

## <a id="section-04"></a>Multiple exports

Companion code: <https://github.com/giltayar/jsm-in-nodejs-guide/tree/main/04-multiple-exports>

We've already seen the `exports` field in `package.json`. Let's see what else it can do:

### Multiple entry points

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/04-multiple-exports/package.json>

```json
{
  // package.json
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
the red and blue ones (`04-multiple-exports/red` and `04-multiple-exports/blue`),
and the "package.json" one (`04-multiple-exports/package.json`).
This is a nice way to define multiple entry points to a package, and as we've already seen,
still "hides" the other implementation modules from the outside.

Let's see how we test this using the "self-referencing" feature of ESM:

### Self-referencing the multiple entry points

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/04-multiple-exports/test/tryout.js>

```js
// test/trout.js
import {banner as whiteBanner} from "04-multiple-exports"
import {banner as redBanner} from "04-multiple-exports/red"
import {banner as blueBanner} from "04-multiple-exports/blue"
```

As you can see, we can self-reference the entry points in the module itself, and thus
are able to test the module entry points.

> **Gotcha**: (already mentioned above, repeated here for reference)
  unfortunately, none of the toolings (eslint, typescript, etc) understand that this is
  is a self-reference that is allowed in Node.js and flag this as an error. But you can always
  disable the error on this specific line, as it most definitely works in runtime.

## <a id="section-05"></a>Dual-mode libraries

Companion code: <https://github.com/giltayar/jsm-in-nodejs-guide/tree/main/05-dual-mode-library>

Up to now, we created ESM code that was exported as a ESM package. So if another ESM
package "`npm install`"-ed this one, then it could use it without a problem (using "`import`").
But if a CJS package "`npm install`"-ed it, it would be difficult to use, because we are not allowed
to "`require`" a ESM package, and the only way to use it would be using "`await import(...)`",
which is problematic because it is an async operation, which can only be used in an async function.

Why does this limitation exist? Because CJS is a synchronous module system,
and ESM is asyncronous, the only way to import ESM from CJS is using an asynchronous operation,
"`await import(...)`".

But what if we could create a module that can be both `import`-ed and `require`-ed? If it is
"`import`"-ed, it uses ESM code, and if it is "`require`"-ed, it uses CJS code.

We can! And again, we use the incredible `exports` field, and its ability to do
"conditional exports". Let's look at the `package.json` of this package.

### Conditional exports in `package.json`

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/05-dual-mode-library/package.json>

```json
// package.json
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

Perfect for our needs! We just point the two entry points to different implementations: one
written for ESM and one for CJS.

Just one small detail... Are we now going to write two implementations of our package, one for
CJS (using `require` everywhere) and one for ESM (using `import` everywhere)? That's asking
a bit too much, I think.

One simple way out of this conundrum, is to write the package using CJS, and then write
a small ESM wrapper that "`require`"-s the code end "`export`"-s it as ESM.
That's a great solution, and works well for existing packages that are
CJS: they can wrap their code with a ESM wrapper, define the conditions like above, and voila:
they're dual-mode! But if we're using ESM, this is not an option for us.
So is there a way to continue writing ESM code, and yet still have a parallel CJS entry point?

Yes, there is. We do what our ancestors did: we transpile. 😎 For this,
I'm going to use [Rollup](https://rollupjs.org/). Rollup is great because it understands
ESM, understands CJS, and can convert from one to another, so we don't have to do a lot
to get Rollup to take all our code and just transpile it from ESM to CJS.

### Dev Dependencies needed for transpiling

We'll need rollup, so we `npm install` it. We'll also `npm install` the `cpr` package,
to use as a cross platform way to copy our non-js assets
(in our case, it's the `text.txt` file used by the code).

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/05-dual-mode-library/package.json>

```json
  // package.json
  "devDependencies": {
    "cpr": "^3.0.1",
    "rollup": "^2.38.1"
  },
```

Now let's see how we configure Rollup to transpile our code.

### Transpiling ESM to CJS using Rollup

Remember, the code is in `src/*.js` and we want to transpile to CJS code in `lib/*.js`.

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/05-dual-mode-library/rollup.config.mjs>

```js
// rollup.config.mjs
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
  because it uses ESM. but Rollup assumes that `.js` is CJS,
  and so forces us to name the file `rollup.config.mjs`.

Let's deconstruct this code. First, the `input`, which is the list of `.js` files
in `src`.

Now for the output: The output `dir` is `lib`, and we want to format it as `cjs`. This makes sense.

But why do we want the output filenames to end with `.cjs`? Because if we ended them with `.js`,
they will be treated as ESM files. So we use the `.cjs` extension to force Node.js to treat
them as CJS files.

And what is that `preserveModules: true`? This forcs Rollup to create a module for each module
in the input, instead of trying to chunk them together.

And that's it, we just need to run Rollup, and not to forget to copy the other assets also
to lib.

### Build script

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/05-dual-mode-library/package.json>

```json
  // package.json
  "scripts": {
    "build": "rollup -c && cpr src/text.txt lib/ --overwrite",
  },
```

### Testing the code

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/05-dual-mode-library/test/tryout.js>

And: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/05-dual-mode-library/test/tryout.cjs>

To test the code, we first `npm run build` to generate the CJS code,
and then run `test/tryout.js`, that executes

```js
// test/tryout.js
import {banner} from '05-dual-mode-library'
//...
```

And then we also run `test/tryout.cjs`, that executes

```js
// test/tryout.cjs
const {banner} = require('05-dual-mode-library')
//...
```

Each one will find the correct entry point using the conditional export, and all will be well.

OK, cool. We're done with the basics of ESM in Node.js. Now for the tooling: using things
like ESLint, test runners, and mocks. You can find that in the next
part, [here](../using-jsm-esm-in-nodejs-a-practical-guide-part-3/).
