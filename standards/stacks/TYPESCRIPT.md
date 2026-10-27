# TypeScript and Node

**Scope:** How the general standards apply to TypeScript code and Node.js tooling.
It adds to [CODE.md](../CODE.md), [TESTING.md](../TESTING.md) and [SECURITY.md](../SECURITY.md) and never relaxes them.

## Types

- Every project uses one strict compiler profile in a shared base configuration, with `exactOptionalPropertyTypes` on.
- Declare explicit access modifiers and return types.
- Never use `any`, double casts or unchecked assertions to bypass a missing model.
  An external declaration that requires `any` keeps it at that boundary; project code accepts and narrows `unknown`.
- Never bypass typing, visibility, construction, invocation or invariants with `Reflect`, `Object` APIs, casts or call indirection.
- Inside a [boundary](../CODE.md#failures), use no `typeof`, `instanceof`, `Array.isArray` or null guards on declared types, and never a non-null assertion for an invariant.
- `undefined` means absent; `null` means deliberately set to nothing; nothing is `T | null | undefined`.
  A class field whose value may be absent is `T | undefined` and assigned in the constructor; a compiled field always exists, so `?` on a field would claim an absence the object never has.
  The one exception is a field that implements an interface's optional member: it stays `?`, as the interface declares it, since `exactOptionalPropertyTypes` refuses `T | undefined` there.
  An interface or options member, or a parameter, that may be absent is optional (`?`) and is left out, never set to `undefined`.
  A parameter that must be passed but whose value may be absent is `T | undefined`; `?` is only for trailing arguments that may be left out.
  A wire model leaves out the key of an absent value and writes `null` as `null`; reading keeps the distinction.
- Declare every public interface and type alias in its owning source.
- Express immutability with `readonly`; never freeze objects at runtime.

## Classes

- Declare fields explicitly; never use constructor parameter properties.
- In a constructor, separate the checks (including `super`) from the assignments with one blank line; omit it when there are no checks.
- Declaration merging and prototype additions are confined to the project's foundation packages, in `.extensions.ts` files, each package adding the members of its own subject, such as character tests in the text package.
  An extension works only on `this` and its parameters, adds no fields or hidden state, never uses declaration merging to carry state, and uses the facts its type's owner provides instead of finding them again.
- Each wire model has `toJson()`, returning explicitly typed plain JSON, and `static fromJson(value: unknown)`, which reads through a validating JSON reader.
- In code that ships as minified JavaScript, each concrete exception sets `name` to its own class name, written out, because a minified build renames classes and the name is what a log shows:

```ts
public override readonly name: string = "ArgumentException";
```

  Unbundled scripts may take the name from the class (`this.name = ApiException.name`).
  CONTEXT's packages take it from the class (`new.target.name`), since CONTEXT keeps class names.
- Member order: public static fields first, then the other fields grouped by visibility, private, then protected, then public, statics before instance members within a group; then the constructor; then accessors and public methods; then protected methods; then private methods.
  A public static field that builds itself from its own class stays after the statics it needs, because static fields initialize in declaration order.
  Fields of one group have no blank line between them; one blank line separates groups and every other member.
  Enum members have no blank lines between them.

## Canonical values

- Each package has one `src/resources.ts` holding one `Resources` class with camelCase `static readonly` members; no resource directories or parallel classes.
  Methods that compose a message start with `format` and combine owned text with runtime values.
- Code left out of packaged builds, such as a component catalog, keeps its strings in its own resources class beside it, reached only from that code, because a bundler cannot drop members of a class the packaged build still uses.
- Empty values, ES module specifiers, declarations and configuration keep their required language and toolchain form.
- Scripts keep their constants on their owning class, without a shared scripts resources file.

## Files and folders

- A repository's packages live under its `src/`, each in a folder named after the package: `Noldova.<Product>.<Name>`, in PascalCase segments (`Noldova.Example.Shell`).
  A further segment is added only when the responsibility is nested inside a broader one (`Noldova.Example.Shell.Windows`), never for folder convenience.
- A package's `package.json` sits at its root, with `src/` and `tests/` beside it, each with its own TypeScript configuration; `resources.ts` sits at the root of `src/`:

```text
src/Noldova.Example.Shell/
  package.json
  src/
    api/index.ts
    api/index.d.ts
    resources.ts
    models/
    services/
  tests/
    services/
```

- Source trees use the concept categories `api`, `decorators`, `enums`, `exceptions`, `extensions`, `interfaces`, `intrinsics`, `models`, `services` and `types`; create only those the package uses.
  Domain subfolders are added only when they make current navigation clearer.
- Models hold state, identities, options and results; services own operations such as tracking, parsing, reading, writing and verifying.
  A service holds no data of its own: it may hold the collaborators it calls, and a service that drives one operation, such as a reader, holds that operation's position.
  Data a caller reads back, and state shared across operations, belong to a model.
  Decorators hold decorators, extensions hold the foundation's additions to existing types, intrinsics hold language-level primitives such as `nameof`.
- Interfaces live beneath `interfaces/`, enums beneath `enums/`, exceptions beneath `exceptions/` and type aliases beneath `types/`, and each of these folders holds only its kind.
  Scripts use domain folders, so only the interfaces rule applies there.
- Every module exports by name, scripts included; never `export default`.
  The only exception is a file that an outside tool loads through its default export, such as a test runner's configuration or a test reporter.
  An import from an outside module follows what that module offers.

## Names

- Classes, enums, type aliases, decorators and other named types use descriptive PascalCase; interfaces use `I` plus PascalCase (`IProviderAdapter`).
  An interface in `declare global` or `declare module` augments an existing type and keeps its name.
- Methods, properties, parameters, locals and fields use camelCase; private fields have no leading underscore.
- Generic parameters use `T` or `T` plus a role (`TKey`, `TValue`).
- Internal constants use UPPER_SNAKE_CASE; `Resources` members and state fields stay camelCase.
- Enum members use PascalCase; enum literals stay out of `Resources`.
  API declarations mirror enum values exactly, and tests keep independent expected literals.
- Concept folders and file names use lowercase kebab-case, and a file name preserves the words of its type's name; an interface's `I` is one of its words (`i-provider-adapter.ts`).
  Product names keep their one-word spelling: `GitHubApi` lives in `github-api.ts`.
- A final role word takes a dot: `node`, `parser`, `lexer`, `reader`, `validator`, `exception` or `service` (`argument.exception.ts`, `source.reader.ts`, `binary-search.service.ts`).
  Other words stay kebab-case (`snapshot-tracker.ts`); a single-word type keeps its name (`exception.ts`); never add `.model`, `.enum` or `.interface`.
- Exception class names end in `Exception` (`ArgumentException`).
- Supporting files use their kind suffix: `.test.ts`, `.extensions.ts`, `.fixture.ts` and `.d.ts`.
  Entry points (`api/index.ts`), `resources.ts`, manifests, TypeScript configuration and the scripts `package.json` runs keep their fixed names.

## Formatting

- Two-space indentation and semicolons.
- Handwritten TypeScript prefers 160 columns as a soft signal, not a gate; a cohesive signature, indivisible string, pattern, URL or test input may exceed it.
- Omit braces around a control-flow body of exactly one statement; use them for several, and keep them where leaving them out would attach a following `else` to a nested `if`.
- Name a single untyped arrow parameter `t` and omit its parentheses: `values.filter(t => t.isValid)`.
  Keep parentheses for zero or several parameters, type annotations, destructuring, defaults or rest parameters.
- Multiline parameter and argument lists have no trailing comma; the closing parenthesis follows the last item:

```ts
public constructor(
  name: string,
  value: number) {
}
```

- Imports are grouped by origin, with one blank line between non-empty groups and none within a group:
  1. host built-in modules, such as `node:fs/promises`;
  2. package imports, scoped and external;
  3. relative imports.
  Type-only, value, namespace and side-effect imports belong to their origin group.
  Within a group, sort declarations and named imports as VS Code's Organize Imports does.
  Required side-effect initialization order takes precedence; formatting never changes initialization behavior.

## API declarations

- Each package exposes its API through `src/api/index.ts` and declares and documents it in a handwritten `src/api/index.d.ts`, the only files with doc comments.
- Every public type, member and overload has JSDoc:
  - `@param name description` for each parameter, without a hyphen: purpose, accepted inputs, constraints, units, defaults and callback obligations, beyond its name and type;
  - `@returns` for non-void results: meaning, ownership, ordering, absence and asynchronous completion; omitted for constructors; a `never` result or an `asserts` signature needs `@throws` and may also have `@returns`;
  - `@throws` for contractual failures: type, trigger, and synchronous throw versus promise rejection;
  - `@example` for normal and boundary use, as a ```` ```ts ```` block that imports the package by name as a consumer does; an expected-error line is marked `// @ts-expect-error`.
  Overloads share their examples; every public callable has one.
- Document generic roles, property meaning, mutation, lifecycle, performance and deprecation; use `@remarks` and `{@link}` where useful.
- Every JSDoc uses the multiline form with separate opening and closing lines.
- Checks refuse an empty JSDoc, an empty tag, an unresolved `{@link}` or `@throws` type, and declarations that differ from the implementation in exports, constructors, types, optionality, generics, overloads, visibility or returns.
- A package's classes can be extended, so it declares their protected members.
- An export that only development builds or generated source use stays out of `src/api/index.ts` and has its own entry.

## Tests

- Tests mirror source paths under `tests/`, replacing `.ts` with `.test.ts` and keeping role suffixes:

```text
src/services/snapshot-tracker.ts
tests/services/snapshot-tracker.test.ts
```

- A test of what a module leaves out of its exports imports the module as a namespace (`import * as api from "…"`) and checks the names it holds.
- Code that a mock clock drives calls `timers.setTimeout` through the default import of `node:timers/promises`, because the mock replaces the module's function, not a named import's binding.

## Scripts and Node

- The company uses one Node version everywhere: every repository's `engines`, every workflow's `node-version`, every test machine and every member's machine.
  Only a maintainer changes it, and then every repository moves together.
- npm is the only package manager.
- Standalone automation is TypeScript; npm commands expose TypeScript entry points with arguments, not inline shell.
  Node runs the entry points directly, with no TypeScript runner while it can.
- Scripts have no tests and no coverage requirement; the packages they build are tested.

## Packages

- Each package has its own `package.json` beside its source, with its name, version and entry points.
  Its name is the scope followed by its folder's segments after `Noldova`, lowercase and joined by hyphens: `Noldova.Context.Text` is `@noldova/context-text`.
  A public repository's packages take the `@noldova/` scope on npm's registry, and a private one's `@noldova-com/` on GitHub's.
- The project's own scripts build the packages: each is built into `_build/packages/<folder name>`, packed with `npm pack` into `_packages/`, and installed from there with `npm install --no-save`, so the repository's own packages never enter its manifest or lockfile.
  No third-party build or packaging tool does this.
- A package's public test helpers live in `src/testing/`, a source tree beside `src/app/` with the same concept categories.
  Only the package's testing entry, `src/api/testing.ts`, reaches them, so an app's bundle never takes them, and no other entry imports them or the test runner.
- A package that needs a compiler beyond `tsc`, such as Angular's compiler for components, keeps its own build script beside it, which the shared build calls in place of `tsc`.
- Everything that uses a package, other packages, apps and tests, installs its packed `.tgz`, exactly as an outside user would; never a path alias or a relative import into another package's source.
- `_build/` and `_packages/`, the repository's own packed output, are not committed.
  The lockfile is committed, and it pins every third-party package, so every install gets the same third-party code.
- A repository that uses another repository's packages installs their published versions from the registry they are published to at exact pins, like any external dependency; never committed `.tgz` copies, fetched files or `file:` paths.

## Dependencies

- External dependency versions are exact pins from the registry, never ranges, tags, aliases, paths, Git or URL sources.
- Each folder with a lockfile keeps `save-exact=true` in its `.npmrc`.
- A shipped package's third-party runtime dependency is pinned at the same version in the manifest of the folder that installs it, usually the repository's root, and locked at it in that folder's lockfile.
