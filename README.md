# Node.js Tricks

A collection of useful Node.js tricks

## Run TypeScript `bin` executable in Node.js

To use TypeScript in Node.js >=v22.6.0 in [a `bin` executable](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#bin), use a shebang with [the `--experimental-strip-types` flag](https://nodejs.org/en/blog/release/v22.6.0#experimental-typescript-support-via-strip-types):

`bin/index.ts`

```ts
#!/usr/bin/env -S node --experimental-strip-types

const a: number = 1;
console.log(a);
```

However, [this will fail if the file is contained within `node_modules`](https://github.com/nodejs/typescript/issues/14), confirmed with Node.js <=v22.18.0. This `node_modules` ban change in the future.

For running `.ts` files in `node_modules`, consider using a Bash script `bin` executable to strip types in `node_modules` on the fly using Node.js built-in type stripping support via [`registerHooks`](https://nodejs.org/api/module.html#moduleregisterhooksoptions) and [`stripTypeScriptTypes()`](https://nodejs.org/api/module.html#modulestriptypescripttypescode-options) from `node:module`:

`bin/index.sh`

```bash
#!/usr/bin/env bash

set -o errexit
set -o nounset
set -o pipefail

exec node --disable-warning=ExperimentalWarning --input-type=module --eval '
import { registerHooks, stripTypeScriptTypes } from "node:module";
import { dirname, resolve } from "node:path";
import { argv } from "node:process";
import { fileURLToPath, pathToFileURL } from "node:url";

const tsRegex = /^file:.*(?<!\.d)\.m?ts$/;

// Intercept .ts / .mts files (skipping .d.ts files) and
// transpile to JS, returning ES module
registerHooks({
  load(url, context, nextLoad) {
    if (tsRegex.test(url)) {
      return {
        format: "module",
        source: stripTypeScriptTypes(nextLoad(url).source.toString(), {
          mode: "transform",
          sourceUrl: url,
        }),
        shortCircuit: true,
      };
    }

    return nextLoad(url, context);
  },
});

await import(
  pathToFileURL(
    resolve(
      dirname(argv[1]),
      // Path to entry point
      "../src/index.ts",
    ),
  ).href
);
' "$0" "$@"
```

Another option is [`tsx`](https://tsx.is/shell-scripts):

1. Add `tsx` to your project dependendencies
2. Use `pnpm exec tsx` to execute your file using `tsx`
   ```ts
   #!/usr/bin/env -S pnpm exec tsx
   ```

Or alternatively, consider [running the executable in Bun](https://github.com/karlhorky/bun-tricks#run-typescript-bin-executable-in-bun).

## Write Zero-Dependency Tests in TypeScript

To write zero-dependency tests in TypeScript in Node.js, create a test file using `node:assert/strict` and `node:test`, eg:

`__tests__/index.test.ts`

```ts
import assert from 'node:assert/strict';
import { spawnSync } from 'node:child_process';
import { test } from 'node:test';

await test('shows process.env error messages', () => {
  const {
    stdout,
    stderr,
    status: exitCode,
  } = spawnSync('node', ['./build/upleveled-drone.mjs'], {
    encoding: 'utf-8',
  });

  assert.equal(exitCode, 1);

  assert.equal(stdout, '');
  assert.equal(
    stderr
      // GITHUB_REPOSITORY is set on GitHub Actions but not locally
      .replace('process.env.GITHUB_REPOSITORY is undefined\n', ''),
    `process.env.DRONE_TOKEN is undefined
process.env.ISSUE_NUMBER is undefined
`,
  );
});
```

And then run it using type stripping in [Node.js v22.18.0](https://nodejs.org/en/blog/release/v22.18.0) or later:

```bash
$ node __tests__/index.test.ts
```

To run the tests in watch mode, use the `--watch` flag:

```bash
$ node --watch __tests__/index.test.ts
```
