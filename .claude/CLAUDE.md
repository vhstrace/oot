# Cloud sessions (Claude Code on the web and mobile)

This part applies only when `baseroms/ntsc-1.0/baserom.z64` is missing,
which is the case in a cloud session. On the Mac, ignore it and follow
`AGENTS.md`.

The target is the N64 US NTSC 1.0 ROM (`VERSION=ntsc-1.0`, US region).
The source covers many versions with `#if` checks on `OOT_VERSION`,
`OOT_NTSC`, `PLATFORM_N64`, `DEBUG_FEATURES` and so on. Read the code
that ntsc-1.0 builds (N64, no debug features), not the debug, GameCube
or iQue parts.

Work on the `cheat-hunt` branch. Everything you make goes in `cheats/`:

- `cheats/oot_patches.md`: one entry per patch or GameShark code.
- `cheats/patches/<slug>.diff`: the C source change for a patch.

The Mac copies these back later. It builds the ROM, makes the BPS
patches and syncs Batocera and MinUI. None of that is possible here, so
don't try it, and don't write to `~/Downloads`.

## What you can't do here

- There is no ROM. Base ROMs are not in git and must never be. Never
  commit a ROM, any file made from one (`extracted/`, `build/`, assets),
  or bytes copied out of one.
- Don't build or run `make setup`. Both need the ROM.
- Until `cheats/oot-ntsc-1.0.map` exists, you have no RAM or ROM
  addresses. Don't make up addresses or copy them from the web.

## Making a patch

A patch here is a change to the C source, saved as a diff. The source
files themselves are never committed.

1. Find the logic in the C source (`src/code/`, `src/overlays/actors/`,
   `src/overlays/gamestates/`, `include/` and so on). Make the smallest C
   change that does the job.
2. Save it, then undo it:

       git diff -- src include > cheats/patches/<slug>.diff
       git checkout -- src include

   `<slug>` is a short kebab-case form of the title. Check with
   `git status` that no file outside `cheats/` is changed.
3. Check that it still applies: `git apply --check cheats/patches/<slug>.diff`.
4. Write the entry at the bottom of `cheats/oot_patches.md`.

Each diff must stand alone: made against `main`, with no other patch
applied. Don't change the Makefile, the `spec` file, linker scripts or
`tools/`.

Keep changes small. Prefer changing a constant, a condition or a single
call. A change that adds code moves every function after it, which is
fine for a BPS but makes the patch hard to combine with others. Say in
the entry whether the change is in `code` (always loaded) or in an
overlay (an actor or game state, loaded only when it is used).

## GameShark codes

Only once `cheats/oot-ntsc-1.0.map` exists. Get variable addresses from
that map and struct offsets from the C headers (for example,
`SaveContext` in `include/save.h`). Actors and other
overlays are loaded at addresses that change, so a code can't point into
them. Format: `8XXXXXXX 00YY` writes one byte, `81XXXXXX YYYY` writes two
bytes (even address). `D0XXXXXX 00YY` / `D1XXXXXX YYYY` run the next line
only if the value there matches.

## Entry format

```
## Short title that says what the player sees

**Source diff:** `cheats/patches/<slug>.diff`
**Changes:** not built yet
**Recorded:** YYYY-MM-DD HH:MM:SS

**Mechanism:** what the original code does (quote the C, with the
function name and source file), and what the patched code does.

**Evidence:** functions, variables and headers checked, the version
`#if` checks that apply to ntsc-1.0, and the `git apply --check` result.

**Revisions:** US NTSC 1.0 only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- Side effects and limits.
```

For a GameShark code, put `**Codes:** \`8011A5D0 0010\` + ...` in place of
**Source diff:** and **Changes:**, and the map symbol, struct field and
source file in **Evidence:**.

- Newest entry last, in **Recorded:** order. Use simple technical
  English.
- No `Tags:` line unless the user asks for tags. Add or remove tags only
  when asked.
- When porting a cheat from another game, say which one in the Mechanism.
  Other games' cheat files are not in this repo; the user can paste one.

## Commits

Commit each change on `cheat-hunt` and push it. Commit only files in
`cheats/`. A git pre-commit hook (`cheats/githooks/pre-commit`, turned on
by the SessionStart hook) refuses any other file. Messages: `Add: <title>`
for a new entry, `Remove: <title>` for a deleted one, otherwise a short
plain sentence. No attribution lines.
