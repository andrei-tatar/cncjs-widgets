# cncjs-widgets

Custom widgets for [CNCjs](https://github.com/cncjs/cncjs). Each widget is a single static HTML file with no build step.

## atc

Two buttons for a tool changer. Each asks for a tool number, then sends:

| Button      | Sent to the controller                              |
| ----------- | --------------------------------------------------- |
| Init tool   | `M61 Q<tool>` then `$LocalFS/Run=atc_measure.nc`    |
| Switch tool | `T<tool> M6`                                        |

### Install

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

2. Restart CNCjs. In the workspace, open **Manage Widgets**, enable **Custom**, then open the Custom widget's settings and set the URL to `/widgets/atc/`.

The widget must be served by CNCjs itself (same origin): it loads the socket.io client from CNCjs and reads the session token from it.
