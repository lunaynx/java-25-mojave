# OpenJDK 25 for macOS Mojave

GitHub Actions build of OpenJDK 25 for macOS x64 with a minimum supported OS version of
macOS 10.14 Mojave, including AWT and all other native libraries.

It's built from the latest `release/jdk-<version>_<build>` branch of
[microsoft/openjdk-jdk25u](https://github.com/microsoft/openjdk-jdk25u), the source of the
Microsoft Build of OpenJDK. This is not an official Microsoft build.

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

Runs on every push, or manually via *Actions → Build → Run workflow*. Each run picks the
newest release branch at that point.

The workflow builds on `macos-15-intel` with Xcode 16.4, fails if any Mach-O file in the image has
a minimum OS version other than 10.14, builds an installer, installs it on the runner,
smoke-tests AWT from the installed JDK, and uploads the JDK and debug symbol bundles and the
installer as artifacts.

## Installing

Open the `.pkg` from the `-installer` artifact. It installs to
`/Library/Java/JavaVirtualMachines/openjdk-mojave-25.jdk`, replacing an earlier install there.
The package isn't signed, so Gatekeeper blocks a double-click: right-click it and choose *Open*.

To uninstall:

```sh
sudo rm -rf /Library/Java/JavaVirtualMachines/openjdk-mojave-25.jdk
sudo pkgutil --forget io.github.lunaynx.openjdk-mojave-25
```

If a patch stops applying to a new release branch, the *Apply patches* step fails; rebase the
patch on that branch.
