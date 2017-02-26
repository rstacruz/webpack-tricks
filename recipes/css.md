Output CSS files
================

> __NOTE:__ This guide only applies to Webpack 2.x.

Webpack supports CSS via [css-loader](https://www.npmjs.com/package/css-loader) and [style-loader](https://www.npmjs.com/package/style-loader). You can then use `require('./style.css')` with some minimal configuration. I personally don't find loading CSS via JS an ideal setup, so here's a complicated way to output `.css` files with source maps in development.

> __NOTE:__ This is complicated and hard to follow along. Check [rstacruz/webpack-starter-kit](https://github.com/rstacruz/webpack-starter-kit) for an example.

## What you'll need

You'll need [ExtractTextWebpackPlugin](https://www.npmjs.com/package/extract-text-webpack-plugin). Let's set up a `DEBUG` constant as described in [§ Development mode](../README.md#development-mode) so we can set up source maps only for development.

```js
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const DEBUG = process.env.NODE_ENV !== 'production'
```

## Entry point

First define an entry point for your CSS. This will generate `assets/css/app.css` based on `web/css/app.css`.

```js
entry: {
  'assets/css/app': './web/css/app.css'
},
```

## Module rule

Add a module rule to parse `*.css` files via ExtractTextPlugin, `postcss-loader` and `css-loader`. The `DEBUG` magic here will only enable source maps in development mode.

```js
module: {
  rules: [
    {
      test: /\.css$/,
      use: ExtractTextPlugin.extract({
        fallback: 'style-loader',
        use: [
          `css-loader?-url${DEBUG ? '&sourceMap&importLoaders=1' : ''}`,
          `postcss-loader${DEBUG ? '?sourceMap=inline' : ''}`
        ]
      })
    }
  ]
}
```

## Extract to .css

The entry point actually generates `assets/css/app.js` (JavaScript), but we want it as `app.css`. By using ExtractTextPlugin, you can move the CSS parts into a CSS file.

```js
plugins: [
  new ExtractTextPlugin({
    filename: '[name].css',
    allChunks: true // preserve source maps
  }),
]
```

## Source maps

You need to use `devtool: 'source-map'` for CSS source maps to work.

```js
devtool: DEBUG ? 'source-map' : 'hidden-source-map',
```

## PostCSS configuration

You most likely want to use PostCSS. Even if you use Less or Sass, PostCSS will let you do autoprefixing and other magic. You'll need a `postcss.config.js` configuration. In this example, I'm adding [postcss-cssnext](https://www.npmjs.com/package/postcss-cssnext), which adds automatic vendor prefixes and transpiling for support for future CSS features.

```js
// postcss.config.js
const DEBUG = process.env.NODE_ENV !== 'production'

module.exports = {
  plugins: [
    require('postcss-import')(),
    require('postcss-cssnext')(),
  ]
}
```
