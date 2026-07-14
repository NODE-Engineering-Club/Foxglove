# Foxglove
Everything to do with the Graphic User Interface (GUI)

# ASKET 2.0 GUI — Mandatory Layout (`ASKET_GUI_mandatory.json`)

## What it is

A Foxglove Studio layout that shows the jury the minimum required information
about the ASV during a task run. Import it into Foxglove
(Layout menu → Import from file) while connected to the boat's foxglove_bridge
(`ws://<pi-ip>:8765`).

## Where these files live in the repo

1. `gui/ASKET_GUI_mandatory.json` — this layout.
2. `gui/README_ASKET_GUI_mandatory.md` — this file.
3. `src/sensors/sensors/gui_telemetry_hw.py` — the telemetry helper node that
   feeds the gauges + status bar (see its own README).

## What it does

It arranges nine panels into a single jury-readable screen:

| Panel               | Shows                          | Topic it reads                 | Data source                     |
| ------------------- | ------------------------------ | ------------------------------ | ------------------------------- |
| LiDAR (3D)          | Live 2D LiDAR scan             | `/lidar_driver/scan_raw`       | `lidar_driver` (real hardware)  |
| RGB Front Camera    | Front camera feed              | `/front_camera_driver/image_raw` | `camera_driver` (real hardware) |
| Map                 | Boat position on a real map    | `/gps_driver/gps_raw`          | `imu_gps_driver` GPS            |
| Latitude / Longitude| Numeric GPS readout            | `/gps_driver/gps_raw`          | `imu_gps_driver` GPS            |
| ASV STATUS          | Auto / Remote / Standby / Out of control | `/vehicle/status`    | `gui_telemetry_hw` (from MAVROS)|
| Heading gauge       | Compass heading (deg)          | `/heading`                     | `gui_telemetry_hw` (from MAVROS)|
| COG gauge           | Course over ground (deg)       | `/cog`                         | `gui_telemetry_hw` (from MAVROS)|
| Speed gauge         | Speed over ground (kn)         | `/sog`                         | `gui_telemetry_hw` (from MAVROS)|
| Battery gauge       | Battery remaining (%)          | `/battery_percentage`          | `gui_telemetry_hw` (from MAVROS)|

The camera, LiDAR, and GPS panels read the boat's real driver topics directly.
The four gauges and the status bar read topics produced by the helper node
`gui_telemetry_hw.py`, which republishes real MAVROS data under those names.

## How to use it

1. Start the boat stack: `ros2 launch bringup njord.launch.py`.
2. Start the telemetry helper (it produces the gauge + status topics):
   `ros2 run sensors gui_telemetry_hw`.
3. In Foxglove, connect to `ws://<pi-ip>:8765` and import `gui/ASKET_GUI_mandatory.json`.
4. All nine panels should populate.

## Limitations

1. **The four gauges + status bar need `gui_telemetry_hw.py` running.** Without
   it, `/heading`, `/cog`, `/sog`, `/battery_percentage`, `/vehicle/status` do
   not exist and those panels stay empty.
2. **Everything depends on MAVROS being connected.** If `/mavros/state.connected`
   is false, the status bar reads OUT_OF_CONTROL and battery/heading are blank.
   Fix the FCU link first.
3. **Battery reads 0% until the ArduPilot battery monitor is configured.** 0%
   here means "not set up," not "empty."
4. **The LiDAR panel needs a valid `base_link` TF frame.** If the TF tree does
   not publish `base_link`, the panel shows an empty grid.
5. **The camera topic is `/front_camera_driver/image_raw` only when launched via
   `njord.launch.py`** (which remaps it). Run the camera standalone and it
   publishes `/image_raw` instead.
6. **No route comparison line.** The layout shows the boat's live position on the
   map, but does NOT draw the travelled track against the ideal GNSS route.

## Mandatory GUI requirements (Njord 2026, section 11.4)

The competition requires the GUI to be operable by someone with no software
background, and understandable by a jury member with no explanation. It must
display the following data:

| # | Required data                                            | Met by this layout |
| - | -------------------------------------------------------- | ------------------ |
| 1 | Camera feed / LIDAR                                       | Yes                |
| 2 | Latitude                                                 | Yes                |
| 3 | Longitude                                                | Yes                |
| 4 | Heading                                                  | Yes                |
| 5 | Course over ground (COG) with trail, vs plot of the      | Partial — COG value shown; trail + ideal-route overlay not included in this layout |
      ideal route from GNSS points 
| 6 | Speed over ground                                        | Yes                |
| 7 | Battery life in %                                        | Yes                |
| 8 | Status indicator (autonomous / remote / standby / out of control) | Yes       |

Nice-to-have (not required, not included):

1. Distance between ASV and next waypoint.
2. Battery life in Wh remaining.
3. Any other parameters that help the jury understand the ASV.
