# OpenJDK 25 for macOS Mojave

GitHub Actions build of OpenJDK 25 (from [openjdk/jdk25u](https://github.com/openjdk/jdk25u))
for macOS x64 with a minimum supported OS version of macOS 10.14 Mojave, including AWT and
all other native libraries.

Upstream has hard-coded the minimum to macOS 11 since JDK 22 (JDK-8317970), which is why every
distribution ships binaries with `minos 11.0`.

## Patches

- `0001`: lowers `MACOSX_VERSION_MIN` in `make/autoconf/flags.m4` to `10.14.00`.
- `0002`: reverts the 25u backport of JDK-8373984, which removed an `@available(macOS 11, *)`
  guard around an 11.0-only constant in `CGraphicsDevice.m`.
- `0003`: `CDesktopPeer.m` (`java.awt.Desktop` browse/mail/open/edit/print) uses
  `NSWorkspaceOpenConfiguration` (10.15+) since the JDK-8359501 security fix. Before 10.15 it now
  falls back to `LSOpenFromURLSpec`, still with the explicitly resolved app so the fix's
  checks keep applying.

The build keeps `-Werror`, so any other use of an API newer than 10.14 without an
`@available` check fails the build (`-Wunguarded-availability-new`).

## Building

Runs on every push, or manually via *Actions → Build → Run workflow* with a different
`jdk-25.0.X+N` tag (use the `+N` build tag that the `-ga` tag points at, not `-ga` itself).

The workflow builds on `macos-15-intel` with Xcode 16.4, fails if any Mach-O file in the image has
a minimum OS version other than 10.14, smoke-tests AWT, and uploads the JDK and debug symbol
bundles as an artifact.

To update to a new 25u release, change the default tag in `.github/workflows/build.yml`; if a
patch no longer applies, rebase it on the new tag.
