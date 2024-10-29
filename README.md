# Node.js Tricks

A collection of useful Node.js tricks

## Run TypeScript `bin` executable in Node.js

To use TypeScript in Node.js 22.6.0+ in [a `bin` executable](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#bin), use a shebang with [the `--experimental-strip-types` flag](https://nodejs.org/en/blog/release/v22.6.0#experimental-typescript-support-via-strip-types):

`bin/index.ts`

```ts
#!/usr/bin/env -S node --experimental-strip-types

const a: number = 1;
console.log(a);
```

However, [this will fail if the file is contained within `node_modules`](https://github.com/nodejs/typescript/issues/14) (as of Node.js 22.9.0).

For running `.ts` files in `node_modules`, consider [`tsx`](https://tsx.is/shell-scripts):

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

And then run it using Node.js v22.6.0+ type stripping ([the `--experimental-strip-types` flag](https://nodejs.org/en/blog/release/v22.6.0#experimental-typescript-support-via-strip-types)):

```bash
$ node --experimental-strip-types __tests__/index.test.ts
```
