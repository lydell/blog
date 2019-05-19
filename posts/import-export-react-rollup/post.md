# Import/export, React and Rollup

_[🔗 Link to a gist containing the files used in this post.](https://gist.github.com/lydell/588591ccee7fed1341247bfae7a6c5b5)_

You know how you can import most React stuff two ways?

```js
import React from "react";

function MyComponent() {
  const [count, setCount] = React.useState(0);
  // ...
}
```

Or:

```js
import { useState } from "react";

function MyComponent() {
  const [count, setCount] = useState(0);
  // ...
}
```

There’s kind of a third way as well:

```js
import * as React from "react";

function MyComponent() {
  const [count, setCount] = React.useState(0);
  // ...
}
```

What’s the difference?

The short answer: In the case of React as of today: No difference at all.

The long answer: The devil is in the details.

First, let’s review how `import`/`export` works. In a nutshell, if you `export` something in one file, you can `import` that thing in another file.

```js
// constants.js
export const NAV_HEIGHT = 80;
```

```js
// main.js
import { NAV_HEIGHT } from "./constants.js";
```

Then there’s this thing called “default exports”. Every file can have one one default export if it wants to. There’s basically two ways of defining them:

```js
// add.js
export default function add(a, b) {
  return a + b;
}
```

Or:

```js
// subtract.js
function subtract(a, b) {
  return a - b;
}
export { subtract as default };
```

There are two ways of importing default exports as well:

```js
// main.js
import add from "./add.js";
import { default as subtract } from "./subtract.js";
```

It doesn’t matter how you defined you default export – you can choose either import style:

```js
// main.js
import { default as add } from "./add.js";
import subtract from "./subtract.js";
```

Also note that you can choose other names for default imports if you want:

```js
// main.js
import { default as plus } from "./add.js";
import minus from "./subtract.js";
```

Alright, with that out of our way we can think about what React could look like.

```js
// Support `import { useState } from "react"`;
export function useState() {
  // whatever
}

const React = {
  useState
  // and lots of other things
};

// Support `import React from "react"; React.useState`.
export default React;
```

Oh – I forgot about `import * as React from "react"`. It imports _all_ exports from a file into an object. With the above implementation of React you’d get this:

```js
import * as React from "react";

console.log(React);
// {
//   useState: function useState() {},
//   default: {
//     useState: function useState() {},
//     // and lots of other things
//   }
// }
```

Let’s have a look at what’s actually in the `react` npm package. Check it out:

https://unpkg.com/react@16.8.6/cjs/react.development.js

Search for `export` in that file. What do you find? Three matches in comments, and this:

```js
module.exports = react;
```

What’s this? It’s Node.js’ style of exporting stuff from files. This is also sometimes called CommonJS. (There are some differences between the Node.js and CommonJS systems, but let’s not get into that.)

Node.js’ module system has been around for much longer than `import`/`export` so it’s still in a lot of packages on npm.

So how could `import React, { useState } from "react"` possibly work? If you play around with `import`/`export` in browsers you’ll see that it doesn’t.

```js
// test.js
// No `export` here!
```

```html
<!DOCTYPE html>
<meta charset="utf-8" />
<title>import/export test</title>
<script type="module">
  import whatever from "./test.js";
  // In the console:
  // SyntaxError: import not found: default
</script>
```

The reason `import React, { useState } from "react"` usually works anyway is because of [webpack](https://webpack.js.org/). I like to think of it as if webpack rewrites `module.exports` to `export` syntax, allowing you to `import` from it. But just how does it do the rewriting?

A very simple way would be:

```js
export default module.exports;
```

That very closely resembles how Node.js’ module system works: You can only export _one_ value – whatever you set `module.exports` to. But this means that there won’t be any named exports. So bundlers started doing this:

```js
export default module.exports;
const { named1, named2, named3 } = module.exports;
export { named1, named2, named3 };
```

Because in the Node.js world it is a common pattern to set `module.exports` to a function (the main export), and then add some extra properties to that function (secondary exports). For example, [express](https://expressjs.com/) does this.

```js
function express() {
  // creates you express server
}

function static() {
  // helper that serves static files
}

express.static = static;
module.exports = express;
```

```js
// your-server.js
const express = require("express");

const app = express();
app.use(express.static("public"));
```

If you use a bundler or [Babel](https://babeljs.io/) for your server-side code, you can now do:

```js
import express, { static } from "express";

const app = express();
app.use(static("public"));
```

However, since bundlers and Babel added support for `import`/`export` before browsers did, lots of people learned `import`/`export` from using their bundler, and got the impression that named exports _always_ work this way.

```js
// constants_node.js
module.exports = {
  NAV_HEIGHT: 80,
  TIMEOUT: 2500
};
```

```js
// constants_broken.js
export default {
  NAV_HEIGHT: 80,
  TIMEOUT: 2500
};
```

```js
// constants_correct.js
export const NAV_HEIGHT = 80;
export const TIMEOUT = 2500;
```

```js
// main.js

// Works because of your bundler:
import { NAV_HEIGHT, TIMEOUT } from "./constants_node.js";

// Error: `{ NAV_HEIGHT, TIMEOUT }` does not destructure the default export!
// It looks for named exports called `NAV_HEIGHT` and `TIMEOUT`, which don’t exist.
import { NAV_HEIGHT, TIMEOUT } from "./constants_broken.js";

// This is OK:
import constants from "./constants_broken.js";
// This (confusingly?) works too:
import constants from "./constants_node.js";

// This works because constants_correct.js does define these exports:
import { NAV_HEIGHT, TIMEOUT } from "./constants_correct.js";

// But this does not work because constants_correct.js does not have a default export.
import constants from "./constants_correct.js";

// Finally, you can import all the exports as an object if you want to:
import * as constants from "./constants_correct.js";
```

To avoid this confusion, there is another way to think of how `module.exports` should be turned into `export` syntax. The idea is that `module.exports` is the object you get (`all`) when you do `import * as all from "something"` (or `const all = await import("something")`). This way you do get named exports as well.

```js
// This is exactly the same as before, but with `.default` added here:
export default module.exports.default;
const { named1, named2, named3 } = module.exports;
export { named1, named2, named3 };
```

However, Node.js packages typically don’t expose anything called `default`. But newer packages are sometimes written with `import`/`export` syntax and transpiled to `module.exports` before shipping to npm. For example, express could have been written like this:

```js
export default function express() {
  // whatever
}

export function static() {
  // whatever
}
```

Before shipping to npm, it could be transpiled to:

```js
module.exports = {
  default: function express() {
    // whatever
  },
  static: function static() {
    // whatever
  }
};
```

So how do bundlers know what to do when you write `import express from "express"`? Should `express` be set to `module.exports.default` (as in the above example), or to `module.exports` (as in the original example, where `module.exports.default` would be undefined)? The solution to this problem is the (non-standard) `__esModule` property. The above example would actually be transpiled to this:

```js
module.exports = {
  __esModule: true,
  default: function express() {
    // whatever
  },
  static: function static() {
    // whatever
  }
};
```

When you then do `import express from "express"`, bundlers inject a little piece of code to determine what to pick. It essentially works like this:

```js
var express = obj.__esModule ? obj.default : obj;
```

[Babel’s `noInterop` documentation](https://babeljs.io/docs/en/babel-plugin-transform-modules-commonjs#nointerop) phrases it well:

> [an] `__esModule` property is exported. This property is then used to determine if the import _is_ the default export or if it _contains_ the default export.

So, what has all of this to do with React? Well, as shown earlier, the code that is shipped to npm contains:

```js
module.exports = react;
```

And if you look where the `react` variable comes from, it is essentially set to this object:

```js
var React = {
  Children: {
    map: mapChildren,
    forEach: forEachChildren,
    count: countChildren,
    toArray: toArray,
    only: onlyChild
  },

  createRef: createRef,
  Component: Component,
  PureComponent: PureComponent,

  createContext: createContext,
  forwardRef: forwardRef,
  lazy: lazy,
  memo: memo,

  useCallback: useCallback,
  useContext: useContext,
  useEffect: useEffect,
  useImperativeHandle: useImperativeHandle,
  useDebugValue: useDebugValue,
  useLayoutEffect: useLayoutEffect,
  useMemo: useMemo,
  useReducer: useReducer,
  useRef: useRef,
  useState: useState,

  Fragment: REACT_FRAGMENT_TYPE,
  StrictMode: REACT_STRICT_MODE_TYPE,
  Suspense: REACT_SUSPENSE_TYPE,

  createElement: createElementWithValidation,
  cloneElement: cloneElementWithValidation,
  createFactory: createFactoryWithValidation,
  isValidElement: isValidElement,

  version: ReactVersion,

  unstable_ConcurrentMode: REACT_CONCURRENT_MODE_TYPE,
  unstable_Profiler: REACT_PROFILER_TYPE,

  __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED: ReactSharedInternals
};
```

And that object does _not_ have `__esModule: true`. So when you write this:

```js
import React, { useState } from "react";
```

Your bundler effectively turns it into:

```js
import React from "react";
const { useState } = React;
```

(Remember: `{ useState }` in the `import` line does _not_ mean destructuring of the default export in _standard_ JavaScript – only in bundlers with Node.js-support world!).

That’s why the short answer to my initial question (what’s the difference between `import React from "react"; React.useState` and `import { useState } from "react"`) is:

> In the case of React as of today: No difference at all.

But there is an interesting quirk to it. After all of this talk about `import`/`export` and Node.js/CommonJS interoperability I can finally come to the thing that made me think about this recently.

[Kent C. Dodds](https://github.com/kentcdodds/) recently [tweeted](https://twitter.com/kentcdodds/status/1128444908887465984) about a nice little React library he had made: [stop-runaway-react-effects](https://github.com/kentcdodds/stop-runaway-react-effects). It monkey-patches React’s `useEffect` (and `useLayoutEffect`) with versions that warn you about accidental infintie loops. Basically, the library does this:

```js
import React from "react";

const originalUseEffect = React.useEffect;

React.useEffect = (callback, deps) => {
  return originalUseEffect(() => {
    // do stuff to detect infinte loops
    return callback();
  }, deps);
};
```

So the next time you run this:

```js
React.useEffect(() => {
  // whatever
}, []);
```

The infinite loop detection is automatically run for you. But what if you prefer importing `useEffect` separately?

```js
import { useEffect } from "react";

useEffect(() => {
  // whatever
}, []);
```

As we’ve learned above, webpack basically does this behind the scenes (in the case of React):

```js
import React from "react";
const { useEffect } = React;

useEffect(() => {
  // whatever
}, []);
```

And if you’ve run the monkey-patch before that, the `React` object has already been mutated, explaining why it works for this import style too.

But. And here’s the big but. This relies on your bundler actually running `const { useEffect } = React;` _after_ the monkey-patching has run. webpack does so (check its compiled output!). But there’s nothing saying it _has_ to. [Rollup](https://rollupjs.org/) is an example of this. Let’s see how it compiles stuff.

```js
// main.js
import "./monkey-patch.js";
import React, { useEffect } from "react";
import ReactDOM from "react-dom";

function App() {
  useEffect(() => {
    console.log("My useEffect code is running.");
  }, []);
  return "test";
}

ReactDOM.render(React.createElement(App), document.getElementById("root"));
```

```js
// monkey-patch.js
import React from "react";

const originalUseEffect = React.useEffect;

React.useEffect = (callback, deps) => {
  return originalUseEffect(() => {
    console.log("Monkey!");
    return callback();
  }, deps);
};
```

```
❯ echo '{"private": true}' > package.json

❯ npm i rollup react react-dom
npm notice created a lockfile as package-lock.json. You should commit this file.
+ react@16.8.6
+ react-dom@16.8.6
+ rollup@1.12.3
added 12 packages from 44 contributors and audited 30 packages in 1.226s
found 0 vulnerabilities

❯ rollup --format=esm main.js
main.js → stdout...
import React, { useEffect } from 'react';
import { render } from 'react-dom';

const originalUseEffect = React.useEffect;

React.useEffect = (callback, deps) => {
  return originalUseEffect(() => {
    console.log("Monkey!");
    return callback();
  }, deps);
};

function App() {
  useEffect(() => {
    console.log("My useEffect code is running.");
  }, []);
  return "test";
}

render(App, document.getElementById("root"));
(!) Unresolved dependencies
https://rollupjs.org/guide/en#warning-treating-module-as-external-dependency
react (imported by main.js, monkey-patch.js)
react-dom (imported by main.js)
created stdout in 54ms
```

As you can see above, Rollup left the React imports intact because it couldn’t find them. This is because by default, Rollup assumes that your code only uses stuff from the web standards. But we can install a plugin to teach it about Node.js’ convention of looking for modules in the `node_modules/` folder.

```
❯ npm i rollup-plugin-node-resolve
+ rollup-plugin-node-resolve@5.0.0
added 118 packages from 102 contributors and audited 939 packages in 3.787s
found 0 vulnerabilities
```

```js
// rollup.config.js
import resolve from "rollup-plugin-node-resolve";

export default {
  plugins: [resolve()]
};
```

```
❯ rollup --config --format=esm main.js
main.js → stdout...
[!] Error: 'default' is not exported by node_modules/react/index.js
https://rollupjs.org/guide/en#error-name-is-not-exported-by-module-
monkey-patch.js (1:7)
1: import React from "react";
          ^
2:
3: const originalUseEffect = React.useEffect;
Error: 'default' is not exported by node_modules/react/index.js
```

Now, Rollup has found its way to the files of React, but it cannot find any `export default` there. Which makes sense. We’ve already looked at the code React ships to npm and observed that it does not contain `export`.

Lucklily, we can install a plugin to teach Rollup about Node.js/CommonJS modules:

```
❯ npm i rollup-plugin-commonjs
+ rollup-plugin-commonjs@10.0.0
added 4 packages from 1 contributor and audited 1849 packages in 2.479s
found 0 vulnerabilities
```

```js
// rollup.config.js
import resolve from "rollup-plugin-node-resolve";
import commonjs from "rollup-plugin-commonjs";

export default {
  plugins: [resolve(), commonjs()]
};
```

```
❯ rollup --config --format=esm main.js
main.js → stdout...
[!] Error: 'useEffect' is not exported by node_modules/react/index.js
https://rollupjs.org/guide/en#error-name-is-not-exported-by-module-
main.js (2:9)
1: import "./monkey-patch.js";
2: import { useEffect } from "react";
            ^
3: import { render } from "react-dom";
Error: 'useEffect' is not exported by node_modules/react/index.js
```

The CommonJS plugin has support for trying to figure out which exports actually exists, but couldn’t find `useEffect` in React’s code. So we need to give it a hint:

```js
// rollup.config.js
import resolve from "rollup-plugin-node-resolve";
import commonjs from "rollup-plugin-commonjs";

export default {
  plugins: [
    resolve(),
    commonjs({
      namedImports: {
        react: ["useEffect"]
      }
    })
  ]
};
```

```
❯ rollup --config --format=esm main.js > bundle.js
main.js → stdout...
created stdout in 2.8s
```

Now it succeeds without warnings. Here are the interesting parts of bundle.js (slightly simplified for brevity):

```js
// Helper function made by Rollup:
function createCommonjsModule(fn, module) {
  return (module = { exports: {} }), fn(module, module.exports), module.exports;
}

// The entire react code:
var react = createCommonjsModule(function(module) {
  // Lots of code.
  var react = {
    // Lots of things.
  };
  module.exports = react;
});

// Here comes `import { useEffect } from "react"` that we wrote in main.js.
// Notice that this appears _before_ the monkey-patch.js code!
var react_1 = react.useEffect;

// monkey-patch.js:
const originalUseEffect = react.useEffect;
react.useEffect = (callback, deps) => {
  return originalUseEffect(() => {
    console.log("Monkey!");
    return callback();
  }, deps);
};

// The entire react-dom code:
var reactDom = createCommonjsModule(function(module) {
  // Lots of code.
});

// main.js:
function App() {
  react_1(() => {
    console.log("My useEffect code is running.");
  }, []);
  return "test";
}
reactDom.render(App, document.getElementById("root"));
```

Let’s make an HTML file and run this!

```html
<!DOCTYPE html>
<meta charset="utf-8" />
<title>Rollup test</title>
<div id="root"></div>
<script src="bundle.js"></script>
```

The browser console tells us this:

```
ReferenceError: process is not defined
```

It’s interesting how much webpack is doing out of the box to get our code running. Lucklily, we can install a plugin to teach Rollup about Node.js globals:

```
❯ npm i rollup-plugin-node-globals
+ rollup-plugin-node-globals@1.4.0
added 7 packages from 83 contributors and audited 2758 packages in 4.257s
found 0 vulnerabilities
```

```js
import resolve from "rollup-plugin-node-resolve";
import commonjs from "rollup-plugin-commonjs";
import globals from "rollup-plugin-node-globals";

export default {
  plugins: [
    resolve(),
    commonjs({
      namedExports: {
        react: ["useEffect"]
      }
    }),
    globals()
  ]
};

```

```
❯ rollup --config --format=esm main.js > bundle.js
main.js → stdout...
created stdout in 3.6s
```

Finally, the browser renders “text” as expected, and in the console we see:

```
My useEffect code is running.
```

But there’s no “Monkey!” in the logs! If we modify main.js to use `React.useEffect` we do see it though:

```js
function App() {
  React.useEffect(() => {
    console.log("My useEffect code is running.");
  }, []);
  return "test";
}
```

```
❯ rollup --config --format=esm main.js > bundle.js
main.js → stdout...
created stdout in 3.7s
```

```
Monkey!
My useEffect code is running.
```

Let’s verify that it works differently with webpack. (Don’t forget to change back to a bare `useEffect` in main.js!)

```
❯ npm i webpack webpack-cli
+ webpack@4.31.0
+ webpack-cli@3.3.2
added 278 packages from 170 contributors and audited 7989 packages in 6.987s
found 0 vulnerabilities
```

```
❯ webpack --mode development --devtool false --output bundle.js main.js
Hash: ba48f3ef1e7ae15f15ed
Version: webpack 4.31.0
Time: 605ms
Built at: 05/19/2019 2:24:08 PM
    Asset     Size  Chunks             Chunk Names
bundle.js  882 KiB    main  [emitted]  main
Entrypoint main = bundle.js
[./main.js] 298 bytes {main} [built]
[./monkey-patch.js] 213 bytes {main} [built]
[./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 472 bytes {main} [built]
    + 11 hidden modules
```

And the browser console does give both logs:

```
Monkey!
My useEffect code is running.
```

So … is Rollup wrong? Not really. Normally, you can’t change what a named export points to from another module.

```js
// test.js
export let something = 1;
```

```html
<!DOCTYPE html>
<meta charset="utf-8" />
<title>export re-assing test</title>
<script type="module">
  import { something } from "./test.js";
  something = 2;
</script>
```

```
TypeError: "something" is read-only
```

This doesn’t help either:

```html
<!DOCTYPE html>
<meta charset="utf-8" />
<title>export re-assing test</title>
<script type="module">
  import * as all from "./test.js";
  all.something = 2;
</script>
```

```
TypeError: "something" is read-only
```

So it shouldn’t really be possible to monkey-patch a bare import of `useEffect`, if you’re following the spec closely. (Unless `useEffect` depends on a mutable object internally that you can modify – but that’s another story). It just happens to work because of implementation details of webpack and React.

The monkey-patching that [stop-runaway-react-effects](https://github.com/kentcdodds/stop-runaway-react-effects) does should work any bundler if you use `import React from "react"; React.useEffect`. Which is kinda nice anyway since you don’t have to adjust your imports all the time every time you use a new hook. But most people use webpack anyway, so you should be fine either way. I would be surprised if webpack suddenly changed their output code in a breaking way.

If you’re interested, there’s an interesting discussion about shipping actual `import`/`export` in React to npm:

https://github.com/facebook/react/issues/11503