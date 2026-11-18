# cncjs-widgets

Custom widgets for [CNCjs](https://github.com/cncjs/cncjs). Each widget is a single static HTML file with no build step.

## atc

Two buttons for a tool changer. Each asks for a tool number, then sends:

| Button      | Sent to the controller                              |
| ----------- | --------------------------------------------------- |
| Init tool   | `M61 Q<tool>` then `$LocalFS/Run=atc_measure.nc`    |
| Switch tool | `T<tool> M6`                                        |

The widget shows the controller's current tool as `T<n>` and offers only the button that applies: **Init tool** while no tool is selected (`T0`), **Switch tool** once one is. Until the controller has reported its tool, both are shown.

## probe

Zeroes the work coordinates with an XYZ touch plate that hooks over a corner of the stock. Defaults match the [Maker Store XYZ Touch Probe (TOOL-TP-V2)](https://www.makerstore.com.au/product/tool-tp-v2/): 5 mm plate thickness, 10 mm lips, 50 mm square.

| Button                                           | Result                                                    |
| ------------------------------------------------ | --------------------------------------------------------- |
| Z only                                           | Z0 at the top of the material                             |
| Front left, Front right, Back left, Back right   | Z0 at the top of the material, X0 Y0 at that corner       |

To use it, enter the tool diameter, place the plate, clip the lead to the tool and jog to about 5 mm above the centre of the plate, then pick a button and confirm. A corner probe touches the top of the plate, then moves outside each lip, drops beside it and touches its outer face. It ends above the centre of the plate. Every touch is a fast pass followed by a slow one. Zeros are set in the active work coordinate system with `G10 L20 P0`.

Buttons are enabled only while the controller is idle. **Stop** soft-resets the controller. A **Probe contact** badge shows while the probe input is active, which is a quick way to check the wiring: touch the plate to the tool.

**Settings…** holds the plate dimensions, probe distances and feed rates. They are saved in the browser.

| Setting                          | Default    |
| -------------------------------- | ---------- |
| Plate thickness (Z)              | 5 mm       |
| Lip width (X/Y offset)           | 10 mm      |
| Plate size                       | 50 mm      |
| Side probe depth below plate top | 6 mm       |
| Side clearance outside plate     | 10 mm      |
| Max Z probe travel               | 20 mm      |
| Retract after contact            | 2 mm       |
| Fast probe feed                  | 100 mm/min |
| Slow probe feed                  | 25 mm/min  |

## Install

1. Mount this repository in CNCjs, either on the command line:

   ```sh
   cncjs -m /widgets:/path/to/cncjs-widgets
   ```

   or in `~/.cncrc`:

   ```json
   {
     "mountPoints": [
       { "route": "/widgets", "target": "/path/to/cncjs-widgets" }
     ]
   }
   ```

2. Restart CNCjs. In the workspace, open **Manage Widgets**, enable **Custom**, then open the Custom widget's settings and set the URL to `/widgets/atc/` or `/widgets/probe/`.

A widget must be served by CNCjs itself (same origin): it loads the socket.io client from CNCjs and reads the session token from it.
