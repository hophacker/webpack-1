# Configuring React

Facebook's [React](https://facebook.github.io/react/) is a popular alternative for developing web applications. Even if you don't use it, it can be valuable to understand how to configure it. Most React setups rely on a transpiler known as [Babel](https://babeljs.io/).

Babel is useful beyond React development and worth understanding as it allows you to use future JavaScript features now without having to worry too much about browser support. Due to technical constraints it doesn't support all features within the specification, but still it can be a good tool to have in your arsenal.

## Setting Up Babel with React

Most of the React code out there relies on a format known as [JSX](https://facebook.github.io/jsx/). It is a superset of JavaScript that allows you to mix XMLis syntax with JavaScript. A lot of people find this convenient as they get something that resembles what they know already while they can use the power of JavaScript. This is in contrast to template DSLs that implement the same logic using custom constructs.

Some React developers prefer to attach type annotations to their code using a language extension known as [Flow](http://flowtype.org/). The technology fits React well, but it's not restricted to it. [TypeScript](http://www.typescriptlang.org/) is another viable alternative. Both work with JSX.

Babel allows us to use JSX with React easily. In addition, we can enable language features we want either using plugins or presets that encapsulate collections of plugins. For instance you can find all ES6 features within a preset. The same goes for React related functionality. We'll be relying on these within our setup.

T> It is a good practice to name React components containing JSX using the `.jsx` suffix. In addition to communicating this fact, your editor can apply syntax highlighting automatically as you open the files.

### Installing *babel-loader*

The first step towards configuring Babel to work with Webpack is to set up [babel-loader](https://www.npmjs.com/package/babel-loader). It will take our futuristic code and turn it into a format normal browsers can understand. Install *babel-loader with:

```bash
npm i babel-loader babel-core --save-dev
```

*babel-core* contains the core logic of Babel so we need to install that as well.

### Connecting *babel-loader* with Webpack

Now that we have the loader installed, we can connect it with Webpack configuration. In addition to a loader definition, we can perform an additional tweak to make imports without an extension possible. This is optional and some people prefer to leave possible `.jsx` extension visible to communicate clearly that the imported file contains JSX syntax.

Webpack provides a field known as [resolve.extensions](https://webpack.github.io/docs/configuration.html#resolve-extensions) that can be used for this purpose. If you want to allow imports like `import Button from './Button';`, set it up as follows:

**webpack.config.js**

```javascript
...
// Important! Do not remove ''. If you do, imports without
// an extension won't work anymore!
resolve: {
  extensions: ['', '.js', '.jsx']
},
...
```

The loader configuration is straight-forward as well. We can use a RegExp to match both `.js` and `.jsx` files. It's up to your tastes to figure out a neat pattern. I prefer to use `\.jsx?$` myself. This just makes `x` optional. Alternatively you could spell out the options using a matcher such as `\.(js|jsx)$`. Latter format can be particularly useful if you have to match against multiple different formats.

*babel-loader* comes with a set of options. In this case I'm going to enable `cacheDirectory` to improve its performance during development. Simply passing it as a flag helps. You can also pass a specific directory to it as a parameter. I.e., `babel?cacheDirectory=<path>`. Here's the full loader configuration:

**webpack.config.js**

```javascript
...
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      // Enable caching for improved performance during development
      // It uses default OS directory by default. If you need something
      // more custom, pass a path to it. I.e., babel?cacheDirectory=<path>
      loaders: ['babel?cacheDirectory'],
      // Parse only app files! Without this it will go through entire project.
      // In addition to being slow, that will most likely result in an error.
      include: PATHS.app
    },
    ...
  }
}
...
```

Even though we have Babel installed and set up, we are still missing one bit - Babel configuration. It would be possible to pass it through the loader definition. Instead, I prefer to handle it using a dotfile known as *.babelrc*.

The benefit of doing this is that it allows us to process our Webpack configuration itself using the same language rules. The rules will also be shared with Babel running outside of Webpack. This can be useful for package authors.

T> If you want to process your Webpack configuration through Babel, name your Webpack configuration as *webpack.config.babel.js*. Webpack will notice you want to use Babel and execute your configuration through it.

### Setting Up *.babelrc*

A minimal Babel and React setup needs just two Babel presets. Install them:

```bash
npm i babel-preset-es2015 babel-preset-react --save-dev
```

T> Instead of typing it all out, we could use brace expansion. Example: `npm i babel-preset-{es2015,react} -D`. `-D` equals `--save-dev` as you might remember.

To make Babel aware of them, we need to write a *.babelrc*:

**.babelrc**

```json
{
  "presets": [
    "es2015",
    "react"
  ]
}
```

Babel should pick up the presets now and you should be able to develop both ES6 and React code using Webpack now.

Sometimes you might want to use experimental features. Although you can find a lot of them within so called stage presets, I recommend enabling them one by one and even organizing them to a preset of their own unless you are working on a throwaway project. If you expect your project to live a long time, it's better to document the features you are using well.

T> There are other possible [.babelrc options](https://babeljs.io/docs/usage/options/) beyond the ones covered here.

## Setting Up Hot Loading

One of the features that sets React and Webpack apart is a feature known as hot loading. This is something that sits on top of Webpack's Hot Module Replacement (HMR). The idea is that instead of forcing a full refresh on modification, we patch the code that changed during the runtime. The advantage of doing this is that it allows our application to retain its state. The process isn't fool proof, but when it works, it's quite neat.

You could achieve something similar by persisting your application state in other ways. For instance you could consider using `localStorage` for a similar purpose. You will still get a refresh, but it's far better than losing the entire state. You can reach the same result using multiple ways.

You can even implement the hot loading interface on your own. I'll show you the basic setup for a state container known as [Redux](http://redux.js.org/). It was designed with hot loading in mind and the approach works very well with it.

### Setting Up *babel-preset-react-hmre*

A lot of the hard work has been done for us already. In order to configure our setup to support hot loading, we need to enable a Babel preset known as [babel-preset-react-hmre](https://www.npmjs.com/package/babel-preset-react-hmre) during development. To get started, install it:

```bash
npm i babel-preset-react-hmre --save-dev
```

Given it doesn't make sense to instrument our code with the hot loading logic for production usage, we should restrict it development only. One way to achieve this is to control *.babelrc* through `BABEL_ENV` environment variable.

If you are following the single file setup discussed in this book, we can control it using npm lifecycle event captured when npm is executed. This gives a predictable mapping between *package.json* and *.babelrc*. You can achieve this as follows:

**webpack.config.js**

```javascript
...

leanpub-start-insert
process.env.BABEL_ENV = TARGET;
leanpub-end-insert

...
```

In addition we need to expand our Babel configuration to include the plugin we need during development. This is where that `BABEL_ENV` comes in. Babel determines the value of `env` like this:

1. Use the value of `BABEL_ENV` if set.
2. Use the value of `NODE_ENV` if set.
3. Default to `development`.

To connect `BABEL_ENV='start'` with Babel, configure as follows:

**.babelrc**

```json
{
  "presets": [
    "es2015",
    "react"
leanpub-start-delete
  ]
leanpub-end-delete
leanpub-start-insert
  ],
  "env": {
    "start": {
      "presets": [
        "react-hmre"
      ]
    }
  }
leanpub-end-insert
}
```

After these steps your development setup should support hot loading. It is one of those features that makes development a little faster.

### Configuring Redux

In order to configure Redux reducers to support hot loading, we need to implement Webpack's hot loading protocol. Webpack provides a hook known as `module.hot.accept`. It gets called whenever Webpack detects a change. This allows you to reload and patch your code.

The idea is useful beyond Redux and can be used with other systems as well. To give you a rough implementation, consider the code below:

```javascript
...

export default function configureStore(initialState) {
  const store = createStoreWithMiddleware(rootReducer, initialState);

  if(module.hot) {
    // Enable Webpack hot module replacement for reducers
    module.hot.accept('../reducers', () => {
      const nextReducer = require('../reducers/index').default;

      store.replaceReducer(nextReducer);
    });
  }

  return store;
}
```

The code doesn't do that much. It just waits for a change and then patches the code. The feasibility of patching depends on the underlying architecture. For a system like Redux it is simple given it was designed to be patched. It might be harder to pull off for something else.

T> You can find [a full implementation of the idea online](https://github.com/survivejs-demos/redux-demo).

## Setting Up Flow

[Flow](http://flowtype.org/) performs static analysis based on your code and its type annotations. This means you will install it as a separate tool. You will then run it against your code. There's a Webpack plugin known as [flow-status-webpack-plugin](https://www.npmjs.com/package/flow-status-webpack-plugin) that allows you to run it through Webpack during development.

When using React, the Babel preset does most of the work. It is able to strip Flow annotations and convert your code into a format that is possible to transpile further.

There's a Babel plugin known as [babel-plugin-typecheck](https://www.npmjs.com/package/babel-plugin-typecheck) that allows you to perform runtime checks based on your Flow annotations. After installing, you should just add the following section to the development section of your *.babelrc* to enable it:

**.babelrc**

```json
"plugins": [
  [
    "typecheck"
  ]
]
```

Even though useful, Flow static checker is able to catch more errors. Runtime checks are still cool, though, and worth enabling if you are using Flow.

## Setting Up TypeScript

Microsoft's [TypeScript](http://www.typescriptlang.org/) is a far more established solution than Facebook's Flow. As a result you will find more premade type definitions for it and overall the quality of support should be better. You can use it with Webpack using at least the following loaders:

* [ts-loader](https://www.npmjs.com/package/ts-loader)
* [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader)
* [typescript-loader](https://www.npmjs.com/package/typescript-loader)

## Conclusion

There are a lot of aspects to keep in mind when configuring Webpack to work with React. Fortunately this is something you don't have to perform often. Once you have a solid basic setup fitting your needs together, it will take you far.
