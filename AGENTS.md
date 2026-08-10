# Maintaining the Codex theme

This repository contains a bb color theme that reproduces the OpenAI Codex
desktop app as closely as bb's theme and component surfaces allow. Treat
`theme.css` as a measured compatibility layer between two rendered apps, not
as an interpretation of token names.

## Non-negotiable principles

1. **Compare rendered leaves.** Establish a match from the actual text, icon,
   fill, border, or shadow-bearing leaf in each app and any ancestor that
   materially contributes to that leaf. Do not infer a match from token names,
   `:root` values, inherited parent colors, or stylesheet declarations alone.
   Tokens are useful only after a rendered value has been measured, to trace
   that value back to a maintainable source.
2. **Verify both light and dark.** bb's palette is global, but appearance mode
   is per client. A change is incomplete until corresponding leaves and states
   have been checked in both modes.
3. **Preserve practical role fidelity.** When bb has a distinct theme token,
   map it to the closest corresponding Codex role in actual use. If bb combines
   roles that Codex separates, prefer a stable, scoped component selector over
   compromising unrelated leaves.
4. **Flag role conflicts.** If one bb token drives components with different
   contrast or semantic needs, document the conflict and its practical effect.
   Do not silently redesign Codex. For example, a color shared by Codex links
   remains the fidelity target even if bb also uses it for a structurally
   different marker.
5. **Do not change geometry by default.** Unless the task explicitly authorizes
   it, do not alter geometry, positioning, sizing, spacing, layout, shape,
   border width, radius, or typography. Color, opacity, background, border
   color, shadow, and blur are in scope. The native bb layout is intentional.
6. **Keep the stylesheet self-contained.** Ship one `theme.css` with
   `:root, .light` and `.dark` variants. Avoid network-loaded assets and fonts.

## Safety and app access

- Never drive the user's live bb desktop app. Inspect bb through a separate web
  client, normally `http://127.0.0.1:38886`, or another separately launched
  instance. If realistic bb state is needed, create or manipulate only threads
  in the auditing agent's own subtree.
- Launch a separate Codex instance with a known remote-debugging port when DOM
  inspection is needed. Prefer a disposable `--user-data-dir`, record the exact
  PID, and stop only that PID afterward. Realistic authenticated state is useful
  evidence, but treat pre-existing threads and data as read-only.
- Appearance radios may persist beyond a window. Record the initial Codex
  appearance, obtain approval before changing a potentially shared preference,
  and restore it before cleanup. The normal expected restoration is **System**.
- Do not archive, pin, edit, send, approve, commit, or otherwise mutate real
  Codex threads while auditing. Navigation, opening menus, hover, focus, and
  reading computed styles are acceptable in the launched audit instance.
- Close browser-automation sessions and remove disposable profiles when done.
  Before terminating an app, resolve and verify its exact PID so the ordinary
  live process remains untouched.

## Standard workflow

### 1. Orient and establish scope

1. Run `git status -sb` and preserve unrelated work.
2. Read `bb guide customization` and the installed bb-cli skill's
   `references/theming.md` before changing theme tokens.
3. Inspect `theme.css` and the recent history to understand deliberate
   component selectors and prior exclusions.
4. Write down the corresponding Codex and bb component roles to inspect. Do not
   start with similarly named variables.

### 2. Launch isolated inspection targets

For Codex on macOS, use a temporary profile and a free debugging port. A typical
launch is conceptually:

```sh
profile_dir="$(mktemp -d /tmp/codex-theme-audit.XXXXXX)"
/Applications/ChatGPT.app/Contents/MacOS/ChatGPT \
  --user-data-dir="$profile_dir" \
  --remote-debugging-port=9333
```

Launch this as a separate process rather than relaunching or controlling the
normal app. Connect browser automation to the debugging port.

For bb, open the local web app in a separate automation session. Do not use the
desktop window. Media emulation alone may not change bb's resolved appearance;
verify the actual `.light` or `.dark` class and computed `color-scheme`. In an
isolated page, changing only the document appearance class is an acceptable
render-only test. Do not change the live desktop client's mode.

### 3. Measure corresponding rendered leaves

Use `getComputedStyle()` on the smallest element that actually paints the role.
For each value, capture enough provenance to relocate it later:

