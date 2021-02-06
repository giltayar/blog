---
title: "Using JS Modules (ESM) in Node.js: A Practical Guide (Part 3)"
description: |
  JSM (formerly ESM) is ready for use in Node.js.
  This guide shows you how, and how to avoid all the small gotchas.
  The guide covers the basics, but also discusses how to write packages that
  can be dual-mode (JSM and CJS), how to configure ESLint, Mocha, and Testdouble, and how
  to use TypeScript with JSM.
date: 2021-02-07
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

This is part 3 of the guide for using JS modules in Node.js.

- Part 1: the basics of JSM

1. [The simplest Node.js JSM project](#section-01)
2. [Using the `.js` extension for JSM](#section-02)

- Part 2: "exports" and its uses (including dual-mode libraries)

3. [The `exports` field](../using-jsm-esm-in-nodejs-a-practical-guide-part-2/#section-03)
4. [Multiple exports](../using-jsm-esm-in-nodejs-a-practical-guide-part-2/#section-04)
5. [Dual-mode libraries](../using-jsm-esm-in-nodejs-a-practical-guide-part-2/#section-05)

- Part 3 (this document): Tooling and Typescript

6. [Tooling](../using-jsm-esm-in-nodejs-a-practical-guide-part-3/#section-06)
7. [TypeScript](../using-jsm-esm-in-nodejs-a-practical-guide-part-3/#section-07)

This guide comes with a monorepo that has 7 directories, each directory being a package
that demonstrates each facet of the Node.js support for JSM. You can find the monorepo
[here](https://github.com/giltayar/jsm-in-nodejs-guide).

In this part we see how to integrate tools like ESlint, Mocha/Chai, Testdouble,
and TypeScript.

## <a id="section-06"></a>Tooling

Companion code: <https://github.com/giltayar/jsm-in-nodejs-guide/tree/main/06-tooling>

All the above directories were "toy" directories. `06-tooling` is a more complete package,
with all the modern necessities needed for JavaScript development. Included in the package is
support for:

- ESLint for linting, including the Node ESLint plugin
- Mocha and Chai for testing
- [Testdouble](https://www.npmjs.com/package/testdouble) for module mocking

> **Gotcha**: as I said in the introduction, all major test runners today support JSM,
  except for Jest that only has experimental support.
  **Gotcha**: the only mocking library that supports mocking JSM modules is currently
  TestDouble.

Let's start exploring the package, tool by tool. We'll start with the most important of them,
ESLint.

### ESlint configuration for Node.js JSM

ESlint already supports JSM well because a lot of code is already using JSM via Babel or TypeScript.
But there is one aspect it doesn't yet support, and that is top-level await. If you're not using
top-level await, then just go ahead and use whatever ESLint configuration you already have. Just
don't forget to use `sourceType: "module"`.

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/06-tooling/.eslintrc.json>

```json
// .eslintrc.json
{
  "parserOptions": {
    "sourceType": "module"
  }
}
```

But if you _are_ using top-level await (and I heartily recommend it!), then ESLint will
complain that it cannot parse it. The solution to this, at least until ESLint supports it, is
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

And to create a small .babelrc file that points to the plugin.

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/06-tooling/.babelrc.json>

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
linting Node code. And it supports JSM out of the box! And pretty well. Unfortunately, it currently
(February 2021) has a bug where it thinks Node.js doesn't support `await import(...)`, and
the only workaround I could find was to disable the error in the eslint configuration:

```json
// .eslintrc.json
  "rules": {
    "node/no-unsupported-features/es-syntax": [
      "error", {"ignores": ["dynamicImport", "modules"]}
    ]
  }
```

And that's it for ESLint. Simple, right? Well, a year ago, it was MUCH more complicated than that,
which shows you that things _are_ progressing! On to the next tools, Mocha and Chai.

## Mocha and Chai

Mocha was actually the first test runner to support JSM (support written by yours truly 😊),
and it now also supports `--require` that loads JSM code too. So write your test files using JSM
with no problem. The _only_ problem is a minor one, for Mocha: it is a
CommonJS packages, and has no JSM wrapper, and so importing it is a two step process.

```js
// test/test.js
import mocha from "mocha"
const {describe, it} = mocha
import {expect} from "chai"
```

Funnily enough, this was also true for Chai, but a new version that dealt with this was
released _while I was writing this guide_. There's a pull request in Mocha to add a JSM wrapper,
so it's just a matter of time till this problem is als solved for Mocha too.

Moreover, in terms of JSM support, I've seen more and more packages adding JSM wrappers to enable
named imports, and it feels like the community is rallying around support for JSM in Node.js

## Mocking imports using Testdouble

What do I mean by "mocking imports"? It's the ability to mock a module so that when you `import`
it, you don't get the real module, but rather a mock of the module.

The only module mocking library that currently supports JSM is
[testdouble.js](https://www.npmjs.com/package/testdouble)
(whose support was also written by yours truly 😊), so if you do module mocking
using libraries such as [proxyquire](https://www.npmjs.com/package/proxyquire) then you're out
of luck.

> **Gotcha**: currently, the only library that supports mocking JSM modules is Testdouble.
  Note that regular function mocking has no problems in any library, as it is not done with
  modules, so you can use regular mocking library (like Sinon) in JSM without any problem.

Testdouble, and other mocking libraries in the future, use a JSM-only feature called
[loaders](https://nodejs.org/api/esm.html#esm_loaders) that enable a module to hook into the
JS module loading process. Unfortunately, for a module to be a loader, it needs to be declared
as such in the command line. Which is why we run Mocha thus:

```shell
mocha --loader=testdouble test/test.js ...
```

(and get a scary looking "loaders are experimental" kind of a warning. Because, well,
they _are_ experimental and not yet stable! Fortunately, Testdouble shields you from the instability
of the API, and will probably support any changes in the loader api that are forthcoming.)

Now, to use mocking, you can just use the appropriate Testdouble functin, `td.replaceEsm`.

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/06-tooling/test/test.js>

```js
// test/test.js
td.replaceEsm("../src/add.js", { add: () => 44 })
```

And it will replace the `add` function in `add.js` with the above mock. Simple and efficient.

That's it for tooling. There's just one more (very important!) tool missing in our toolbox, but
that is covered in the next section.

## <a id="section-07"></a>TypeScript

Companion code: <https://github.com/giltayar/jsm-in-nodejs-guide/tree/main/07-typescript>

I gave a talk last year, and said that TypeScript wasn't yet ready for transpiling to Node.js
JSM. Not sure what changed, but it's definitely ready now. Let's first look at the `tsconfig.json`.

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/07-typescript/tsconfig.json>

```json
// tsconfig.json
{
  "compilerOptions": {
    "lib": ["es2020", "DOM"],
    "target": "ES2020",
    "module": "esnext",
    "moduleResolution": "node",
    "esModuleInterop": true
    // ...
  }
  //...
}
```

All the above options are nexessary to make TypeScript transpile the code
and keep the `import` so that they work with JSM.
There's just one gotcha, and it's not even a gotcha, _as this is by design_:
because JSM in Node.js needs to write relative paths with extensions, you must write the extensions
in TypeScript too, _and they have to be `.mjs` or `.js`_! Let's look at how this looks in the code.

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/07-typescript/src/banner-in-color.ts>

```js
// src/anner-in-color.ts
import {add} from "./add.js"
```

Even though `add.ts` is a TypeScript file, we still write `./add.js` and not `./add.ts`. This is
by design of TypeScript and is not a bug. Weird, though, right?

So just remember this weirdness, modify your `tsconfig.json` and start transpiling your TypeScript
code to JSM! 🎉🎉🎉🎉

Just don't forget that the main entry point is now in the `lib` directory

Code: <https://github.com/giltayar/jsm-in-nodejs-guide/blob/main/07-typescript/package.json>

```json
// package.json
{
  "main": "./lib/main.js",
  "exports": "./lib/main.js",
}
```

### Bonus: using JSDoc typings with JSM

Given that I wrote a whole
[blog post on JSDoc typings](./jsdoc-typings-all-the-benefits-none-of-the-drawbacks),
I would be amiss if I didn't say that JSDoc typings also work well, with the same `tsconfig.json`
(just don't forget to turn on `allowJS` and `checkJS`). You can see an example project that
uses JSDoc typings and JSM in
["08-jsdoc-typing"](https://github.com/giltayar/jsm-in-nodejs-guide/tree/main/08-jsdoc-typing).

## Summary

This guide is over. I've shown:

- How to use JSM in Node.js
- How to configure your projects and tools to support JSM
- Some of the gotchas around this

Hope you enjoyed this guide!

By the way, I'm sure you're using tools that I haven't covered.
I'd love to hear about them and if there were any JSM gotchas or tuning to do to support it.
Drop me a Twitter DM at @giltayar: I'd love to add the information to this document.
