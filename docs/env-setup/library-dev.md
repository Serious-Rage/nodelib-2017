## Dev environment 2016-2017

## Tools to install

- linting: [xo](https://github.com/sindresorhus/xo) using [eslint]()
- testing: [ava](https://github.com/ava/ava)
- BDD: [ava-spec]()
- test doubles: [testdouble.js]()
- browser testing: [browser-env]()
- code coverage: [nyc](https://github.com/istanbuljs/nyc) with [coveralls.io](https://www.npmjs.com/package/coveralls)
- Bundling [webpack]()
- Complexity analysis [plato]

## Recipe

Remove `/usr/local/lib/node_modules/` 

`yarn add babel-cli -g`

*bonus npm tools*
`yarn add npm-run npm-quick-run npm-watch -g` 

*create package.json for project*
- `yarn init`

## Ava test config

*install ava binary on system*
`npm install --global ava`

*init ava on project*
`ava --init`

Configure `ava` entry in `package.json`. 

```json
  "ava": {
    "babel": "inherit",
    "require": [
      "babel-register"
    ]
  },
```

## Babel config

*babel extras*
- `yarn add babel-register babel-plugin-transform-runtime --dev`

*Add babel-polyfill*

- `yarn add babel-polyfill --save`

Update `ava` config. We need to always require `babel-register` and `babel-polyfill` in order that imports work when tests are run.

```json
  "ava": {
    "babel": "inherit",
    "require": [
      "babel-register",
      "babel-polyfill"
    ]
  },
```

*Add babel: preset-latest minimal*

To minimize transform plugins and use native node functionality as much as possible. Use preset es2015, stage2 when building however!

- `yarn add babel-preset-latest-minimal --dev`

`babel-node-list-required` to see list of required transform plugins for current Node.js version

- `yarn add babel-plugin-transform-es2015-duplicate-keys babel-plugin-transform-es2015-modules-commonjs babel-plugin-syntax-trailing-function-commas babel-plugin-transform-async-to-generator --dev`

*add legacy decorators (optional)*
- `yarn add babel-plugin-transform-decorators-legacy --dev`

To be merged with any other config where typescript should be enabled.

## File watches and triggers

See [ava watch mode](https://github.com/avajs/ava/blob/master/docs/recipes/watch-mode.md)

*watching*

Watch config for `test` script, will run whenever triggered, in this case in quiet mode.

```json
{
  "watch": {
    "test": {
      "patterns": ["src", "test"],
      "extensions": "js,jsx",
      "quiet": true
    }
  },
  "scripts": {
    "test": "ava",
    "watch": "npm-watch"
  }
}
```

We recommend co-locating tests with the `/src` code so it is clear which `.js` files have a `.test.js` sibling.
If you follow this convention, you can remove the `"test"` from file `"patterns"` to match on. 
Also remove `"jsx"` if you are not developing for React.

`npm run watch` to watch everything and trigger

*lint style checks and fixes*
- `yarn add xo eslint eslint-plugin-babel babel-eslint eslint-plugin-filenames --dev`

Installs `babel-eslint` to be used as the ESLint parser. This is useful when we are using advanced language 
features *such as decorators etc) that ESLint doesn't yet support.

We also add the `filenames` plugin to add conventions for filenames, here that we only allow alphanumeric, `-` and `.`, 
thus no numbers or underscores `_`.

In `package.json` add `xo` eslint settings:

```json
  "xo": {
    "esnext": true,
    "parser": "babel-eslint",
    "plugins": [
      "filenames"
    ],
    "rules": {
      "filenames/match-regex": [2, "^[a-z-\\.]+$", true]
    }
  }
```

In `test` script, prefix with `xo &&` to run `xo` before running tests.
We also add a `fix` script to have `xo` automatically fix the code!

```
"test": "xo && ava",
"fix": "xo --fix -- -s",
```

We could even have it run and fi the code before running tests!

`"test": "npm run fix && ava",`

Install [VSCode XO plugin](https://github.com/SamVerschueren/vscode-linter-xo) - let's you autofix!

On Mac, Cmd-P, then type: `ext install linter-xo` or go to `View->Extensions` menu.

*testing extras*
- `yarn add ava-spec testdouble --dev`

```js
import td from 'testdouble'

test('fetch', t => {
	let fetch = td.function();
	td.when(fetch(42)).thenReturn('Jane User');

	t.is(fetch(42), 'Jane User');
});
```

or using new BDD syntax from `ava-spec`

```js
import test from 'ava-spec';

test('AVA Spec is 100% compatible with ava', t => {
  t.is(true, true);
});
```

*coverage*
- `yarn add nyc --dev`

Use `nyc` to run test coverage reports:

```
    "report": "nyc report --reporter=html",
    "cover": "nyc ava",
```

*watch and cover*

`yarn add nodemon --dev`
    
`    "watch:cover": "nodemon --quiet --watch sec --exec npm run cover -s"`

Alternatively use `npm-watch` config:

```json
"watch": {
  "cover": {    
    "quiet": true
  }
}
```

Add to `.gitignore`

```
coverage
.nyc_output
```

### Coveralls reporting

- `yarn add coveralls --dev`

Register repo on [coveralls.io](coveralls.io)

"takes json-cov output into stdin and POSTs to coveralls.io"

Add a `coverall` script to `package.json`

`    "coveralls": "npm-run nyc report --reporter=text-lcov | npm-run coveralls"`

*complexity analysis*
- `yarn add plato --dev`

Add plato reporting to `package.json` `scripts` entry:

```
    "plato:report": "npm-run plato -r -d report src",
    "plato:report:display": "open report/index.html",
```

`plato` outputs report in the `report` folder.
Add `report` folder to `.gitignore`

```
report
```

*CI*

Register repo on on [travis.io](travis.io)

`.travis.yml`


```
language: node_js
sudo: false
node_js:
  - v7
after_script:
- npm run coveralls
```

### Documentation

`yarn add global documentation`

`documentation build ./index.js -f md > docs/lib/API.md`

Then add a few `docs` script

```   
    "docs": "npm run docs:html && npm run docs:md",
    "docs:html": "documentation build ./index.js -f html -o documentation/lib/html",
    "docs:md": "documentation build ./index.js -f md > documentation/lib/md/API.md"
```

- [Getting started](https://github.com/documentationjs/documentation/blob/master/docs/GETTING_STARTED.md)
- [JsDocs 3 guide](http://usejsdoc.org/about-getting-started.html)

To view Markdown docs try [typora](http://www.typora.io/)
Typora [themes](http://theme.typora.io/)

*Mac OSX only*
- [macdown](http://macdown.uranusjr.com/)
- [mou](http://25.io/mou/)

[Add as default app](http://www.imore.com/how-change-default-apps-os-x) for `.md` documents

### Add flow

```
touch .flowconfig
npm install --save-dev flow-bin
```



### Decorator

- [lodash-decorators](https://www.npmjs.com/package/lodash-decorators)
- [core-decorators](https://www.npmjs.com/package/core-decorators)

### Documentation

- [documentation.js](https://github.com/documentationjs/documentation)

### Karma

- [karma-mocha-reporter](https://www.npmjs.com/package/karma-mocha-reporter)

### Flowtype

- [flowtype](https://flowtype.org/)

### Plato reports

`npm-run plato -r -d reports ./`

### Code style

- [xo](https://github.com/sindresorhus/xo)

### Babel plugins

List current plugins needed according to the version of node:

`npm-run babel-node-list-required`

```
[ 'transform-es2015-duplicate-keys',
  'transform-es2015-modules-commonjs',
  'syntax-trailing-function-commas',
  'transform-async-to-generator' ]
```

`npm i babel-plugin-transform-es2015-duplicate-keys babel-plugin-transform-es2015-modules-commonjs babel-plugin-syntax-trailing-function-commas babel-plugin-transform-async-to-generator --dev`
