telecommunications smart home technology # @amamo/oxlint-config

English | [简体中文](./README-zh.md) | [Rule Explorer](https://jikkai.github.io/oxlint-config/)

## Introduction

`@amamo/oxlint-config` is an opinionated set of composable Oxlint presets with a safe Oxc
initializer. The default export builds an `OxlintConfig`; named exports expose the same stable
fragments for advanced composition.

```ts
import amamo from "@amamo/oxlint-config";

export default amamo();
```

The package configures linting. Oxfmt remains a separate formatter, and the optional initializer
can install both tools and prepare a project without replacing an existing lint configuration.

## Features and Design Choices

- One factory for the common path, plus named stable fragments for manual composition.
- A default baseline for Oxlint's native Base, TypeScript, Import, and Promise plugins.
- Explicit React, Next.js, Vue, Tailwind CSS, Node, Jest, Vitest, JSDoc, and React Perf integrations.
- Optional type-aware linting through `oxlint-tsgolint`.
- Opt-in adapters for seven ESLint-compatible JavaScript plugins.
- A project initializer that detects dependency and config signals, previews changes, preserves
  conflicts, and writes only after validation.
- A bilingual [Rule Explorer](https://jikkai.github.io/oxlint-config/) for inspecting the actual
  generated presets, scopes, severities, options, schemas, and official rule links.

> [!WARNING]
> This is a personal, opinionated configuration. Review changes when upgrading. If its decisions
> do not fit your project, override the relevant rules or fork the configuration. Native preset
> rules do not have a separate stability promise beyond the package's published version, and the
> JavaScript plugin adapters are explicitly Alpha.

## Requirements and Compatibility

| Requirement          | Supported range   | Notes                                                                   |
| -------------------- | ----------------- | ----------------------------------------------------------------------- |
| Node.js              | `>=22.18`         | Enforced by this package's `engines` field.                             |
| `oxlint`             | `>=1.80.0 <2.0.0` | Required peer dependency and source of native plugins.                  |
| `oxfmt`              | `>=0.1.0 <1.0.0`  | Required peer dependency used by the initializer's formatting workflow. |
| `oxlint-tailwindcss` | `^1.7.1`          | Optional; required only when `tailwindcss` is configured.               |
| `oxlint-tsgolint`    | `^7.0.2001`       | Optional; required only when `typeAware` is enabled.                    |

The initializer supports pnpm, npm, Yarn, and Bun projects when it can identify one package
manager safely. Repository development uses pnpm `11.18.0`.

For upstream behavior and supported syntax, consult the official
[Oxlint documentation](https://oxc.rs/docs/guide/usage/linter.html) and
[Oxfmt documentation](https://oxc.rs/docs/guide/usage/formatter.html).

## Starter Wizard

Run the initializer from the project root:

```sh
pnpm dlx @amamo/oxlint-config init
```

The interactive flow shows detected manifests, stable ecosystems, experimental packages, package
manager, planned files, install command, notices, and conflicts before asking for confirmation.
Start with a dry run when evaluating it in an existing project:

```sh
pnpm dlx @amamo/oxlint-config init --dry-run --yes
```

Non-interactive use requires `--yes`. The initializer is a setup tool; `amamo()` itself does not
perform runtime framework detection.

## Manual Installation

Install the package with its required Oxlint and Oxfmt peers:

```sh
# pnpm
pnpm add --save-dev @amamo/oxlint-config oxlint oxfmt

# npm
npm install --save-dev @amamo/oxlint-config oxlint oxfmt

# Yarn
yarn add --dev @amamo/oxlint-config oxlint oxfmt

# Bun
bun add --dev @amamo/oxlint-config oxlint oxfmt
```

Create `oxlint.config.ts` in the project root:

```ts
import amamo from "@amamo/oxlint-config";

export default amamo();
```

Framework and test presets are explicit:

```ts
import amamo from "@amamo/oxlint-config";

export default amamo({
  nextjs: true,
  node: true,
  reactPerf: true,
  test: ["jest", "vitest"],
});
```

## Recommended Package Scripts

The initializer adds only missing script keys and preserves existing values:

```json
{
  "scripts": {
    "lint": "oxlint .",
    "lint:fix": "oxlint --fix .",
    "format": "oxfmt .",
    "format:check": "oxfmt --check ."
  }
}
```

Use `lint` in CI for diagnostics and `format:check` to verify formatting without changing files.
`lint:fix` applies fixes supported by Oxlint; `format` runs Oxfmt independently.

## Default Behavior

`amamo()` always starts with Base and enables TypeScript, Imports, and Promise unless explicitly
disabled. Other integrations are opt-in, with React and Next.js dependency rules shown below.

| Preset or mode               | Default               | Selection behavior                                                                 |
| ---------------------------- | --------------------- | ---------------------------------------------------------------------------------- |
| Base                         | Always on             | Enables the `eslint`, `oxc`, and `unicorn` native plugins and category severities. |
| TypeScript                   | On                    | Disable with `typescript: false`.                                                  |
| Imports                      | On                    | Disable with `imports: false`.                                                     |
| Promise                      | On                    | Disable with `promise: false`.                                                     |
| React                        | Off                   | Enable with `react: true`; also enables JSX A11y by default.                       |
| Next.js                      | Off                   | Enable with `nextjs: true`; also selects React and JSX A11y.                       |
| JSX A11y                     | Follows React/Next.js | Set `jsxA11y` explicitly to override the derived value.                            |
| Vue, Node, JSDoc, React Perf | Off                   | Enable each with its matching boolean option.                                      |
| Tailwind CSS                 | Off                   | Pass its required Tailwind CSS v4 entry point through `tailwindcss`.               |
| Jest, Vitest                 | Off                   | Select one or both through `test`. Duplicate entries are ignored.                  |
| Type-aware mode              | Off                   | Enable with `typeAware: true` after installing `oxlint-tsgolint`.                  |
| Experimental adapters        | Off                   | Each adapter must be installed and explicitly selected.                            |

Preset order is deterministic: Base, default native fragments, React-related fragments, Vue,
Tailwind CSS, Node, JSDoc, tests, experimental adapters, `options.rules`, then arbitrary trailing
overrides.

## Options Reference

The default export accepts `IAmamoOptions` followed by any number of `OxlintConfig` overrides:

```ts
function amamo(options?: IAmamoOptions, ...overrides: OxlintConfig[]): OxlintConfig;
```

| Option         | Type                                                             | Default             | Effect                                                        |
| -------------- | ---------------------------------------------------------------- | ------------------- | ------------------------------------------------------------- |
| `experimental` | `IExperimentalOptions`                                           | `{}`                | Selects installed Alpha JavaScript plugin adapters.           |
| `ignores`      | `string[]`                                                       | `undefined`         | Sets the root `ignorePatterns`.                               |
| `imports`      | `boolean`                                                        | `true`              | Selects the native Import plugin fragment.                    |
| `jsdoc`        | `boolean`                                                        | `false`             | Selects the JSDoc fragment for script files.                  |
| `jsxA11y`      | `boolean`                                                        | React/Next.js state | Selects or suppresses JSX accessibility rules.                |
| `nextjs`       | `boolean`                                                        | `false`             | Selects Next.js and implies React.                            |
| `node`         | `boolean`                                                        | `false`             | Selects Node rules and the Node environment for script files. |
| `promise`      | `boolean`                                                        | `true`              | Selects Promise rules for script files.                       |
| `react`        | `boolean`                                                        | `false`             | Selects React rules and implies JSX A11y.                     |
| `reactPerf`    | `boolean`                                                        | `false`             | Selects explicit React allocation warnings.                   |
| `rules`        | `OxlintConfig["rules"]`                                          | `undefined`         | Adds root rule settings after selected presets.               |
| `tailwindcss`  | `ITailwindcssOptions`                                            | `undefined`         | Enables `oxlint-tailwindcss` with an explicit v4 entry point. |
| `test`         | `"jest" \| "vitest" \| readonly ("jest" \| "vitest")[] \| false` | `false`             | Selects one or both test-runner fragments.                    |
| `typeAware`    | `boolean`                                                        | `false`             | Sets root `options.typeAware`.                                |
| `typescript`   | `boolean`                                                        | `true`              | Selects the TypeScript fragment.                              |
| `vue`          | `boolean`                                                        | `false`             | Selects Vue script-block rules.                               |

`IExperimentalOptions` has seven optional boolean keys, all disabled by default: `cypress`,
`mocha`, `playwright`, `regexp`, `sonarjs`, `storybook`, and `testingLibrary`.

## Rules, Overrides, and Ignores

Use `rules` for ordinary root-level changes:

```ts
import amamo from "@amamo/oxlint-config";

export default amamo({
  rules: {
    eqeqeq: "error",
    "typescript/consistent-type-imports": "off",
  },
});
```

Every additional argument is an arbitrary `OxlintConfig` applied last. This is the escape hatch for
settings, environments, globals, plugin configuration, or imported third-party config objects:

```ts
import amamo from "@amamo/oxlint-config";

export default amamo(
  { rules: { "no-console": "warn" } },
  {
    rules: { "no-console": "off" },
    settings: { react: { linkComponents: [{ name: "Link", linkAttribute: "to" }] } },
  },
);
```

For file-specific behavior, pass an Oxlint override with an explicit `files` scope:

```ts
import amamo from "@amamo/oxlint-config";

export default amamo(
  {},
  {
    overrides: [
      {
        files: ["scripts/**/*.ts"],
        rules: { "no-console": "off" },
      },
    ],
  },
);
```

Set project-level ignores through the first argument:

```ts
export default amamo({ ignores: ["dist/**", "coverage/**", "generated/**"] });
```

See the official [Oxlint configuration guide](https://oxc.rs/docs/guide/usage/linter/config.html)
for the full `OxlintConfig` surface.

## Named Stable Exports

Named exports are the same fragments used by `amamo()`. Import them when you need manual
composition; the default factory is simpler for most projects.

| Export       | Scope and explicit behavior                                                                                                                   |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `base`       | Native `eslint`, `oxc`, and `unicorn`; `correctness` errors, `perf` and `suspicious` warnings, other categories off.                          |
| `typescript` | TypeScript files; enforces separate type-only imports and rejects duplicate imports while allowing one type-only import from the same module. |
| `imports`    | Enables the native `import` plugin.                                                                                                           |
| `promise`    | Script files; `catch-or-return` and `no-return-wrap` errors.                                                                                  |
| `react`      | Script files; Hooks rules with exhaustive dependencies as a warning; supports the automatic JSX runtime.                                      |
| `reactPerf`  | JSX/TSX; warns about new JSX, array, function, and object values passed as props.                                                             |
| `nextjs`     | JSX/TSX; async client component and duplicate head errors, HTML link warning.                                                                 |
| `jsxA11y`    | JSX/TSX; alt text, ARIA props/roles, and anchor validity checks.                                                                              |
| `vue`        | `*.vue`; Vue environment plus duplicate key, computed side effect, and `defineProps` validation rules.                                        |
| `node`       | Script files; Node environment, exports assignment error, and path concatenation warning.                                                     |
| `jsdoc`      | Script files; warns on invalid JSDoc tag names.                                                                                               |
| `jest`       | Test files; Jest environment, focused-test error, disabled-test and missing-expect warnings.                                                  |
| `vitest`     | Test files; Vitest environment, focused-test error, disabled-test and missing-expect warnings.                                                |

```ts
import { base, node, typescript } from "@amamo/oxlint-config";
import { defineConfig } from "oxlint";

export default defineConfig({ extends: [base, typescript, node] });
```

## Ecosystem Presets

### React

`react: true` applies Hooks rules to script files, supports the automatic JSX runtime, and enables
JSX A11y unless `jsxA11y: false` is explicit.

### Next.js

`nextjs: true` selects React first, then Next.js, and derives JSX A11y. It does not inspect the
runtime; the option or initializer-generated config is the source of truth.

### Vue

`vue: true` selects Oxlint's native Vue plugin for `*.vue` script content and enables the Vue
environment. It does not add template-specific JavaScript-plugin rules.

### Tailwind CSS

Install the optional plugin and pass the CSS file that imports Tailwind CSS v4:

```sh
pnpm add --save-dev oxlint-tailwindcss
```

```ts
import amamo from "@amamo/oxlint-config";

export default amamo({
  tailwindcss: { entryPoint: "src/index.css" },
});
```

For monorepos, `entryPoint` also accepts ordered `{ files, use }` mappings. The preset enables a
focused set of the plugin's rules; use `rules` for severity changes. Tailwind CSS v3 is not supported.
See the [oxlint-tailwindcss setup guide](https://oxlint-tailwindcss.pages.dev/setup). Because it runs
through Oxlint's JavaScript plugin API, that upstream API remains Alpha.

### Node

`node: true` applies the Node environment and Node rules to JavaScript and TypeScript script-file
patterns, including ESM and CommonJS extensions.

### Jest

Use `test: "jest"` for Jest globals and rules on `*.test.*`, `*.spec.*`, and `__tests__` script
files.

### Vitest

Use `test: "vitest"` for the equivalent Vitest environment and focused/disabled/expect checks.
Select both runners with `test: ["jest", "vitest"]`.

### JSDoc

`jsdoc: true` enables the native JSDoc plugin and its tag-name validation warning on script files.

### React Perf

`reactPerf: true` enables four allocation-related warnings for JSX/TSX props. It is independent of
`react`; enable the React preset too when the project uses React Hooks rules.

```ts
export default amamo({
  jsdoc: true,
  nextjs: true,
  node: true,
  reactPerf: true,
  test: ["jest", "vitest"],
  vue: true,
});
```

## Type-aware Linting

Install the optional engine within its tested peer range:

```sh
pnpm add --save-dev oxlint-tsgolint
```

Then enable the root Oxlint option:

```ts
import amamo from "@amamo/oxlint-config";

export default amamo({ typeAware: true });
```

This option sets `options.typeAware: true`; it does not add a separate exported preset fragment.
Keep TypeScript enabled, and configure any additional type-aware rules through `rules` or a trailing
override. Type-aware linting requires `oxlint-tsgolint`; follow the official
[type-aware linting guide](https://oxc.rs/docs/guide/usage/linter/type-aware.html) for upstream
TypeScript compatibility, supported rules, diagnostics, and performance notes.

## Experimental JavaScript Plugins

Oxlint's ESLint-compatible JavaScript plugin API is Alpha. These adapters are opt-in, their peer
dependencies are optional, and this package does not promise SemVer stability for their behavior or
compatibility. Install only the packages you enable.

| Option           | Package                         | Tested peer range | Scoped rule                                                                 |
| ---------------- | ------------------------------- | ----------------- | --------------------------------------------------------------------------- |
| `cypress`        | `eslint-plugin-cypress`         | `^6.4.3`          | `cypress/no-assigning-return-values` as an error in Cypress files.          |
| `mocha`          | `eslint-plugin-mocha`           | `^12.0.1`         | `mocha/no-exclusive-tests` as a warning in test files.                      |
| `playwright`     | `eslint-plugin-playwright`      | `^2.11.0`         | `playwright/no-focused-test` as an error in test files.                     |
| `regexp`         | `eslint-plugin-regexp`          | `^3.1.1`          | `regexp/no-dupe-characters-character-class` as an error in script files.    |
| `sonarjs`        | `eslint-plugin-sonarjs`         | `^4.2.0`          | `sonarjs/no-all-duplicated-branches` as an error in script files.           |
| `storybook`      | `eslint-plugin-storybook`       | `^10.5.5`         | `storybook/default-exports` as an error in stories and `.storybook` files.  |
| `testingLibrary` | `eslint-plugin-testing-library` | `^7.16.2`         | `testing-library/no-global-regexp-flag-in-query` as an error in test files. |

For example:

```sh
pnpm add --save-dev eslint-plugin-playwright eslint-plugin-testing-library
```

```ts
import amamo from "@amamo/oxlint-config";

export default amamo({
  experimental: {
    playwright: true,
    testingLibrary: true,
  },
});
```

The initializer detects already-installed experimental packages but keeps every adapter off by
default, including under `--yes`. Interactive users must opt in. See the official
[Oxlint JavaScript plugin guide](https://oxc.rs/docs/guide/usage/linter/js-plugins) for current
upstream API support and limitations.

## Initializer Detection and Write Safety

### Detection

The initializer reads the root manifest plus ordinary package.json or pnpm workspace manifests.
It derives project choices from dependencies and a small set of config signals:

- TypeScript from a `typescript` dependency, a root `tsconfig.json`, or a root
  `tsconfig.*.json`.
- React, Next.js, Vue, Jest, and Vitest from their package dependencies.
- Tailwind CSS from one unambiguous project CSS file importing `"tailwindcss"`; multiple or missing
  v4 entry points are reported and left for manual configuration.
- Node from the root `engines.node` field or known server packages (`express`, `fastify`, `hono`,
  `koa`, or `@nestjs/core`). `@types/node` alone does not enable Node rules.
- Storybook from `storybook`, `eslint-plugin-storybook`, or a package under `@storybook/`; other
  experimental adapters from their exact plugin packages.
- The package manager from the invoking user agent or one unambiguous lockfile.

This is initialization-time detection only. Multiple package-manager lockfiles, a missing root
`package.json`, or a required install without a safely detected manager stops the command.

### Flags

| Flag           | Behavior                                                                      |
| -------------- | ----------------------------------------------------------------------------- |
| `--dry-run`    | Prints detection and the complete plan; starts no install and writes no file. |
| `--no-install` | Writes an approved safe plan without starting a package manager.              |
| `--yes`        | Accepts detected stable choices and safe defaults for non-interactive use.    |

Only `init`, `--dry-run`, `--no-install`, and `--yes` are accepted. Unknown, repeated, or valued
boolean flags are rejected with usage text.

### Generated and Updated Files

| Target                    | Behavior                                                                                                           |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `oxlint.config.ts`        | Created only when `.oxlintrc.json`, `.oxlintrc.jsonc`, `oxlint.config.ts`, and `oxlint.config.mts` are all absent. |
| `package.json`            | Adds only missing `lint`, `lint:fix`, `format`, and `format:check` scripts.                                        |
| `.vscode/settings.json`   | Adds Oxfmt then Oxlint save actions only when they can be merged safely.                                           |
| `.vscode/extensions.json` | Adds the `oxc.oxc-vscode` recommendation without removing existing entries.                                        |
| `.zed/settings.json`      | Adds Oxfmt formatting and safe Oxlint fixes on save for languages supported by the Oxc extension.                  |
| Development dependencies  | Installs the package, Oxlint, Oxfmt, and only selected optional engines/adapters.                                  |

If a lint config already exists, it is preserved and the command prints a composition snippet
instead. JSONC edits keep comments and existing values where possible; incompatible values are
reported as conflicts rather than overwritten.

### Conflicts and Write Safety

The plan is assembled before execution. Install failures write no planned files. Before writing,
the initializer validates that every target is unique, inside the physical project root, and not a
symbolic link. It snapshots existing content, rechecks hashes and missing targets, writes through an
exclusively created temporary file, preserves existing modes, and renames only after the final
check. Concurrent or unsafe changes stop the operation and leave remaining paths unwritten.

Package-manager edits to `package.json` are refreshed after installation so new dependencies are
preserved; newly conflicting scripts remain user-owned and are reported. A second run over the
managed result plans no duplicate changes.

## VS Code Save Formatting and Safe Fixes

Install the official [Oxc VS Code extension](https://marketplace.visualstudio.com/items?itemName=oxc.oxc-vscode).
The initializer recommends it and writes Oxfmt before Oxlint's safe fixes in the ordered save-action
object:

```json
{
  "editor.formatOnSave": false,
  "editor.codeActionsOnSave": {
    "source.format.oxc": "always",
    "source.fixAll.oxc": "always"
  }
}
```

This provides save formatting through Oxfmt and then applies Oxlint fixes. If both keys already
exist in reverse order, the initializer reports a conflict and preserves the file byte-for-byte.
Editor type-aware behavior follows the root Oxlint config; see the official
[editor setup guide](https://oxc.rs/docs/guide/usage/linter/editors.html).

## Zed Save Formatting and Safe Fixes

The initializer asks Zed to install the [Oxc extension](https://zed.dev/extensions/oxc) and writes
project settings for every language it supports. JavaScript, JSX, TypeScript, TSX, Vue, and Svelte
run Oxfmt followed by safe Oxlint fixes; Astro runs only the supported Oxlint fixes. Other supported
data and markup languages run Oxfmt only. Existing settings are merged without replacing user
values.

```json
{
  "languages": {
    "TypeScript": {
      "format_on_save": "on",
      "formatter": [
        { "language_server": { "name": "oxfmt" } },
        { "code_action": "source.fixAll.oxc" }
      ]
    }
  }
}
```

The initializer preserves the file and reports a conflict when an existing `fixKind` permits unsafe
fixes or an existing formatter sequence runs Oxlint before Oxfmt.

## Oxfmt and Prettier Boundary

This package exports lint configuration, not a formatter preset. The initializer installs Oxfmt,
adds `format` scripts, and otherwise uses Oxfmt's own defaults. Create an Oxfmt config only when the
project needs formatter-specific customization; see the official
[Oxfmt configuration reference](https://oxc.rs/docs/guide/usage/formatter/config.html).

Oxfmt and Prettier are independent formatters. Avoid running both over the same files on save or in
CI unless their responsibilities are deliberately separated. This package does not disable
Prettier, migrate its config, emulate Prettier plugins, or add formatting rules to Oxlint. Use the
official [Prettier migration guide](https://oxc.rs/docs/guide/usage/formatter/migrate-from-prettier.html)
when moving an existing project.

## Svelte and Astro Script-block Limitations

Oxlint can lint JavaScript and TypeScript inside Svelte and Astro script blocks. This package does
not select Svelte- or Astro-specific presets, add template rules, or promise linting for markup and
styles. Oxfmt supports Svelte formatting; this package makes no claim about Astro formatting.

Check the current upstream support in the official
[Oxlint guide](https://oxc.rs/docs/guide/usage/linter.html) before adopting it for framework files.

## Rule Explorer

The static [Rule Explorer](https://jikkai.github.io/oxlint-config/) is generated from the built
package, the installed Oxlint schema and rule inventory, controlled `--print-config` profiles, and
the package's experimental fixture. It does not evaluate uploaded or user-supplied configuration.

It provides:

- Default and All Stable configuration controls, including dependencies and Alpha labels.
- Enabled and All rule views, where All adds disabled native inventory to active applications.
- Search plus plugin, preset, severity, state, and scope filters.
- Rules, Presets, Scopes, and generated Config views.
- Rule severities, source preset/plugin, scopes, options, native schemas, external-plugin
  descriptions, and official documentation links.
- English and Chinese UI, shareable URL state, browser history restoration, and desktop/mobile
  layouts.

The build snapshot is deterministic and contains package/Oxlint versions but no generation
timestamp.

## Compatibility and Stability Policy

- Supported package compatibility is the Node engine and peer ranges published in `package.json`.
- Native preset behavior follows the exact package version. Review rule changes on every upgrade;
  no additional stable-rule SemVer policy is claimed.
- JavaScript plugin adapters and Oxlint's JS plugin API are Alpha. Adapter compatibility is tested
  only against the peer ranges listed above.
- Tailwind CSS linting supports v4 only and depends on `oxlint-tailwindcss` plus Oxlint's Alpha JS
  plugin API.
- Type-aware mode depends on `oxlint-tsgolint` and upstream TypeScript compatibility.
- The initializer preserves conflicts instead of making destructive guesses, but review every plan
  before approving writes in an established repository.

## FAQ

### Does `amamo()` auto-detect my framework?

No. Frameworks and tests are explicit factory options. Only the one-time initializer inspects
dependencies and config files to propose defaults.

### Can I override any rule or add another Oxlint config?

Yes. Use `options.rules` for root rules and trailing `OxlintConfig` arguments for arbitrary or
file-scoped composition. Trailing overrides are applied last.

### Can Jest and Vitest be enabled together?

Yes. Use `test: ["jest", "vitest"]`. Repeated values are deduplicated; when both are enabled, Jest
is applied before Vitest.

### Why is JSX A11y active when I selected React or Next.js?

React derives `jsxA11y: true`, and Next.js derives React. Set `jsxA11y: false` explicitly to disable
that dependency.

### Does this package format code?

No. It configures Oxlint. Oxfmt is installed and invoked separately by the recommended workflow and
editor save actions.

### Will the initializer overwrite my existing config or scripts?

No. It preserves recognized lint configs, prints a composition snippet, adds only missing script
keys, and reports conflicting values. Unsafe or concurrently changed targets abort the write.

### Why is an experimental rule unavailable?

Install the matching optional peer in its tested range and enable its `experimental` key. These
adapters are never enabled by default.

## Local Development and Verification

Install dependencies and build the root package:

```sh
pnpm install
pnpm build
```

Run the repository's complete package verification entry:

```sh
pnpm check
```

`pnpm check` runs `check:static` followed by the source test suite. `check:static` checks formatting
with Oxfmt, lints with Oxlint, and type-checks with TypeScript; `pnpm test` runs Vitest under `src`.

Documentation commands are separate from package verification:

```sh
pnpm docs:build
pnpm docs:dev
```

`docs:build` builds the package and static site.

`.github/workflows/docs.yml` deploys the exact commit from a successful same-repository `Release`
workflow run. An explicit manual dispatch deploys `github.sha`, the commit resolved from the ref
selected by the operator. A repository administrator must select **GitHub Actions** as the Pages
source once under **Settings → Pages**; the workflow does not change that setting.

## License

[MIT](./LICENSE) © 白熱
