# Maintaining the Codex theme

`theme.css` is a measured compatibility layer between bb and the rendered
OpenAI Codex desktop UI—not an interpretation of token names.

## Rules

- **Compare rendered leaves.** Measure the actual text, icon, fill, border, or
  shadow-bearing element in both apps with `getComputedStyle()`. Root tokens,
  wrappers, inherited parents, and names are not proof; use them only to trace
  a measured value to a maintainable source.
- **Check light and dark**, including relevant idle, hover, selected, focus,
  open, checked, and disabled states.
- **Match practical roles.** Map every distinct bb token to the closest Codex
  counterpart in actual use. If bb combines roles Codex separates, flag the
  conflict and use the narrowest stable selector. Do not redesign Codex.
- **Do not change geometry by default.** Unless explicitly authorized, do not
  alter positioning, sizing, spacing, layout, shape, border width, radius,
  typography, or font. Color, opacity, background, border color, shadow, and
  blur are in scope.
- Ship one self-contained `theme.css` with `:root, .light` and `.dark`. Avoid
  network-loaded assets and fonts.

## Workflow

1. Run `git status -sb`. Read `bb guide customization` and the installed
   bb-cli skill's `references/theming.md` before changing tokens.
2. Use safe inspection targets:
   - Never drive the live bb desktop app. Use a separate web client, normally
     `http://127.0.0.1:38886`; mutate only the auditing agent's own subtree.
   - Launch Codex separately with a known remote-debugging port and preferably
     a disposable `--user-data-dir`. Record its PID and treat existing data as
     read-only: navigate and inspect, but do not send, edit, pin, archive,
     approve, or commit.
   - Record the initial appearance, obtain approval if changing a potentially
     shared preference, and restore it afterward (normally **System**). Stop
     only the audit PID and remove disposable state.
   - Media emulation may not switch bb. Verify its actual `.light`/`.dark`
     class and `color-scheme`; changing the class in an isolated page is valid
     for render-only testing.
3. For each corresponding Codex and bb leaf, capture its visible label,
   role/data attributes, useful classes, foreground, background, border,
   opacity, shadow, background image, and contributing ancestors. Inspect the
   smallest painted descendant—not merely its anchor or row.
4. In both modes, spot-check:
   - sidebar thread titles/fills in every state, project/group titles, and
     section headings;
   - transcript prose, user bubbles, links, inline code, metadata, activity,
     file paths, and unread markers;
   - main/composer/resource surfaces, right-panel tabs, dividers, focus rings,
     menus, popovers, tooltips, and all shadows;
   - Settings navigation, cards, text tiers, selects, buttons, switches, and
     disabled states;
   - semantic states, diffs, merged PRs, and terminal ANSI colors.
   Inspect at least five relevant components not named in the task.
5. Implement the smallest evidence-backed mapping, preferring: dedicated bb
   token; component custom property; stable role/data/class selector; then a
   structural selector. Preserve explanatory comments. Do not restate bb's
   default font.
6. Run `git diff --check`, then `bb theme set Codex`—do not assume a file
   watcher. Re-read the same leaves in isolated light and dark clients, confirm
   `bb theme show --css` and `bb theme list`, check adjacent consumers, and
   reject unauthorized geometry or typography. Keep audit tables limited to
   remaining mismatches. Restore and clean up both audit apps before committing.

Hackiness: **Low** = token/stable role; **Low–Medium** = narrow direct selector;
**Medium** = scoped descendant/compound state; **High** = shared role,
pseudo-element, or shell coupling; **Very high** = structural/data
reconstruction. Explain high or very-high tradeoffs before implementation.

Known exception: bb shares a utility between project/group and section labels,
while Codex renders them at 85% and 50% foreground. Preserve the
`[data-sidebar-sticky-project-item]` override.

Keep fidelity changes and publication cleanup in separate commits when asked.
Retain `theme.css`, `README.md`, `LICENSE`, `AGENTS.md`, and the
`CLAUDE.md -> AGENTS.md` symlink at the repository root. The install directory
must remain `$(bb theme dir)/Codex` so the theme id is **Codex**.
