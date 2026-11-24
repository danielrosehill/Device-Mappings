# Device Mappings

An index of input devices I have identified and mapped on Linux: what each one presents to the system
(USB vendor:product, driver, evdev node) and what each physical button sends. Each entry is its own repo; this one
is the index.

## Devices

| Device | Type | USB ID | Driver / route | Mapping format | Repo |
|---|---|---|---|---|---|
| Logitech F310 (XInput mode) | Gamepad | `046d:c21d` | `xpad` | evdev + browser Gamepad API indexes + SDL line, all buttons press-verified | [Logitech-F310-Gamepad-Linux-Profile](https://github.com/danielrosehill/Logitech-F310-Gamepad-Linux-Profile) |
| 8BitDo Micro (D mode, Bluetooth) | Gamepad | `2dc8:9020` | `hid-generic` | evdev + joydev indexes + SDL line (untested); home and star not pressed | [8BitDo-Micro-Gamepad-Linux-Profile](https://github.com/danielrosehill/8BitDo-Micro-Gamepad-Linux-Profile) |
| Jabra Evolve 30 II | Headset call buttons | `0b0e:0314` | `usbhid`, hidraw listener → uinput | decoded hidraw protocol, hwdb (volume keys off), centre button → F13 | [Jabra-Headset-Button-Hook](https://github.com/danielrosehill/Jabra-Headset-Button-Hook) |
| Olympus RS-27H | Foot switch | `07b4:0218` | `usbhid` | identity + descriptor; per-pedal codes **not yet captured** | [Olympus-RS27H-Foot-Switch](https://github.com/danielrosehill/Olympus-RS27H-Foot-Switch) |
| Generic USB HID button | Single button | `5131:2019` | `hid-generic`, remapped with `keyd` | keyd config (button → F13) | [USB-Button-Remap](https://github.com/danielrosehill/USB-Button-Remap) |
| K811 wired gaming keypad | Macro keypad | `5566:000a` | `hid-generic` | `evtest` event captures | [K811-EV-Test-Events](https://github.com/danielrosehill/K811-EV-Test-Events) |
| USB HID foot mouse | Foot pedal / mouse | `3553:b001` | `hid-generic`, udev hwdb | hwdb remap + `evtest` notes | [USB-Foot-Mouse-Linux](https://github.com/danielrosehill/USB-Foot-Mouse-Linux) |

Machine-readable copy: [`devices.json`](devices.json) (`schema: device-mappings-index/1`). Agents look devices up by
`usb` (vendor:product) there, then read `profile` in the linked repo.

## Is there a standard format?

There is no single cross-device standard. Each class of device has a de facto one, and an entry should use the one
that matches its class, plus a plain JSON profile so any tool can read it:

| Device class | De facto format | Who reads it |
|---|---|---|
| Gamepads, joysticks | **SDL GameController mapping string** ([SDL_GameControllerDB](https://github.com/mdqinc/SDL_GameControllerDB)): `GUID,name,a:b0,b:b1,...,platform:Linux` | SDL games and apps, Steam, Godot, many emulators |
| Gamepads in a browser or Electron app | **W3C Gamepad "standard" mapping** (button indexes 0-16) | `navigator.getGamepads()` |
| Keyboards, macro pads, remotes, pedals that send keys | **udev hwdb** (`evdev:input:b0003v5131p2019*` + `KEYBOARD_KEY_<scancode>=<key>`) | the kernel input layer, every desktop |
| Same, remapped in userspace | `keyd` config, or `input-remapper` preset JSON | the remapper daemon |
| Drawing tablets | `libwacom` `.tablet` files | GNOME / KDE tablet settings |

Each device repo here should carry:

1. `device.json`: vendor:product, mode, driver, stable `/dev/input/by-id` path pattern (with the per-unit serial
   left out), and per physical button its evdev code, plus the browser index for gamepads, with a flag saying whether
   it was press-verified.
2. The class format from the table above: the SDL line for a pad, the hwdb snippet for a key device.
3. A README table that a person can read.

## Mapping a new device

Plugin [`input-device-mapper`](https://github.com/danielrosehill/Input-Device-Mapper-Plugin) (Claude Code):
`idmap.py list`, `idmap.py info <vvvv:pppp>`, then `idmap.py press <device> --label <button>` once per button.
Add the new repo to the table above **and** to `devices.json`. To look a connected device up in this library, use the
same plugin's `lookup-device-mapping` skill.
