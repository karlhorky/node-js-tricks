# Node.js Tricks

A collection of useful Node.js tricks

## Run TypeScript `bin` in Node.js

To use TypeScript in Node.js 22.6.0+ in [a `bin` executable](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#bin), use a shebang with [the `--experimental-strip-types`](https://nodejs.org/en/blog/release/v22.6.0#experimental-typescript-support-via-strip-types):

`bin/index.ts`

```ts
#!/usr/bin/env -S node --experimental-strip-types

const a: number = 1;
console.log(a);
```
