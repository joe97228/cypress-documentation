---
title: file:preprocessor
---

The `file:preprocessor` event allows you to modify how your spec files and support files are preprocessed for the browser.

A preprocessor is the plugin responsible for preparing a {% url "support file" writing-and-organizing-tests#Support-file %} or a {% url "test file" writing-and-organizing-tests#Test-files %} for the browser.

A preprocessor could transpile your file from another language (CoffeeScript or ClojureScript) or from a newer version of JavaScript (ES2017).

A preprocessor also typically watches the source files for changes, processes them again, and then notifies Cypress to re-run the tests.

# Environment

Event | {% url "Browser" catalog-of-events#Browser-Events %} | {% url "Background Process" background-process %}
--- | --- | ---
`file:preprocessor` | | {% fa fa-check-circle green %}

# Arguments

**{% fa fa-angle-right %} file** ***(Object)***

The file being processed.

# Usage

## In the background process

Using your {% url "`backgroundFile`" background-process %} you can tap into the `file:preprocessor` event.

```javascript
// background file
module.exports = (on, config) => {
  on('file:preprocessor', (file) => {
    // ...
  })
}
```

# Examples

We have created three preprocessors as examples. These are fully functioning preprocessors.

The code contains comments that explain how it utilizes the preprocessor API.

* {% url 'Browserify Preprocessor' https://github.com/cypress-io/cypress-browserify-preprocessor %}
* {% url 'Webpack Preprocessor' https://github.com/cypress-io/cypress-webpack-preprocessor %}
* {% url 'Watch Preprocessor' https://github.com/cypress-io/cypress-watch-preprocessor %}

# Recipes

We also have some recipes showing how to utilize these preprocessors.

Here are two recipes using both webpack and browserify to write your tests in TypeScript.

- {% url 'TypeScript with Browserify Preprocessor' https://github.com/cypress-io/cypress-example-recipes/tree/master/examples/preprocessors__typescript-browserify %}
- {% url 'TypeScript with Webpack Preprocessor' https://github.com/cypress-io/cypress-example-recipes/tree/master/examples/preprocessors__typescript-webpack %}

# Defaults

By default, Cypress comes packaged with the **Browserify Preprocessor** already installed.

The Browserify Preprocessor handles:

- CoffeeScript `1.x.x`
- ES2015 via Babel
- JSX and CJSX
- Watching and caching files

The exact default configuration options {% url 'can be found here' https://github.com/cypress-io/cypress-browserify-preprocessor#browserifyoptions %}.

{% note info "Are you looking to change the default options for Browserify?" %}

Changing the Browserify options lets you:

- Add your own Babel plugins
- Add support for Typescript
- Add support for CoffeeScript `2.x.x`

Please read this link in the {% url 'browserify preprocessor' https://github.com/cypress-io/cypress-browserify-preprocessor#modifying-default-options %} repo for instructions on modifying these.
{% endnote %}

### The callback function should return one of the following:

* A promise\* that eventually resolves the path to the **built file**\*\*.
* A promise\* that eventually rejects with an error that occurred during processing.

> \* The promise should resolve only after the file has completed writing to disk. The promise resolving is a signal that the file is ready to be served to the browser.

---

> \*\* The built file is the file that is created by the preprocessor that will eventually be served to the browser.

If, for example, the source file is `spec.coffee`, the preprocessor should:

1. Compile the CoffeeScript into JavaScript `spec.js`
2. Write that JavaScript file to disk (example: `/Users/foo/tmp/spec.js`)
3. Resolve with the absolute path to that file: `/Users/foo/tmp/spec.js`

{% note warning %}
This callback function can and *will* be called multiple times with the same `filePath`.

The callback function is called any time a file is requested by the browser. This happens on each run of the tests.

Make sure not to start a new watcher each time it is called. Instead, cache the watcher and, on subsequent calls, return a promise that resolves when the latest version of the file has been processed.
{% endnote %}

# File object

The `file` object passed to the callback function has the following properties:

Property | Description
-------- | ----------
`filePath` | The full path to the source file.
`outputPath` | The suggested path for saving the preprocessed file to disk. This is unique to the source file. A preprocessor can choose to write the file elsewhere, but Cypress automatically provides you this value as a convenient default.
`shouldWatch` | A boolean indicating whether the preprocessor should watch for file changes or not.

# File events

The `file` object passed to the callback function is an {% url "Event Emitter" https://nodejs.org/api/events.html#events_class_eventemitter %}.

### Receiving 'close' event

When the running spec, the project, or the browser is closed while running tests, the `close` event will be emitted. The preprocessor should do any necessary cleanup in this function, like closing the watcher when watching.

```javascript
const watcher = fs.watch(filePath, /* ... */)

file.on('close', () => {
  watcher.close()
})
```

### Sending 'rerun' event

If watching for file changes, emit `rerun` after a file has finished being processed to let Cypress know to rerun the tests.

```javascript
fs.watch(filePath, () => {
  file.emit('rerun')
})
```

# Publishing

Publish preprocessors to {% url "npm" https://www.npmjs.com/ %} with the naming convention `cypress-*-preprocessor` (e.g. cypress-clojurescript-preprocessor).

Use the following npm keywords:

```json
"keywords": [
  "cypress",
  "cypress-plugin",
  "cypress-preprocessor"
]
```

Feel free to submit your published plugin to our {% url "list of plugins" plugins %}.

# See also

- {% url `before:spec` before-spec-event %}
- {% url "Plugins" plugins %}