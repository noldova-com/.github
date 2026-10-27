# Angular

**Scope:** How the general standards apply to Angular user interfaces.
It adds to [CODE.md](../CODE.md), [TESTING.md](../TESTING.md), [UI.md](../UI.md) and [TYPESCRIPT.md](TYPESCRIPT.md) and never relaxes them.

## Layout

- Angular parts that one Angular project compiles from source keep `src/` and `tests/` without their own `package.json`.
  Parts are not packages; within that project they import each other by path alias.
- Their `src/app` uses the TypeScript concept categories plus `components` and `directives`.
- `Component` is reserved for Angular UI building blocks.
- `component` and `directive` are final role words, like TypeScript's: `MenuTriggerDirective` lives in `menu-trigger.directive.ts`.

## Components and services

- A component pairs `<name>.component.ts` and `<name>.component.html` through `templateUrl`; `<Name>Component` lives under `src/app/components/<name>/`.
- A subordinate component shares its owner's folder, and its file name starts with the folder name, such as `menu-item.component.ts` in `menu/`.
- An `@Injectable` class named `<Name>Service` lives in `<name>.service.ts`, and every `.service.ts` file holds one.
- Styles live in `<name>.component.css` through `styleUrl`.
- Selectors start with the project's prefix (`nol` in the components library), and a feature's selectors with the project's prefix and the feature's id; complete selector names are unique.
- Templates reach the package's text through a `resources` field.
- A member that only its own class's template and host bindings use is protected, and specs reach it through the rendered element.

## Comments and API

- A part that other parts build on declares and documents its `src/api/index.ts` in a handwritten `src/api/index.d.ts` beside it, under the [API declaration rules](TYPESCRIPT.md#api-declarations); its examples import the part by its path alias.
  The declarations are compared with the implementation, as a package's are.
- A part's classes are not meant to be subclassed, so their protected members are template wiring, not API, and the declarations leave them out.

## Framework facilities

- Angular CDK provides behavior only: overlays and portals, dialogs, menus, trees and listboxes, and focus handling (focus traps, keyboard focus in lists).
  The look is always the components library's; drag uses the shared pointer drag ([UI.md](../UI.md)).
- Announcements go through one live region, CDK's `LiveAnnouncer`, which shared controls also use.

## Specs

- Specs use `.spec.ts` in place of TypeScript's `.test.ts`, mirroring paths relative to `src` beneath a sibling `tests`: `src/app/services/layout-store.service.ts` maps to `tests/app/services/layout-store.service.spec.ts`.
- A stylesheet whose behavior its package owns may have a spec that mirrors it the same way, replacing `.css` with `.spec.ts`.
- Shared fixtures use `tests/fixtures`; environment configuration lives directly under `tests`.
- Specs run through Angular's supported testing surface, with framework error propagation and DOM-state isolation.
  When they run in a real browser (Vitest in headless Chromium through Playwright), styles, layout and computed values are real.
- Install a throwing `ErrorHandler` through the unit-test builder's provider configuration.
  Unexpected framework errors fail the run; a test of an expected error asserts it explicitly.
- Specs that depend on styles, storage, preferences or document focus establish their own initial state and restore it after pending effects and fixtures are destroyed.
- Poll with Vitest's `vi.waitFor`.

## Results and coverage

- The run expects a result for every file its test target's include patterns match, and refuses a test target that excludes files, since it does not apply exclusions.
- Before preparing the project and before each run, remove the test runner's pre-bundled dependencies under `node_modules/.vite`: they are keyed by the lockfile alone, so they would otherwise keep an earlier build of the project's own packages.
- Coverage is Vitest's V8 coverage through the Angular unit-test builder, whose threshold fails the gate like the other coverage gates.
- Enum files hold no executable code after the build, which inlines their members, and are left out of the measurement.
- The compiler emits, after each class, a development-mode call that records the class's decorator metadata (`ɵsetClassMetadata`); it never runs in production.
  Mark exactly that call for the coverage to skip, and report how many were skipped.
  Besides enum files, it is the only Angular-side exclusion.
- Vitest's report keeps no skip reason, so specs are never skipped; Vitest fails a spec file without tests.
