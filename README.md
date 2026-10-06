# clauth (Josh's fork)

This is Josh's fork of [clauth](https://github.com/uwuclxdy/clauth), the Rust CLI and daemon that
rotates the macOS Keychain login Claude Code uses between his Claude accounts. The fork owns the
account rollover from here on. It does not track upstream, keeps no mirror branch and has no
`upstream` remote. For the full feature documentation, see the
[upstream README](https://github.com/uwuclxdy/clauth/blob/v0.17.0/README.md).

## Base

The fork starts from upstream tag `v0.17.0` (released 2026-10-02, commit `8002027`). Our tag
`fork-base` marks the same commit. Upstream's earlier tags are kept.

## Branches

`main` is the only long-lived branch. Do each change on a short task branch in a worktree at
`~/code/.worktrees/clauth/<task>`, and merge it into `main` only after the build and the test
suite pass.

## Build, install and restart

The main checkout is `~/code/jitwaru2/clauth`.

1. Build and install. rustup's `stable` on this Mac (1.94.1) cannot compile v0.17.0: it fails
   with E0658 because `atomic_try_update` is unstable there. Use the installed 1.98.1 toolchain:

   ```sh
   cargo +1.98.1 install --path ~/code/jitwaru2/clauth --locked --force
   ```

2. Check that the daemon is not mid-Keychain-write. The last line in `~/.clauth/daemon.log`
   matching "Keychain item" must be at least several minutes old. These writes come roughly
   every six hours.

   ```sh
   date -u; grep 'Keychain item' ~/.clauth/daemon.log | tail -1 | cut -c1-80
   ```

3. Confirm `~/Library/LaunchAgents/local.clauth.daemon.plist` still sets `NO_PROXY` in its
   environment. The install does not touch the plist. Do not change it.

4. Restart the daemon:

   ```sh
   launchctl kickstart -k gui/$(id -u)/local.clauth.daemon
   ```

5. Verify. `launchctl print gui/$(id -u)/local.clauth.daemon` shows `state = running` with a
   new pid, `~/.clauth/daemon.log` shows "clauth daemon: running", and the mtime of
   `~/.clauth/status.json` advances within the refresh interval (90 s). Then run
   `clauth --version`, `clauth which` and `clauth list`.

## NO_PROXY

clauth trusts only its bundled CAs, so it must bypass the local proxy (Proxyman) for Anthropic
hosts. The launchd plist sets `NO_PROXY` for the daemon. The `clauth` alias in `~/.zshrc` adds
`api.anthropic.com,.anthropic.com` for interactive use. When running clauth outside that shell,
prefix commands with `NO_PROXY=api.anthropic.com,.anthropic.com`.

## Tests

```sh
cargo +1.98.1 test --locked
```
