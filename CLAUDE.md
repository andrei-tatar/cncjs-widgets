# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Custom widgets for [CNCjs](https://github.com/cncjs/cncjs). Each widget is one self-contained static HTML file at `<name>/index.html` (inline CSS and JS, ES5 style, no dependencies). There is no build step, package manifest, linter or test suite.

## Running a widget

A widget only works when served by CNCjs itself (same origin), because it loads `/socket.io/socket.io.js` from CNCjs and reads the session token from CNCjs's `localStorage`. Opening the file directly or from another server will not connect.

```sh
cncjs -m /widgets:/path/to/cncjs-widgets
```

Then in the CNCjs workspace: **Manage Widgets** → enable **Custom** → set its URL to `/widgets/<name>/`. Edits take effect on reload of the widget iframe; there is nothing to rebuild.

## How a widget talks to CNCjs

The widget runs in an iframe inside the CNCjs Custom widget and uses two channels (see `atc/index.html`):

- **socket.io to the CNCjs server**, authenticated with the session token. The token comes from the `?token=` query parameter, else `localStorage.cnc` (`state.session.token`), else the first `postMessage` from the parent. The widget emits `list` and reads `serialport:list` (polled every 3 s) to find the open port (`inuse`), and sends lines with `socket.emit('writeln', port, line)`. It also emits `open` for that already-open port, which only registers the socket with the controller so it receives `controller:state`; the current tool is `state.parserstate.tool`.
- **`postMessage` with the parent CNCjs window**, in the shape `{ token, action: { type, payload } }`. The widget sends `connect` (ask which port the UI is using) and `resize` (with `scrollHeight`, so the iframe fits its content; call it after any layout change). CNCjs sends `change` with `payload.port`, which is preferred over the first open port when several are open.

Send tool-change commands with `writeln`, not the `gcode` command: CNCjs intercepts `M6` on the `gcode` path, comments it out and pauses the feeder. `writeln` is the same path as typing in the CNCjs console, and it writes straight to the serial port with no flow control. Long sequences without `M6` (the probe widget) go through `socket.emit('command', port, 'gcode', text)` instead, because the feeder waits for each `ok`.

## atc widget

Commands are defined in the `ACTIONS` map (action key → label and a function from tool number to the lines sent). The buttons' `data-action` attributes are the keys, so adding an action means adding a map entry and a button. The commands assume a controller that supports `M61 Q<tool>` and `$LocalFS/Run=` (e.g. FluidNC), with `atc_measure.nc` present on the controller's local filesystem. Keep the command table in `README.md` in sync when changing them.

## probe widget

Probes with an XYZ touch plate hooked over a stock corner. `ACTIONS` maps each button's `data-action` to the corner's `sx`/`sy` (direction from the corner into the stock; absent for Z only). `buildLines` generates the sequence in relative moves from a start above the plate centre: touch Z, then for each axis move outside the lip, drop, touch its outer face and set the zero with `G10 L20 P0`. `FIELDS` defines the settings panel and its defaults (the Maker Store TOOL-TP-V2); settings and the tool diameter are stored in `localStorage` under `cncjs-widgets.probe`. Keep the settings table in `README.md` in sync when changing defaults.
