# signalrgb-corsair-bragi-scimitar

A SignalRGB add-on that gives the **Corsair Scimitar Elite Wireless SE** its
twelve side buttons back on Windows, without iCUE.

## The problem

SignalRGB puts the mouse into software mode so it can paint the RGB. In that
mode the Scimitar stops sending the side buttons as ordinary input and reports
them as a vendor bitmask on the Slipstream dongle instead.

SignalRGB's own Corsair plugin decodes that bitmask correctly and turns it into
macro events — but the tab that would bind those events fails to load, so
nothing receives them:

```text
ThirdpartyMacroTab.qml:2:1: "../Macroblocks/": no such directory
```

The device shows a Macros tab, the buttons produce events, and the buttons still
do nothing.

## What this changes

Each side button sends a real Windows virtual key, picked per button in the
device's own settings. Buttons default to `1` through `=`.

Only the input path differs. Lighting, DPI, battery and polling rate are
untouched, so changing an effect or a colour never disturbs the buttons.

It claims the Slipstream dongle (`1b1c:2b00`) alone, so every other Corsair
device stays on the plugin SignalRGB ships.

The buttons work while SignalRGB is running, which is also what holds the mouse
in software mode.

## Provenance

`corsair-bragi-scimitar.js` is **not** original work. It is SignalRGB's own
`Corsair_Bragi_Device.js`, by WhirlwindFX, with a small patch applied to the
side-button path. The patch, and the script that applies it to whichever version
of the plugin is installed, live in
[`headless-rgb`](https://github.com/drungrin/headless-rgb) as
`tools/vendor_bragi.py`. Regenerate rather than editing this file by hand:

```bash
python tools/vendor_bragi.py
python tools/sync_addon.py bragi ../signalrgb-corsair-bragi-scimitar
```

## License

The patch and this repository's own files are MIT; see [LICENSE](LICENSE).

That does not extend to the rest of `corsair-bragi-scimitar.js`. Upstream
publishes the plugin at <https://gitlab.com/signalrgb/signal-plugins> without a
license file, so those parts stay under their author's terms and are
redistributed here only for use with SignalRGB itself.

## Installing

**Not through Settings → Add-ons.** That route cannot work here, and the reason
is worth knowing: SignalRGB keeps several plugin sources, and for a given
`VID:PID` the last one crawled wins. Add-ons added by URL are crawled *before*
the plugins SignalRGB ships, so this plugin would register `1b1c:2b00` and the
stock Corsair plugin would immediately take it back. Its own plugin folder is
crawled after, and does win.

So copy `corsair-bragi-scimitar.js` into:

```text
%LOCALAPPDATA%\VortxEngine\app-<version>\Signal-x64\Plugins\
```

under a name that sorts after `Corsair`, such as
`ZZZ_Corsair_Bragi_Scimitar.js`, and restart SignalRGB. The crawler log then
reads:

```text
HID plugin with id 0x1B1C:0x2B00 already exists. Overwriting with new path: ...ZZZ_Corsair_Bragi_Scimitar.js
```

and the dongle appears as **Corsair Bragi Device (Scimitar side buttons)**. Set
each button under that device's settings.

A SignalRGB update installs a new `app-<version>` folder and leaves the copy
behind, so it has to be put back afterwards.
[`headless-rgb`](https://github.com/drungrin/headless-rgb) does both steps with
`python tools/vendor_bragi.py --install`.
