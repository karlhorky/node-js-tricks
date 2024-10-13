# Node.js Tricks

A collection of useful Node.js tricks

## Run TypeScript `bin` executable in Node.js

To use TypeScript in Node.js 22.6.0+ in [a `bin` executable](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#bin), use a shebang with [the `--experimental-strip-types`](https://nodejs.org/en/blog/release/v22.6.0#experimental-typescript-support-via-strip-types):

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
