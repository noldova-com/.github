# UI

**Scope:** Appearance, layout, interaction and accessibility principles for every product interface.
The organization's components library (prefix `nol`) owns the shared tokens, component metrics and keys; a project's UI document owns its theme and adds to these rules.

**Company-wide:** every product uses the Noldova Sans and Noldova Mono fonts and the Material Symbols icons until the company's own icon set replaces them.
A project never swaps them for others.

Automation and screenshots are owned by [TESTING.md](TESTING.md#ui-automation).

Dimensions yield to text fitting, responsive layout and accessibility.
Reference applications do not automatically change a project's rules.

## Units and text

- **Use rem for dimensions.** Interface text, icons, spacing, radii, shadows and geometry scale together with the root size.
- Keep pixels only for thin borders, dividers, outlines and progress bars, for fixed minimums such as pointer targets, and for font sizes the person chooses.
- Use percentages, flex or grid for dimensions tied to the available space.
- Text roles (body, label, title, heading, prose, code) come from shared tokens; components define no text sizes.
- Inline code takes the code font at the size and line height of the text around it.
- Row and tab selection does not change font weight.
- Rows, controls, headings and overlays grow with fonts, zoom, translations and validation text; never clip glyphs or labels vertically.
- Specified horizontal truncation ends with an ellipsis and keeps the full text as the accessible name and in a tooltip while it is cut.
- A mark that must sit centred in a drawn frame, such as a checkbox tick, is an inline vector shape, because a glyph is centred by its font box, not by what it draws.
- Glyph size and control geometry are separate: a small icon still needs a usable pointer target.
- Icons in a control never shrink; its label truncates instead.

## Color and themes

- Every color uses a theme token; provide complete initial tokens before painting themed content.
- A theme is data: colors per mode and the values of geometry tokens.
  It contains no styles, selectors or code.
  Fonts and text sizes stay the person's choice, and layout, interaction and accessibility rules hold in every theme and mode.
- The person chooses the mode as System, Light or Dark, offered in that order, with System the default.
- Use one interaction accent per theme; errors and additions or removals have their own semantic colors.
- A nested preview that paints a theme resolves its derived colors within its own scope.

## Alignment and edges

Every edge lines up with something: text starts, control ends, field edges, borders and insets.

- Content in a panel starts on one shared edge, such as the edge where the first tab's icon starts; headings, descriptions, rows, tables and actions start and end on the content edges.
- Controls at the ends of a bar end on the same edges as the panels below it, so their hover fills line up with the panels' borders.
- A hover or selected fill around a label in a bar or a group of pills leaves the same padding on each side, so fills side by side match.
  An icon-only button is a square with no side padding.
- The rows of a menu, list, tree or dropdown fill their column, with the same inset for every row; a row without an icon keeps the icon's space, so every label lines up.
- Cells of a table row align on their first line's baseline, so text sits level with the labels of the row's buttons.
- A table split into groups keeps one header row above them, and each column starts on the same edge in every group.
- A popup opens aligned with its anchor's start or end edge.
- A scrolling pane's scrollbar sits at the pane's own end edge: the scrolling area fills the pane, and the pane's padding goes inside it.

## Lines and space

- Groups are separated by space, never by lines: no dividers under titles or group headers, under every bar, or between every container.
- The space above a group header is more than twice the space below it, so the label belongs to its own rows.
- Tables keep their row lines; functional borders, code-header dividers and separators inside menus and toolbars stay.
- Spacing comes from a small fixed scale of tokens; the components library's metrics table assigns geometry and radii, and components add no literal sizes of their own.
- A fully round radius is only for circles and avatars.
- Ordinary panels and content cards have no shadow; overlays such as menus, tooltips, dialogs and drag ghosts may.
- Cards may clip content to their corners; overlays, focus indicators and drag guides stay visible, and controls stay reachable.

## Responsive layout

- Overlays fit inside the window with a margin; reduce preferred widths as needed, bound their height and scroll their content, keeping essential actions reachable.
- A popup anchored to a control (menu, popover, select list, tooltip, search) closes when that control scrolls away, is removed or stops rendering.
- Fields and selects shrink to their container; settings rows and dialog actions wrap or stack; errors wrap within their owner.
- A table too narrow for its columns gives up its least important column first, then the next, keeping the rest lined up and each row's name whole.
- Prose wraps; code and genuinely two-dimensional tables and diffs may scroll sideways within their own region.
  A long string never widens its card or the window.
- Nothing else scrolls sideways.
- Preserve the person's saved sizes, so they return when space becomes available.
  When preferred minimums cannot fit, collapse into reachable controls rather than overflow the window or produce negative sizes.
- Navigation, hidden-panel controls and dialog actions stay reachable at the project's smallest supported window and at enlarged zoom.
- Drafts, selections and reading position survive layout changes.
- Native window controls and application controls occupy separate regions; an overlay's controls are never covered by the operating system's.

## Interaction states

- **Hover:** use the row, toolbar or button hover token; a checkbox or field needs no extra hover fill.
  Hover-only actions also appear on keyboard focus and stay usable while the pointer moves to them.
  Hover never erases the selection or the keyboard-focus cue.
- **Pointer:** buttons, links, tabs, selectable rows and menu items use a hand cursor; editable text uses the text cursor, resize handles their resize cursor, disabled controls the default cursor.
- **Selection:** rows, tabs and choice pills use the selected surface; keyboard-active options in a select or dropdown list use the active-list pair, and the quick input's active result uses the selected surface; a checked checkbox shows its mark.
- **Focus:** editable fields use the accent border; an invalid field keeps its error border while focused.
  Other controls have a visible focus outline that survives hover and selection and is not clipped or hidden behind overlays.
  Use a contrast-safe alternative or a two-color treatment where the accent blends into a control's fill.
- **Disabled:** a genuinely unavailable control may dim but never responds to activation; explain why where useful.
  Readable status text and unselected tabs are not disabled controls and keep full contrast.
- **Working:** show local progress, and cancellation where supported, in the component that owns the work.
  Announce significant progress and completion without announcing every tick or streamed token.
  Completion and errors stay understandable without animation.
- **Errors:** show a clear message with an applicable recovery action, below the field it explains and never inside the input.
  A field keeps a rejected entry as typed, also after focus leaves, marked invalid with its error associated, and stores nothing until it is corrected; Escape restores the stored value.
  Never rely on a transient toast to explain a failure.
- **Dragging:** show the source, valid targets and the intended destination.
  Clear previews, insertion lines and highlights on drop, cancellation, blur, loss of the target, or when the pointer leaves the window.
  Every drag uses the shared pointer drag; Escape cancels it.
  The click that ends a drag chooses nothing.
- **Motion:** respect reduced motion; fades and rearrangement animations are skipped when it is preferred.

## Accessibility

Every product meets [WCAG 2.2](https://www.w3.org/TR/WCAG22/) level AA, applied to desktop software as [WCAG2ICT](https://www.w3.org/TR/wcag2ict-22/) describes.
The rules below make it concrete, and go further where a product needs more.

- Operable text, placeholders, tooltips and informative secondary text meet at least 4.5:1 contrast at normal sizes; large text at least 3:1 ([text contrast](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)).
  Measure the composited pair in every relevant state, such as hover, selection and overlays, and correct the mapping where it fails.
  A terminal's stored ANSI colors may keep the earlier terminal's values below this, as long as the terminal sets xterm's `minimumContrastRatio` to 4.5, so every color it draws meets 4.5:1 on screen.
- Essential control boundaries, state indicators and focus cues meet at least 3:1 against adjacent colors ([non-text contrast](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html)).
  Decorative separators do not substitute for an identifiable control.
  Until the first public release has shipped, a selected fill (rows, tabs and choice pills) or a keyboard-active fill is a stated exception: it needs no 3:1 against its surface, since the part's semantics convey the state, and a second, subtle visual cue follows after the release.
- Color alone never communicates success, error, availability or identity; use an icon, sign or text as well.
- Use semantic controls, accessible names and associated labels.
  Icon-only actions expose their purpose.
  Errors and help text are associated with their inputs; a tooltip or placeholder is never the only label.
  Expose selected, expanded, checked, pressed and disabled states through semantics.
- Pointer targets are at least 24 by 24 CSS pixels, or meet a documented spacing or equivalent-control exception ([target size](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)).
  A target sized in rem keeps this minimum when the root is small.
  Hit regions never overlap adjacent controls.
  A narrow resize handle needs adequate hit spacing or an equivalent control, plus keyboard operation.
- Tooltips open on focus as well as hover, can be dismissed without moving focus, stay available while the pointer moves over them, and hold descriptions only; interactive content goes in a popover or dialog ([hover and focus content](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus.html)).
- Modal dialogs take sensible initial focus, contain focus, make the rest of the window inert, and return focus to the opener or a surviving control when closed ([modal dialog](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)).
  Escape dismisses a dismissible overlay; forms with unsaved work have an explicit discard policy.
  Non-modal search and popovers never trap focus.
- Enlarged text, zoom, high-contrast and forced-color modes and reduced motion stay usable.
  Under forced colors every state keeps a cue in system colors, and overlays keep their borders once shadows are gone.
- Provide text alternatives for images, and status announcements that do not repeatedly steal focus.
- A virtualized list marks each rendered row's position and the list's size, marks itself busy while rows in view load, and keeps the focused row rendered and focused while it scrolls and while rows are added above it.

## Keyboard

- Every operation is usable by keyboard, with a predictable focus order.
- Activation and dismissal never depend on middle-click, double-click, drag or a context menu alone; each has a keyboard or menu alternative.
- Tabs, lists, trees, menus and completion use their established keyboard patterns, such as the [toolbar](https://www.w3.org/WAI/ARIA/apg/patterns/toolbar/), [menubar](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/), [listbox](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/) and [feed](https://www.w3.org/WAI/ARIA/apg/patterns/feed/) patterns.
- A composite control (toolbar, strip, list, tree, radio group) is one Tab stop that returns to the item last focused; arrow keys move within it, and disabled items stay focusable and mark themselves disabled.
- Context menus open from a right click, the menu key and Shift+F10.
- A row with several actions gathers them in one menu button at its end, whose accessible name names the row; destructive actions come last, after a separator.
- Default keys follow common desktop conventions for each platform, and menus show them.
- When a control holding the focus goes away, the focus moves to a sensible surviving control.

## Shared components

- Colors, geometry, text roles, spacing and radii come only from the shared tokens and layout and component classes; features add no literal values, never restyle shared components or redefine tokens, and add only named, themed tokens checked in every theme and mode.
  A feature component's own document defines its geometry and behavior.
- Styles are plain CSS: no preprocessor such as Sass and no CSS framework.
  Shared values are the components library's CSS custom properties; a shared rule is written out where it applies, because a custom property can't hold a selector.
- Adding a component to the shared library, or a new shape of one, is a reviewed change to the components library's table, not a private visual exception.
- A long collection uses the shared virtual list; a feature neither renders a whole long collection nor virtualizes one on its own.
- Framework overrides preserve semantics, focus and hit testing as well as appearance.
  Verify the actual cascade and theme scope rather than assuming a stylesheet order or raising specificity.

## Verification

- A catalog page shows each reusable control's meaningful states (default, own states, disabled, focus, error, long text, overlays) in every theme and mode, and a test fails when an exported control is missing from it.
  It does not replace checks in real layouts.
- A fixture theme whose values all differ from the default verifies that every control takes its colors and geometry from the theme.
- Verify default, minimum and maximum font preferences, that geometry scales with the root while borders keep their pixel width, font fallbacks, 100% and 200% zoom, the smallest supported window, reduced motion and forced colors.
- Test popup anchoring during scroll, resize and removal, and check that code and diff scrolling never widens the page.
