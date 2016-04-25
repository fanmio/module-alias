# module-alias
[![NPM Version][npm-image]][npm-url]
[![Build Status][travis-image]][travis-url]

Create aliases of directories and register custom module paths in NodeJS like a boss!

No more shit-coding paths in Node like so:

```js
require('../../../../some/very/deep/module')
```
Enough of this madness!

Just create an alias and do it the right way:

```js
var module = require('@deep/module')
// Or ES6
import module from '@deep/module'
```

**WARNING:** This module should not be used in other npm modules since it modifies the default `require` behavior! It is designed to be used for developing of projects such as web-sites, applications etc.

## Install

```
npm i --save module-alias
```

## Usage

Add these lines to your `package.json` (in your application's root)

```js
// Aliases
"_moduleAliases": {
  "@root"      : ".", // Application's root
  "@client"    : "src/client",
  "@deep"      : "src/some/very/deep/directory",
  "@my_module" : "src/some-file.js",
  "something"  : "src/foo", // Or without @. Actually, it could be any string
}

// Custom module directories (optional)
"_moduleDirectories": ["node_modules_custom"],
```

Then add these line at the very main file of your app, before any code

```js
require('module-alias/register')
```

**And you're all set!** Now you can do stuff like:

```js
require('something')
const module = require('@root/some-module')
const veryDeepModule = require('@deep/my-module')
const customModule = require('my_custom_module') // module from `node_modules_custom` directory

// Or ES6
import 'something'
import module from '@root/some-module'
import veryDeepModule from '@deep/my-module'
import customModule from 'my_custom_module' // module from `node_modules_custom` directory
```

## Advanced usage

If you don't want to modify your `package.json` or you just prefer to set it all up programmatically, then the following methods are available for you:

* `addAlias('alias', 'target_path')` - register a single alias
* `addAliases({ 'alias': 'target_path', ... }) ` - register multiple aliases
* `addPath(path)` - Register custom modules directory (like node_modules, but with your own modules)

_Examples:_
```js
const moduleAlias = require('module-alias')

//
// Register alias
//
moduleAlias.addAlias('@client', __dirname + '/src/client')

// Or multiple aliases
moduleAlias.addAliases({
  '@root'  : __dirname,
  '@client': __dirname + '/src/client',
  ...
})

//
// Register custom modules directory
//
moduleAlias.addPath(__dirname + '/node_modules_custom')
moduleAlias.addPath(__dirname + '/src')

//
// Import settings from a specific package.json
//
moduleAlias(__dirname + '/package.json')

// Or let mudule-alias to figure where your package.json is
// located. By default it will look in the same directory
// where you have your node_modules (application's root)
moduleAlias()
```

## Usage with WebPack

Luckily, WebPack has a built in support for aliases and custom modules directories so it's easy to make it work on the client side as well!

```js
// webpack.config.js
const npm_package = require('./package.json')

module.exports = {
  entry: { ... },
  resolve: {
    root: __dirname,
    alias: npm_package._moduleAliases || {},
    extensions: ['', '.js', '.jsx'],
    modulesDirectories: npm_package._moduleDirectories || [] // eg: ["node_modules", "node_modules_custom", "src"]
  }
}
```

## How it works?

In order to register an alias it modifies the internal `Module._resolveFilename` method so that when you use `require` or `import` it first checks whether the given string starts with one of the registered aliases, if so, it replaces the alias in the string with the target path of the alias.

In order to register a custom modules path (`addPath`) it modifies the internal `Module._nodeModulePaths` method so that the given directory then acts like it's the `node_modules` directory.

[npm-image]: https://img.shields.io/npm/v/module-alias.svg
[npm-url]: https://npmjs.org/package/module-alias
[travis-image]: https://img.shields.io/travis/ilearnio/module-alias/master.svg
[travis-url]: https://travis-ci.org/ilearnio/module-alias
