# Corne (wireless) — ZMK config

A 42-key wireless Corne (nice!nano v2 + nice!view) running ZMK. This config uses
**three OS-aware base layers**, **homerow mods**, and **Bluetooth-profile macros**
that switch the active layer to match whichever device you're connected to.

Pair with a visualizer (keymap-editor or the ZMK keymap viewer) for exact key art —
this README documents *behavior and usage*, not the grid.

---

## Layers

| # | Name (display) | Purpose |
|---|----------------|---------|
| 0 | Colemak (DEF)  | True Colemak-DH output. Default / fallback. Use on Mac, phones, any device with no OS remap. |
| 1 | Windows (WIN)  | Raw QWERTY output, pairs with an OS-level Colemak-DH layout on Windows. Gaming-friendly. |
| 2 | Mac (MAC)      | Colemak-DH output with Ctrl/GUI swapped on the homerow. |
| 3 | Num (NUM)      | Left-hand numpad with F1–F12 hold-taps; right-hand arrows/brackets/nav. |
| 4 | Config (FUN)   | Media, brightness/volume, caps-word, BT management + profile macros. |

---

## The core idea: OS-aware base layers

One physical board, three typing layers, each tuned to the device it's paired with.

- **Colemak (DEF)** sends real Colemak-DH key codes. The device needs **no** keyboard
  layout installed — what you type is what you get. This is the right layer for macOS,
  iOS/Android, or any fresh machine.
- **Windows (WIN)** sends **raw QWERTY** scancodes and expects an **ANSI US Colemak-DH**
  layout installed in Windows. Windows performs the Colemak conversion; games that read
  raw scancodes still see physical QWERTY, so WASD stays correct.
- **Mac (MAC)** is Colemak-DH like DEF, but **Ctrl and GUI are swapped** on the homerow
  so Windows-Ctrl muscle memory becomes Mac-Cmd.

You don't switch these manually key-by-key — the BT macros do it (below).

---

## Bluetooth profile macros (Config layer)

Each macro selects a Bluetooth profile **and** jumps to the matching base layer in one
press. ZMK remembers the active layer across reboots, so you only press these when you
first set up or switch **between** devices — after that the right layer comes back on its
own at power-on.

| Macro | BT profile (0-indexed) | Switches to |
|-------|------------------------|-------------|
| `bt_win` | 0 | Windows |
| `bt_mac` | 1 | Mac |
| `bt_3`   | 2 | Colemak |
| `bt_4`   | 3 | Colemak |
| `bt_5`   | 4 | Colemak |

`bt_win` / `bt_mac` are on the right side of **Config row 2**; `bt_3/4/5` on **Config row 3**.
`BT_CLR` (clear current bond) is the **top-right corner** of the Config layer.
A `to DEF` escape key (jump back to Colemak) sits on the Config layer right thumb.

> Gotcha: these are **press-once-then-persistent**. Don't keep re-pressing them — that
> just re-selects the same profile.

---

## Homerow mods

Every base layer puts modifiers on the homerow via hold-tap. **Tap = letter, hold = mod.**

- Colemak / Windows: `GUI ALT SHFT CTRL` (left) · `CTRL SHFT ALT GUI` (right)
- Mac: same, but Ctrl/GUI swapped.

Behaviors: `hml` (left) / `hmr` (right) — balanced flavor, 280 ms tapping term,
175 ms quick-tap, 150 ms require-prior-idle, with **positional hold triggers** so a mod
only fires when you also press a key on the **opposite** hand.

> Usage: lead slightly with the held key and use opposite hands. Same-hand rolls stay as
> letters by design — that's the misfire protection, not lag.

---

## Num layer

- **Left hand:** 4×3 numpad. Digits double as **F-keys on hold** — `1/F1 2/F2 3/F3`,
  `4/F5 5/F6 6/F7`, `7/F9 8/F10 9/F11`; the operator column (`= + -`) holds for F4/F8/F12;
  outer column has `/ * 0`.
- **Right hand:** arrows, brackets, parens, PgUp/PgDn, punctuation.

Enter/leave with `tog NUM` (toggle) on the **middle right-thumb** key.

---

## Config layer

Left hand: F1–F10, caps-word, INS, PrtSc. Right hand: navigation + the BT macros above.
`caps_word` is smart caps-lock that turns itself off at the end of a word.

Enter by **holding** the rightmost right-thumb key (`lt FUN RET`).

---

## Thumb keys

| Key | Tap | Hold |
|-----|-----|------|
| Right-thumb outer | Enter | Config layer (`lt FUN RET`) |
| Right-thumb middle | toggle Num (`tog NUM`) | — |
| Right-thumb inner | Backspace | **Shift = Delete** (`bspc_del` mod-morph) |
| Left-thumb outer | LGUI on Windows/Mac, LCTRL on Colemak | — |

---

## Windows Z-fix (Windows layer only)

ANSI Colemak-DH relocates **Z** to the end of the bottom-left row. To put it back in the
corner on a columnar board, the Windows layer sends raw scancodes **`B Z X C V`** instead
of `Z X C V B`. After the ANSI Colemak-DH transform this outputs **`z x c d v`**.

> This is tied to **ANSI US Colemak-DH** on Windows. If you change that Windows layout,
> the Windows-layer bottom row must be re-tuned. Colemak and Mac layers send true
> Colemak-DH directly and are unaffected.

---

## Windows setup (for the Windows layer)

1. Install **ANSI US Colemak-DH** (the `cdh_us_*.msi` / `setup.exe` build), reboot.
2. Settings → Time & language → Language & region → English (US) → ⋯ → Language options →
   Add a keyboard → Colemak DH.
3. Set it as default: Typing → Advanced keyboard settings → **Override for default input method**.
4. On the board, press `bt_win` once (Config layer) to select profile 0 + Windows layer.
5. Bottom-left row should read `z x c d v`.

Remove any **UK / ISO** Colemak-DH layout you installed — the UK variant maps `\`→`#` and
swaps quotes, and pulls in an extra display language. ANSI US avoids all of that.

---

## conf notes

`config/corne.conf` keeps the tuned link/scan settings:
deep sleep (1 h), TX power +8, BLE ACL buffers 6/6, 1M PHY, connection-interval
preference (`PREF_MAX_INT=6`), and eager debounce (press 1 ms / release 7 ms).

It also sets `CONFIG_LV_Z_MEM_POOL_SIZE=5120` to fix nice!view widgets not fully
refreshing (Zephyr 4.1 LVGL memory-pool bug, ZMK #3219). **If the screen still won't
refresh** after flashing, the value must be patched in ZMK source
(`app/src/display/Kconfig`) — a `.conf` override doesn't always take.

If typing ever feels laggy after switching devices: that's usually the **host**
renegotiating a slower BLE connection interval (Windows especially ignores the 7.5 ms
preference), not a lost setting — the conf tuning is intact. A re-pair often resets it.

---

## Board target

Built for `nice_nano//zmk` (Zephyr 4.x hardware-model naming). Builds run via GitHub
Actions; flash the per-half `.uf2` artifacts from the Actions run.
