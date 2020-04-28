# JavaScript API

**Topics**
* [Introduction](./readme.md)
* [Command Line Interface](./01-command-line-interface.md)
* [ES Module Syntax](./03-es-module-syntax.md)
* [Tutorial](./04-tutorial.md)
* [Integrating Rollup with Other Tools](./05-integrating-rollup-with-other-tools.md)
* [Troubleshooting](./06-troubleshooting.md)
* [Big List of Options](./07-big-list-of-options.md)

**Contents**
* [rollup.rollup](#rolluprollup)
* [rollup.watch](#rollupwatch)

Rollup provides a JavaScript API which is usable from Node.js. You will rarely need to use this, and should probably be using the command line API unless you are extending Rollup itself or using it for something esoteric, such as generating bundles programatically.

## rollup.rollup
[Back to Top](#javascript-api)

The `rollup.rollup` function receives an input options object as a parameter and returns a Promise that resolves to a `bundle` object with various properties and methods as shown below. During this step, Rollup will build the module graph and perform tree-shaking, but will not generate any output.

On a `bundle` object, you can call `bundle.generate` multiple times with different output options objects to generate different bundles in-memory. If you directly want to write them to disk, use `bundle.write` instead.

```js
const rollup = require('rollup');

// see below for details on the options
const inputOptions = {...};
const outputOptions = {...};

async function build() {
  // create a bundle
  const bundle = await rollup.rollup(inputOptions);

  console.log(bundle.watchFiles); // an array of file names this bundle depends on

  // generate output-specific code in-memory
  // you can call this function multiple times on the same bundle object
  const { output } = await bundle.generate(outputOptions);

  for (const chunkOrAsset of output) {
    if (chunkOrAsset.type === 'asset') {
      /*
        For assets, this contains:
        {
          fileName: string,             // the asset file name
          source: string | Uint8Array,   // the asset source
          type: 'asset'
        }
      */
     console.log('Asset', chunkOrAsset);
    } else {
      /*
        For chunks, this contains:
        {
          code: string,                   // the generated JS code
          dynamicImports: string[],       // external modules imported dynamically by the chunk
          exports: string[],              // exported variable names
          facadeModuleId: string | null,  // the id of a module that this chunk corresponds to
          fileName: string,               // the chunk file name
          imports: string[],              // external modules imported statically by the chunk
          isDynamicEntry: boolean,        // is this chunk a dynamic entry point
          isEntry: boolean,               // is this chunk a static entry point
          map: string | null,             // sourcemaps if present
          modules: {                      // information about the modules in this chunk
            [id: string]: {
              renderedExports: string[];  // exported variable names that were included
              removedExports: string[];   // exported variable names that were removed
              renderedLength: number;     // the length of the remaining code in this module
              originalLength: number;     // the original length of the code in this module
            };
          },
          name: string,                   // the name of this chunk as used in naming patterns
          type: 'chunk'                   // signifies that this is a chunk
        }
      */
      console.log('Chunk', chunkOrAsset.modules);
    }
  }

  // or write the bundle to disk
  await bundle.write(outputOptions);
}

build();
```

#### inputOptions object

The `inputOptions` object can contain the following properties (see the [big list of options](./07-big-list-of-options.md) for full details on these):

```js
const inputOptions = {
  // core input options
  external,
  input, // required
  plugins,

  // advanced input options
  cache,
  inlineDynamicImports,
  manualChunks,
  onwarn,
  preserveEntrySignatures,
  preserveModules,
  strictDeprecations,

  // danger zone
  acorn,
  acornInjectPlugins,
  context,
  moduleContext,
  preserveSymlinks,
  shimMissingExports,
  treeshake,

  // experimental
  experimentalCacheExpiry,
  perf
};
```

#### outputOptions object

The `outputOptions` object can contain the following properties (see the [big list of options](./07-big-list-of-options.md) for full details on these):

```js
const outputOptions = {
  // core output options
  dir,
  file,
  format, // required
  globals,
  name,
  plugins,

  // advanced output options
  assetFileNames,
  banner,
  chunkFileNames,
  compact,
  entryFileNames,
  extend,
  externalLiveBindings,
  footer,
  hoistTransitiveImports,
  interop,
  intro,
  minifyInternalExports,
  outro,
  paths,
  sourcemap,
  sourcemapExcludeSources,
  sourcemapFile,
  sourcemapPathTransform,

  // danger zone
  amd,
  esModule,
  exports,
  freeze,
  indent,
  namespaceToStringTag,
  noConflict,
  preferConst,
  strict
};
```

## rollup.watch
[Back to Top](#javascript-api)

Rollup also provides a `rollup.watch` function that rebuilds your bundle when it detects that the individual modules have changed on disk. It is used internally when you run Rollup from the command line with the `--watch` flag.

```js
const rollup = require('rollup');

const watchOptions = {...};
const watcher = rollup.watch(watchOptions);

watcher.on('event', event => {
  /*
    event.code can be of:
    // START          - the watcher is (re)starting
    // BUNDLE_START   - building an individual bundle
    // BUNDLE_END     - finished building a bundle
    // END            - finished building all bundles
    // ERROR          - encountered an error while bundling
  */
});

// stop watching
watcher.close();
```

#### watchOptions

The `watchOptions` argument is a config (or an array of configs) that you would export from a config file.

```js
const watchOptions = {
  ...inputOptions,
  output: [outputOptions],
  watch: {
    chokidar,
    clearScreen,
    skipWrite,
    exclude,
    include
  }
};
```

See above for details on `inputOptions` and `outputOptions`, or consult the [big list of options](./07-big-list-of-options.md) for info on `chokidar`, `include`, and `exclude`.

#### Programmatically loading a config file

In order to aid in generating such a config, rollup exposes the helper it uses to load config files in its command line interface via a separate entry-point. This helper receives a resolved `fileName` and optionally an object containing command line parameters:

```js
const loadConfigFile = require('rollup/dist/loadConfigFile');
const path = require('path');
const rollup = require('rollup');

// load the config file next to the current script;
// the provided config object has the same effect as passing "--format es"
// on the command line and will override the format of all options
loadConfigFile(path.resolve(__dirname, 'rollup.config.js'), { format: 'es' })
  .then(async ({options, warnings}) => {
    // "warnings" wraps the default 'onwarn' handler passed by the CLI.
    // This prints all warnings up to this point:
    console.log(`We currently have ${warnings.count} warnings`);

    // This prints all deferred warnings
    warnings.flush();

    // options in an "inputOptions" object with an additional "output"
    // property that contains an array of "outputOptions".
    // The following will generate all outputs and write them to disk the same
    // way the CLI does it:
    const bundle = await rollup.rollup(options);
    await Promise.all(options.output.map(bundle.write));

    // You can also pass this directly to "rollup.watch"
    rollup.watch(options);
  })
```