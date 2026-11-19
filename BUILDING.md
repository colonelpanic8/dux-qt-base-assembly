# Building and releasing the dux-qt-base assembly

dux-qt-base is Dux Shell's embedded Quickshell fork, built as the shared
library `libdux-qt-base.so` that the `dms` binary links. The installed library
is upstream `duxlabs-dev/dux-qt-base` `master` plus the system tray topics in
`manifest.toml`, published to `colonelpanic8/dux-qt-base` branch `assembled`.
Read `AGENTS.md` for the model and invariants.

The topics carry the tray API that the Dux Shell assembly's QML relies on:
`watcherAddress` on tray items, `contextMenu()`, icon theme path handling,
and the pixmap/overlay fixes. They were ported from the retired standalone
Quickshell stack (`colonelpanic8/quickshell` branch `assembled`, head
`23b10e0`), whose topic branches of the same names remain on that fork.

## Locations

| Item | Location |
| --- | --- |
| Topic development checkout | `~/projects/dux-qt-base` (`origin` = upstream, `fork` = colonelpanic8/dux-qt-base) |
| Published artifact | `colonelpanic8/dux-qt-base` branch `assembled`, tags `dux-qt-base-assembled-*` |
| Flake output | `github:colonelpanic8/dux-qt-base/<assembled commit>#dux-qt-base` (also `#default`) |
| Consumer | `/srv/dotfiles/nixos/dux/shell.nix`: `programs.dank-material-shell.duxQtBase.package`, which the dux-shell module passes to `dms-shell.override { duxQtBase = ...; }` |
| Shell QML that needs these topics | the Dux Shell assembly, `~/projects/dux-shell-assembly` |

## Changing a topic

Work on the topic branch in a worktree of the development checkout
(`~/projects/dux-qt-base/.worktrees/<topic>`), cut from upstream `master`, one
concern per branch. Push it to `fork`, then `fork-assembler update
mine:<topic>`. New topics go in with `fork-assembler add mine:<topic>`.

Two coherence fixups in `patches/` repair edits outside the conflict hunks
that rerere cannot hold: `sni/icon-theme-path-updates` re-applies
`sni/quiet-missing-optional-props`' error handling inside its rewritten
`requestPropertyUpdates`, and `sni/777-pixmap-rendering` applies its
aspect-ratio change to the pixmap helper `sni/777-icon-fallbacks` introduced.

## Release

Release eagerly; validate afterwards and fix forward.

1. `fork-assembler update` (all pins) or `fork-assembler update ENTRY...`.
2. `fork-assembler build`, resolving any stopped conflict and running
   `fork-assembler continue`.
3. `fork-assembler build --locked` must report the lock's tree reproduced.
4. `fork-assembler gc`, then commit `manifest.toml`, `manifest.lock.json`,
   `resolutions/`, and `patches/` with explicit paths, and push.
5. `just publish` pushes the build worktree to `assembled` and tags it.
6. In `/srv/dotfiles/nixos`, repin the dux-qt-base input to the published
   commit (`nix flake update <input>`), then `just switch`. The dms binary
   embeds the library, so the shell package rebuilds too.
7. Validate: `nix build github:colonelpanic8/dux-qt-base/<commit>#dux-qt-base`
   builds the library on its own; after switching, restart the shell
   (`systemctl --user restart dms.service`) and check the tray.
8. Commit `nixos/flake.lock` in the dotfiles repository with an explicit path.
