React Testing Patterns
----------------------

Mostly reasonable patterns for testing React on Rails

Table of Contents
=================

1. [Scope](#scope)
1. [Constraints](#constraints)
1. [A Dirty-UMD](#a-dirty-umd)
1. [Mocha vs Jasmine vs Jest](#mocha-vs-jasmine-vs-jest)
1. [A Mocha Setup](#a-mocha-setup)
1. [Assertion Libraries](#assertion-libraries)
1. [Mock a Doc](#mock-a-doc)
1. [A Test Boilerplate](#a-test-boilerplate)
1. [React Shallow Rendering](#react-shallow-rendering)

Scope
=====

This is how we write tests for [React.js](http://reactjs.org/) on [Rails](http://rubyonrails.org/). We've struggled to find the happy path. This is our ongoing attempt to carve out the most direct path to testing React components on a golden-path Rails app. Recommendations here represent a good number of failed attempts. If something seems out of place, it probably is; let us know what you've found.

Constraints
===========

Our approach has the following constraints.

* Scripts work in the Asset Pipeline
* Scripts work in a JS-testing framework
* Module unit tests should work without a DOM
* Testing envirnoment should be flexible for app/team-specific needs
* ES2015 syntax support

A Dirty-UMD
===========

Component definitions will need to work in `window` and as a module. This is ugly but it works.

```javascript
(function (global) {
  "use strict";

  let React;

  if (typeof module === "object" && module.exports) {
    React = require("react");
  } else {
    React = global.React;
  }

  class MyWidget extends React.Component {
    render () {
      return <div />
    }
  }

  if (typeof module === "object" && module.exports) {
    module.exports = MyWidget;
  } else {
    global.MyWidget = MyWidget;
  }
})(this);
```

*Feel free to DRY this out however feel right to you.*

Mocha vs Jasmine vs Jest
========================

Here's how it shook out against our particular constraints.

### TL;DR

In 2015, use Mocha. That might change if these Jest issue get resolved:

* Speedy ES2015 support
* Jasmine updated to 2.x

### Jest

* Not as flexible as Mocha for app-specific needs
* Ships with DOM implementation, `jsdom`
  = locked to unsuportted `3.x` tag
* Slow. A test suite of only 4 tests and 23 assertions took 10.7 seconds.
  - https://github.com/facebook/jest/issues/116
* Locked to Jasmine 1.3

### Jasmine

* No obvious Babel/ES2015 path
  - https://babeljs.io/docs/setup/ has very obvious setups for the other frameworks.

### Mocha

* Flexible for teams
  - Very few opinions about anything
  - Well documented
  - Simple to configure
* No default DOM implementation
* Fast. A test suite of only 4 tests and 23 assertions took 1.2 seconds.

#### Other considerations

Jest has very nice feature for auto-mocking and running tests in parallel. But, for now, it's an order of magnitude slower than Mocha (given our constraints).

A Mocha Setup
=============

Init `package.json`, if you havent done so:

```bash
$ npm init
```

Install Mocha:

```bash
$ npm install mocha --save-dev
```

Create the `test/` directory, if one doesn't exist:

```bash
$ mkdir `test/`
```

*mocha will automatically run tests matching `./test/*.js`. [ref](http://mochajs.org/#the-test-directory)

Configure the `npm test` command:

```json
{
  "scripts": {
    "test": "mocha"
  }
}
```

Now you can run your local version of `mocha` via the `npm test` command.

Additionally, you can run the local version of `mocha` with flags via the `node_modules` directory:

```bash
$ node_modules/.bin/mocha --watch --compilers js:babel/register --recursive --reporter nyan
```

Add a `test/mocha.opts` for shared options:

```bash
--watch
--compilers js:babel/register
--recursive
--reporter nyan
```

*This configuration will be used in both `npm test` and `node_modules/.bin/mocha`*

Assertion Libraries
===================

People like what they like.

My $.02. If you don't have a PM that pretends to read specs, write assertions. Just sayin'.

### node/assert

```bash
$ # you already have node/assert. lucky you
```

```js
import assert from "assert";

assert(result.type, "div");
```

[API](https://nodejs.org/api/assert.html)

### expect.js

```bash
$ npm install expect.js --save-dev
```

```js
import expect from "expect.js";

expect(result.type).to.be("div");
```

[API](https://github.com/Automattic/expect.js#api)

### should

```bash
$ npm install should --save-dev
```

```js
import expect from "should";

(result.type).should.be.exactly("div");
```

[API](http://shouldjs.github.io/)

### chai

```bash
$ npm install chai --save-dev
```

Chai has 3 included assertion libraries. Chose your favorite.

```js
import chai from "chai";

const assert = chai.assert;
const expect = chai.expect;
const should = chai.should;

assert.strictEqual(result.type, "div");
expect(result.type).should.equal("div");
result.type.should.equal("div");
```

[API](http://chaijs.com/api/)
Mock a Doc
=============

One of the criteria is testing without a DOM. That sholud be possible using [React shallow rendering]() but [there's a bug](https://github.com/facebook/react/issues/4019).

You can bypass it by mocking `document`.

```javascript
// ./test/utils/document.js

if (typeof document === 'undefined') {
  global.document = {};
}
```

Add this flag to your `mocha.opts`:

```bash
--require test/utils/document.js
```

Should you want a full fledged DOM, follow [this guide](http://jaketrent.com/post/testing-react-with-jsdom/) by [Jake Trent](http://jaketrent.com/).

A Test Boilerplate
==================

Here's what a standard test looks like.

```js
"use strict";

import assert from "assert";

import React, { addons } from "react/addons";
import AppIcon from "../AppIcon.js.jsx";

let shallowRenderer = React.addons.TestUtils.createRenderer();

describe("MyWidget", () => {
  shallowRenderer.render(<MyWidget />);
  let result = shallowRenderer.getRenderOutput();

  it("renders an div tag as its root element", () => {
    assert.strictEqual(result.type, "div");
  });
});
```

You can required your assertion library in `mocha.opts` to avoid requiring in each test.

```bash
--require assert
```

React Shallow Rendering
=======================

Where reasonable, use [shallow-rendering](https://facebook.github.io/react/docs/test-utils.html#shallow-rendering). Shollow rendering does not Require a DOM.

The [ReactTestUtils-test.js](https://github.com/facebook/react/blob/7deab28347eaf206217ffee8e4475a1935b02cb8/src/test/__tests__/ReactTestUtils-test.js#L32-L174) file are the best documentation on how shallow rendering works.

Here are the basics:

### Create a Renderer

```js
import { addons } from "react/addons";

let shallowRenderer = React.addons.TestUtils.createRenderer();
```

### Render and get Result

```js
shallowRenderer.render(<MyWidget />);
let result = shallowRenderer.getRenderOutput();
```

### Asserting type

```js
assert.strictEqual(result.type, "div");
```

### Assert children

```js
assert.deepEqual(result.props.children, [
  <div>hi!</div>,
  <div>okay, bye.</div>
]);
```

### Assert update

```js
shallowRenderer.render(<InterfacesIcon name="" hoverStyle={{ color: "blue" }} />);
result = shallowRenderer.getRenderOutput();

assert.deepEqual(result.props.style, {});

result.props.onMouseEnter();
let mouseEnterResult = shallowRenderer.getRenderOutput();

assert.deepEqual(mouseEnterResult.props.style, { color: "blue" });
```
