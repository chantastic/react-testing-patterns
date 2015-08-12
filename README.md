React Testing Patterns
----------------------

Mostly reasonable patterns for testing React on Rails

Table of Contents
=================

1. [Scope](#scope)
1. [Constraints](#constraints)
1. [A Dirty-UMD](#a-dirty-umd)

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
* ES6 syntax support

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
