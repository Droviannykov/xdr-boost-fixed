# xdr-boost-fixed

A fork of [levelsio/xdr-boost](https://github.com/levelsio/xdr-boost) with fixes for screen glitches on unlock and flickering when switching windows.

Free and open-source XDR brightness booster for MacBook Pro. Like [Vivid](https://www.getvivid.app/), but free.

Unlocks the full brightness of your Liquid Retina XDR display beyond the standard SDR limit. Your MacBook Pro can go up to 1600 nits — this tool lets you use it.

## What's fixed

The original xdr-boost has two issues:

1. **Screen glitch on unlock** — the watchdog timer aggressively restores the overlay during the macOS unlock animation, causing a visible flash/glitch. Fixed by tracking lock state via `com.apple.screenIsLocked` / `com.apple.screenIsUnlocked` notifications and waiting 1.5s after unlock for the animation to complete before restoring the overlay.

2. **Flicker when switching windows** — display parameter change events trigger a full destroy-and-recreate cycle of the overlay window, causing a visible off/on flash. Fixed by resizing the overlay in-place instead of tearing it down. Display change events during lock are also ignored.

## Features

- Boosts screen brightness beyond the standard 500 nit SDR limit using XDR hardware
- No white tint or washed-out colors — uses multiply compositing to preserve colors perfectly
- Menu bar icon with brightness presets (1.5x, 2.0x, 3.0x, 4.0x)
- Global keyboard shortcut (**Ctrl+Option+Cmd+V**) to toggle from anywhere
- Survives sleep/wake, lid close/open, and lock/unlock — brightness auto-restores
- Starts with XDR off — rebooting always gives you a normal screen
- Emergency kill switch (`xdr-boost --kill`) if anything goes wrong
- Single binary, no dependencies, ~300 lines of Swift
- Launch agent for auto-start on login

## How it works

MacBook Pro displays can output up to 1600 nits, but macOS caps regular desktop content at ~500 nits. The extra brightness is reserved for HDR content.

xdr-boost creates an invisible Metal overlay using `multiply` compositing with EDR (Extended Dynamic Range) values above 1.0. This triggers the display hardware to boost its backlight, making everything brighter while preserving colors perfectly — no white tint, no washed-out look.

## Requirements

- MacBook Pro with Liquid Retina XDR display (M1 Pro/Max or later)
- macOS 12.0+

## Build

### Prerequisites

You need Swift compiler tools. Either **Xcode** (from the App Store) or **Command Line Tools** will work:

```bash
xcode-select --install
```

> **Note:** On newer macOS versions with Command Line Tools (no full Xcode), you may hit a `SwiftBridging` module redefinition error. This is a known Apple bug caused by a duplicate modulemap file. Fix it with:
>
> ```bash
> sudo mv /Library/Developer/CommandLineTools/usr/include/swift/module.modulemap \
>         /Library/Developer/CommandLineTools/usr/include/swift/module.modulemap.bak
> ```
>
> Alternatively, install Xcode.app and switch to its toolchain:
> ```bash
> sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
> ```

### Build

```bash
git clone https://github.com/Droviannykov/xdr-boost-fixed.git
cd xdr-boost-fixed
make build
```

The binary will be at `.build/xdr-boost`.

### Install to PATH

```bash
sudo make install
```

### Start on login

```bash
sudo make launch-agent
```

### Uninstall

```bash
make remove-agent
sudo make uninstall
```

## Usage

```bash
# Run with menu bar icon (default 2x boost)
xdr-boost

# Run with custom boost level
xdr-boost 3.0
```

Click the **☀** icon in your menu bar to:
- Toggle XDR brightness on/off
- Choose brightness level (1.5x, 2.0x, 3.0x, 4.0x)
- Quit

### Keyboard shortcut

**Ctrl+Option+Cmd+V** — toggle XDR brightness on/off from anywhere, no need to find the menu bar icon.

### Emergency kill

If something goes wrong and you can't see your screen:

```bash
# From terminal (even blind-type it)
xdr-boost --kill

# Or just
pkill xdr-boost
```

The app always starts with XDR **off** — you have to manually turn it on. So rebooting will always give you a normal screen.

### Sleep, lid close, and lock screen

A common problem with XDR brightness apps is that closing your laptop or locking the screen kills the brightness boost, and it doesn't come back when you return. xdr-boost fixes this with a watchdog that automatically restores your brightness after:

- Closing and reopening the laptop lid
- Locking and unlocking the screen (with a smooth delay to avoid glitches)
- Sleep and wake
- Plugging/unplugging external displays

If you turned XDR on, it stays on — no matter what.

## Known limitations

- **Brief flicker when switching windows** — the screen may briefly flash when you switch between apps. This is a macOS compositor limitation: the window server momentarily disrupts the `multiply` compositing filter on the Metal overlay during window transitions. This affects the original xdr-boost as well and cannot be fixed without a fundamentally different approach (e.g. a custom display color profile instead of a Metal overlay).

## License

MIT