- visible text or accessible name;
- tag, role, stable data attributes, and useful classes;
- foreground and background color, including alpha and color space;
- border color and width;
- `box-shadow`, `background-image`, opacity, and relevant ancestor compositing;
- state: idle, hovered, selected, focused, open, checked, unchecked, or disabled.

Do not stop at an anchor or row wrapper if a descendant span paints a different
color. Do not treat an inherited foreground as evidence until the visible leaf
has the same computed value. Inspect pseudo-elements and gradient-bearing child
elements when they are the visible divider or marker.

At minimum, spot-check all of these in light and dark:

- sidebar thread titles: idle, hover, and selected;
- project/group titles separately from section headings;
- selected and hovered sidebar fills;
- transcript assistant prose, user prose, bubbles, links, inline code,
  metadata, activity headers, file paths, and unread markers;
- main surface, composer, resource/change cards, and all elevation shadows;
- right utility panel surface and selected, idle, and hovered tabs;
- pane divider idle, hover, focus, and active treatments;
- menus, menu items, popovers, and tooltips;
- Settings navigation, cards, separators, headings, descriptions, select-style
  controls, visible buttons, hover-revealed buttons, switches, and shadows;
- semantic states: destructive, warning, attention, success, diffs, merged PRs,
  focus, selection, and terminal ANSI colors.

Also inspect at least five components not named in the task. Themes often miss
resource cards, tooltips, composer controls, file-path tiers, focus rings, and
disabled states.

### 4. Map the evidence into bb

Apply this order of preference:

1. A dedicated bb token whose actual consumers correspond to the Codex role.
2. A component-only custom property used by a narrowly scoped rule.
3. A stable role, data-attribute, or component-class selector.
4. A structural selector only when bb collapses visually distinct roles.

Avoid broad selectors based only on generic utilities. One known example is
bb's shared low-emphasis utility for both project/group labels and section
labels: Codex renders project/group titles at 85% foreground and section titles
at 50%, so the theme distinguishes project labels through
`[data-sidebar-sticky-project-item]` rather than trusting the shared utility.

When evaluating a potential override, record its hackiness:

- **Low:** dedicated token or stable role selector.
- **Low–Medium:** narrow component selector with a direct counterpart.
- **Medium:** scoped descendant or compound state selector.
- **High:** shared-role conflict, pseudo-element, or layout-shell coupling.
- **Very high:** new structure or data reconstruction.

Call out the tradeoff before implementing high or very-high changes.

### 5. Edit and reapply

1. Make the smallest evidence-backed change with `apply_patch`.
2. Preserve existing comments that explain non-obvious role mappings.
3. Do not add font overrides merely to restate bb's built-in default.
4. Run `git diff --check`.
5. Reapply with `bb theme set Codex`. Editing the file is not sufficient for a
   reliable verification pass; do not assume a file watcher.

### 6. Verify the result

1. Re-read the same bb leaves in the isolated web client in both modes.
2. Confirm the active stylesheet with `bb theme show --css` and the active
   palette with `bb theme list`.
3. Verify that targeted fixes did not change adjacent consumers of shared
   tokens or selectors.
4. Confirm the diff contains no unauthorized geometry or typography changes.
5. If an audit table exists, keep it as a table of **remaining** mismatches.
   Remove fixed rows and include hackiness for each remaining fix.
6. Restore Codex appearance, close automation sessions, stop the exact audit
   process, and delete its disposable profile.

### 7. Commit discipline

- Keep baseline captures, fidelity changes, geometry removal, and publication
  cleanup in separate commits when the task calls for that history.
- Commit only after the theme has been reapplied and both modes verified.
- Leave the worktree clean and report the commit hash, active theme, modes
  checked, and any intentionally excluded mismatches.

## Publishing checklist

- `theme.css` remains at the repository root.
- `README.md`, `LICENSE`, `AGENTS.md`, and the `CLAUDE.md -> AGENTS.md` symlink
  are present.
- The README installation target is `$(bb theme dir)/Codex` so the visible
  theme id remains **Codex**.
- `git diff --check` passes and the repository is clean.
- The GitHub repository is public and MIT-licensed.
- Re-run `bb theme set Codex` after the final checkout changes and verify the
  active palette before publishing.
