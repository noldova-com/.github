# Electron

**Scope:** How the general standards apply to Electron desktop applications.
It adds to [SECURITY.md](../SECURITY.md), [TESTING.md](../TESTING.md) and [TYPESCRIPT.md](TYPESCRIPT.md) and never relaxes them.

## Renderer security

These requirements follow [Electron's security guidance](https://www.electronjs.org/docs/latest/tutorial/security).

- Keep renderers sandboxed and context-isolated, with Node integration disabled.
- Expose a narrow preload API; keep raw Electron and Node capabilities out of renderer-facing interfaces.
- Validate IPC senders and request data in the privileged process.
- Use a restrictive Content Security Policy.
- Control navigation, window creation and permission handling.

## Main-process tests

- Main-process and preload code is tested through package tests that drive it with fakes of Electron's main-process and preload APIs.
- Only a file that runs solely inside Electron, such as the entry that composes the application from Electron's own modules, may be excluded from coverage, declared with its reason, and verified by the desktop UI workflows or a native check instead.
- Playwright starts Electron with its hang monitor disabled, so workflows cannot see a page stop responding; package tests cover that path.
  A workflow that stops a page ends its renderer process itself rather than asking Electron to crash it, which some targets do not do reliably.

## Desktop UI automation

- Required workflows drive the actual Electron window, preload and runtime with generated data and service fixtures.
- Recorded and verified tool versions ([TESTING.md](../TESTING.md#ui-automation)) include Playwright, Electron, host Node and Electron's embedded Node, with the application configuration.
- Keep sandbox, context isolation, web security and the Content Security Policy intact under automation, and inspect packaging, debugger and fuse requirements explicitly.
- A suite launched with GPU acceleration and software GL turned off has no WebGL; drawing with WebGL needs its own check with GPU acceleration.
- The test host supplies the required native libraries and a working display; Linux CI may use Xvfb.
  Linux workflows run with a session bus of their own (`dbus-run-session`), as on a Linux desktop.
  A local WSL graphical session is not a witness for a hosted VM or Xvfb.

## Viewport and pointer

- Set the renderer viewport through Playwright, whose screenshots keep it on a display scaled above 100%; a viewport set through a separate DevTools session is dropped by its screenshots there.
- On a scaled display the reported device pixel ratio carries a rounding error; compare it to six decimals.
- Chromium sends pointer events at the real cursor's position while it lies inside the window.
  After each start, move the window off the real cursor, send a pointer leave, and wait until the page reports no hovered element.
  A workflow that ends with the real cursor back inside the window fails, unless it tests the window's own size or position and opts out by name.
