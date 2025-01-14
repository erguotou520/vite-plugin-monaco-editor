# Vite Plugin Monaco Editor

A plugin to simplify loading the [Monaco Editor](https://github.com/Microsoft/monaco-editor) with [vite](https://vitejs.dev/).

- It uses Vite specific plugin hooks: configResolved, configureServer, transformIndexHtml.
- It uses esbuild to bundle worker in the `node_modules/.monaco` directory, via the `server.middlewares` proxy http server for the bundle worker.

## Installing

```ts
// make sure you have it installed monaco-editor.

yarn add -D vite-plugin-monaco-editor monaco-editor

// or
npm install -D vite-plugin-monaco-editor monaco-editor

// or
pnpm install -D vite-plugin-monaco-editor monaco-editor
```

## Using

- `vite.config.ts`:

```ts
import { defineConfig } from 'vite';
import monacoEditorPlugin from 'vite-plugin-monaco-editor';

export default defineConfig({
  plugins: [monacoEditorPlugin()],
});
```

### Import all monaco functions

- `index.ts`:

```ts
import * as monaco from 'monaco-editor';

monaco.editor.create(document.getElementById('container'), {
  value: 'console.log("Hello, world")',
  language: 'javascript',
});
```

### Import part of monaco functions

The `import * as monaco from 'monaco-editor'` is import all features and languages of the Monaco Editor. Assume you only need part of the features and languages:

- `customMonaco.ts`

```ts
import 'monaco-editor/esm/vs/editor/editor.all.js';
import 'monaco-editor/esm/vs/editor/standalone/browser/accessibilityHelp/accessibilityHelp.js';

import * as monaco from 'monaco-editor/esm/vs/editor/editor.api';

export { monaco };
```

The Complete list of imports: [customMonaco.ts](test/src/mona/customMonaco.ts)

- `index.ts`

```ts
import { monaco } from './customMonaco.ts';
monaco.editor.create(document.getElementById('container'), {
  value: 'console.log("Hello, world")',
  language: 'javascript',
});
```

## Options

- `languageWorkers` (`string[]`) - include only a subset of the languageWorkers supported.

  - default value: ['editorWorkerService', 'css', 'html', 'json', 'typescript'].
  - Assuming only use css worker(editorWorkerService is must include base worker), you can set ['editorWorkerService', 'css']

- `customWorkers` (`IWorkerDefinition[]`) - include your custom worker.

  - default value: []
  - example value: `[{label: "graphql", entry: "monaco-graphql/esm/graphql.worker"}]`, The `entry` is relative path in the node_modules Or you can set absolute path.

- `publicPath` (`string`) - custom public path for worker scripts, overrides the public path from which files generated by this plugin will be served. Or you can set CDN e.g `https://unpkg.com/vite-plugin-monaco-editor@1.0.5/cdn`

  - default value: `monacoeditorwork`

- `globalAPI` (`boolean`) - specifies whether the editor API should be exposed through a global `monaco` object or not. This option is applicable to `0.22.0` and newer version of `monaco-editor`. Since `0.22.0`, the ESM version of the monaco editor does no longer define a global `monaco` object unless `global.MonacoEnvironment = { globalAPI: true }` is set ([change log](https://github.com/microsoft/monaco-editor/blob/main/CHANGELOG.md#0220-29012021)).
  - default value: `false`.
- `customDistPath` (`(root: string, buildOutDir: string, base: string) => string`) - Custom callback returns the worker path
- `forceBuildCDN` (`boolean`) - If you use CDN, the build is skipped by default. Set to true if you want to generate woker
  - default value: `false`

Some languages share the same web worker. If one of the following languages is included, you must also include the language responsible for instantiating their shared worker:

| Language   | Instantiator |
| ---------- | ------------ |
| javascript | typescript   |
| handlebars | html         |
| scss, less | css          |
