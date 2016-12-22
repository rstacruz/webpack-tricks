# Webpack tricks

Just a small catalog of Webpack tips and tricks I've learned. All of those tips and tricks concern Webpack 1. Webpack 2 has a different API, so some of those tips won't work there. A detailed guide about migrating to v2 [can be found here](http://javascriptplayground.com/blog/2016/10/moving-to-webpack-2/).

Table of contents
-----------------

  * [Table of contents](#table-of-contents)
  * [Progress reporting](#progress-reporting)
  * [Minification](#minification)
  * [Multiple bundles](#multiple-bundles)
  * [Split app and vendor code](#split-app-and-vendor-code)
  * [Source maps](#source-maps)
  * [CSS](#css)
  * [Development mode](#development-mode)
  * [Investigating bundle sizes](#investigating-bundle-sizes)
  * [Smaller React](#smaller-react)
  * [Smaller Lodash](#smaller-lodash)
  * [Requiring all files in a folder](#requiring-all-files-in-a-folder)

Progress reporting
------------------

Invoke Webpack with:

```
--progress --colors
```

Minification
------------

Invoke Webpack with `-p` for production builds.

```js
webpack -p
```

Multiple bundles
----------------

Export multiple bundles by setting the output to `[name].js`. This example produces `a.js` and `b.js`.

```js
module.exports = {
  entry: {
    a: './a',
    b: './b'
  },
  output: { filename: '[name].js' }
}
```

Concerned about duplication? Use the [CommonsChunkPlugin](https://webpack.github.io/docs/list-of-plugins.html#commonschunkplugin) to move the common parts into a new output file.

```js
plugins: [ new webpack.optimize.CommonsChunkPlugin('init.js') ]
```

```html
<script src='init.js'></script>
<script src='a.js'></script>
```

Split app and vendor code
-------------------------

Use CommonsChunkPlugin to move vendor code into `vendor.js`.

```js
var webpack = require('webpack')

module.exports = {
  entry: {
    app: './app.js',
    vendor: ['jquery', 'underscore', ...]
  },

  output: {
    filename: '[name].js'
  },

  plugins: [
    new webpack.optimize.CommonsChunkPlugin('vendor')
  ]
}
```

How this works:

- We make a `vendor` entry point and load it with some libraries
- CommonsChunkPlugin will remove these libraries from `app.js` (because it appears in 2 bundles now)
- CommonsChunkPlugin also moves the Webpack runtime into `vendor.js`

> Reference: [Code splitting](https://webpack.github.io/docs/code-splitting.html#split-app-and-vendor-code)

Source maps
-----------

The best source maps option is `cheap-module-eval-source-map`. This shows original source files in Chrome/Firefox dev tools. It's faster than `source-map` and `eval-source-map`.

```js
const DEBUG = process.env.NODE_ENV !== 'production'

module.exports = {
  debug: DEBUG ? true : false,
  devtool: DEBUG ? 'cheap-module-eval-source-map' : 'hidden-source-map'
}
```

Your files will now show up in Chrome Devtools as `webpack:///foo.js?a93h`. We want this to be cleaner like `webpack:///path/to/foo.js`.

```js
  output: {
    devtoolModuleFilenameTemplate: 'webpack:///[absolute-resource-path]'
  }
```

> Reference: [devtool documentation](https://webpack.github.io/docs/configuration.html#devtool)

CSS
---

It's complicated. TBD

Development mode
----------------

Want to have certain options only appear in development mode?

```js
const DEBUG = process.env.NODE_ENV !== 'production'

module.exports = {
  debug: DEBUG ? true : false,
  devtool: DEBUG ? 'cheap-module-eval-source-map' : 'hidden-source-map'
}
```

Be sure to invoke Webpack as `env NODE_ENV=production webpack -p` when building your production assets.

Investigating bundle sizes
--------------------------

Want to see what dependencies are the largest? Use webpack-bundle-size-analyzer.

```js
$ yarn global add webpack-bundle-size-analyzer
```

```js
$ ./node_modules/.bin/webpack --json | webpack-bundle-size-analyzer
jquery: 260.93 KB (37.1%)
moment: 137.34 KB (19.5%)
parsleyjs: 87.88 KB (12.5%)
bootstrap-sass: 68.07 KB (9.68%)
...
```

> Reference: [webpack-bundle-size-analyzer](https://github.com/robertknight/webpack-bundle-size-analyzer)

Smaller React
-------------

React will build dev tools by default. You don't need this in production. Use the DefinePlugin to make these dev tools disappear. This saves you around 30kb.

```js
plugins: [
  new webpack.DefinePlugin({
    'process.env': {
      'NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development')
    }
  })
]
```

Be sure to invoke Webpack as `env NODE_ENV=production webpack -p` when building your production assets.

Smaller Lodash
-------------

[Lodash](https://lodash.com/) is very useful but usually we only need a small part of its full functionality. [lodash-webpack-plugin](https://github.com/lodash/lodash-webpack-plugin) can help you shrink the lodash build by replacing [feature sets](https://github.com/lodash/lodash-webpack-plugin#feature-sets) of modules with [noop](https://lodash.com/docs#noop), [identity](https://lodash.com/docs#identity), or simpler alternatives.

```js
const LodashModuleReplacementPlugin = require('lodash-webpack-plugin');

const config = {
  plugins: [
    new LodashModuleReplacementPlugin({
      path: true,
      flattening: true
    })
  ]
};
```

This may save you >10kb depending on how much you use lodash.

Requiring all files in a folder
-------------------------------

Ever wanted to do this?

```js
require('./behaviors/*')  /* Doesn't work! */
```

Use require.context.

```js
// http://stackoverflow.com/a/30652110/873870
function requireAll (r) { r.keys().forEach(r) }

requireAll(require.context('./behaviors/', true, /\.js$/))
```

> Reference: [require.context](http://webpack.github.io/docs/context.html#require-context)
