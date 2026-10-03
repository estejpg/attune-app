<p align="center">
  <a href="https://github.com/estejpg/attune-app/releases">
    <img src="public/readme-icon.svg" width="144" alt="View Attune releases for this fork">
  </a>
</p>

# Attune App

Attune App is a desktop control panel for the sibling
[`attune`](https://github.com/Panchangam18/attune) runtime. It scans supported Chromium desktop apps,
applies Attune theme adapters, and launches/stops live CSS sessions without
requiring an LLM to run commands by hand.

Official themes and attunements live in the public
[`Panchangam18/attunements`](https://github.com/Panchangam18/attunements)
catalog. Development reads a sibling checkout directly; packaged releases
include a pinned catalog snapshot so the app remains reproducible and works
offline.

## Development

```sh
npm install
npm run dev
```

By default the app expects the runtime at `../attune`. You can override that
with:

```sh
ATTUNE_ROOT=/path/to/attune npm run dev
```

The package catalog defaults to `../attunements` and can be overridden with
`ATTUNE_CATALOG_ROOT=/path/to/attunements`.

At startup, Attune installs versioned catalog packages into
`~/Library/Application Support/Attune/workspaces`. Catalog-managed packages
receive an `.attune-package.json` marker and can be upgraded safely; unmarked
folders are considered user-owned and are preserved. Built-in themes are read
from the bundled catalog, while user themes override matching catalog IDs.

If the runtime is not built yet, either run `npm run build` in `../attune` or
use the app's build button.

### Model refresh in this fork

This fork uses sibling checkouts of
[`estejpg/attune`](https://github.com/estejpg/attune) and
[`estejpg/attunements`](https://github.com/estejpg/attunements). Clone all three
forks into the same parent directory before building or packaging the app.
The app bundles its sibling runtime and catalog; the upstream siblings do not
contain this model refresh.

No installer has been published for this fork yet. The releases page above
will show a DMG after a signed and notarized build is published.

For personal use on your own Mac, build a local DMG without an Apple Developer
Program membership. Install dependencies in the `attune` and `attune-app`
checkouts, then run `npm run package:mac:local` from `attune-app`. The DMG will
be at `release/Attune-0.1.23-mac-universal.dmg`. This path uses an ad hoc
signature and skips notarization; the regular `package:mac` and `release:mac`
commands retain their Developer ID signing settings. The local app has the
same name and bundle ID as upstream Attune, so copying it to Applications
replaces an existing Attune installation there.

The ChatGPT desktop picker retains the models supplied by its installed Codex
runtime and adds Claude Opus 5, Claude Fable 5, and Grok 4.7 through their
authenticated local CLIs. The Claude desktop picker retains its native Claude
models and adds GPT-6 Astra, GPT-6.1 Sol, and GPT-6 Luna through the local
Codex bridge. Available models still depend on the user's provider account
and installed CLI versions.

## Coding agent integrations

Settings can install Attune's skill for ChatGPT, Cursor, and Claude. Each
toggle writes an Attune-managed skill to that agent's global skill directory
and removes only files previously created by Attune. Existing user-managed
skills are preserved and reported as conflicts.

The installed skill uses a launcher in Attune's application-support directory
so packaged agents can invoke the bundled runtime. Start a new agent session
after changing an integration toggle so the agent reloads its skill catalog.
Enabled managed skills are refreshed from the bundled canonical skill whenever
Attune reads its state, so runtime and agent instructions remain in sync after
an app update.

## Pick an element

With Attune running, press `Option+Command+A` while an attached Chromium app,
Attune-opened Chrome window, or Safari tab is in front. Attune highlights the
component under the pointer. Use the up and down arrow keys to move between the
highlighted element and its ancestors, click to copy a chat-ready element
reference, or press Escape to cancel.

Option-click chooses the highlighted component as a component-smuggling
source. The source remains visible and authoritative in its original app.
Bring a different Attune app to the front, press `Option+Command+A` again, and
move across the highlighted destination component to choose placement. Its
top, bottom, left, and right zones reserve an internal lane within the destination's
existing bounds; the center uses the normal inside placement. Option-click the center
to replace the destination component instead; stopping the smuggle restores it.
The transplant stays at its source size, and the destination becomes the scroll
boundary when both full-size panes cannot fit.

The transplanted view defaults to a live ScreenCaptureKit stream of the source
component while the source app remains authoritative. Attune requests 30 frames
per second and discards stale frames whenever encoding or delivery falls behind,
then relays hover, clicks, typing, keyboard shortcuts, and scroll gestures back
to the source. Icons, canvas, video, and other source-rendered surfaces therefore
arrive exactly as the source compositor drew them.

If the native stream cannot start, Attune falls back to its live DOM twin. That
path mirrors source structure, computed styles, form state, focus, selection,
scrolling, menus, and popovers using stable node identities and frame-batched
incremental patches; source-local icon fonts are embedded in the destination.
Canvas, video, and other non-DOM surfaces remain capture-backed visual islands
inside that fallback twin. To resize the complete view, enter selection mode in
the destination app and drag any edge or corner handle; pointer
coordinates remain mapped to the original source component. Its close control
also stays hidden during normal use. In selection mode, the `×` control appears
on the transplant and closes both ends when clicked.

While picking, Attune freezes the visible frame, pauses CSS motion, and blocks
host input. Existing semantic roles are copied directly. Unmapped selections
carry bounded resolver evidence so an agent can add a durable, purpose-named
role before styling.

Detailed selection receipts stay local in Attune's application-support folder
and expire after 24 hours. The clipboard reference remains useful by itself and
does not include a page-wide DOM dump or screenshot.

Chrome uses its Attune DevTools session and can run in an isolated profile next
to a normal Chrome session. Safari works as a component source without an
extension; enable Safari's `Develop > Allow JavaScript from Apple Events` once
so Attune can install the picker and relay compact DOM commands. In both
browsers, the live picture comes from the selected native window through
ScreenCaptureKit rather than repeated browser screenshots. Safari destinations
are not yet supported.

## Scripts

- `npm run dev` starts Vite and Electron.
- `npm run build` type-checks and builds the renderer and Electron main process.
- `npm start` builds and opens the production Electron bundle locally.
