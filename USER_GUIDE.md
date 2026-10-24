# Active Light Optimizer (ALO) 1.0.0: User guide

Read `README.md` first for the install. Read `LIMITS.md` before you trust a number.

## 1. How it works in one minute

Rhino and Grasshopper run the small component. The heavy planning runs in a second program, the external CPython you installed. The planner works out, for the whole day, where each window's beams should sit. It saves the result as a plan. The component then looks up the pose for your clock minute. A clock change only costs the lookup, so it is instant. A change of window, slider, days, weather or building starts a new plan.

## 2. Component inputs

All inputs are optional. Add them on the component with these exact names.

| Input | Type | Meaning |
|---|---|---|
| `run` | bool | True: plan again when inputs changed. False: keep the last plans and only look up poses. |
| `windows` | list of text or Plane | One window per item, 1 to 15. A Plane gives the outward window normal. Text is a JSON spec (below). No window means one default north window. |
| `north` | number | Angle from the model's +Y axis to true north, clockwise, in degrees. 0 (default) = +Y is north. Example: a model drawn with north pointing to -Y uses 180. It turns Plane inputs and a JSON `plane` only; a JSON `facing` is already a compass bearing and is not changed. |
| `epw` | text | Weather file (.epw). Order: this input, then `engine_config.json`, then `ALO_EPW`. There is no built-in default: with none set the component shows "ERROR: no weather file set ...". See "Weather" below. |
| `days` | text | "dayahead" (default), "all" (full year), or day numbers such as "171,354" (0 = 1 January). |
| `slider` | number | 1 = light (default). 0 = comfort. Between 0 and 1 mixes the two. |
| `month`, `day`, `hour`, `minute` | integer | The clock, in the LOCAL STANDARD TIME of the weather file (no daylight saving). Default 1 May 10:00. |
| `building` | text | Optional building JSON or a file path. Comfort mode only. See section 7. |
| `background` | bool | True (default): plan in a second process and keep Rhino usable. |
| `nproc` | integer | Number of worker processes for planning. Default from `engine_config.json`, which ships as 4; set it to the number of CPU cores of your PC (lower it if Rhino must stay responsive while planning). |
| `config` | text | Full path of `engine_config.json`. Optional; it wins over every other source (section 5). |
| `plans_file` | text | A plan file made elsewhere (section 8). |
| `scene` | scene (dict), JSON text or file path | Optional. Your own window mechanism, made with the ALO nodes (section 12), or a scene.json path. Not set = the shipped base window. The status ends with "scene: base" or "scene: custom (<hash8>)". |

Weather. The Active Light Optimizer does not choose a weather file for you. Before the first plan, set `epw` in `engine_config.json` (next to RELEASE.txt) to the full path of an EnergyPlus weather file (.epw), or set the environment variable `ALO_EPW`, or give the `epw` input of the component. Without it the component shows: ERROR: no weather file set. Run `check_install.cmd` to test the setting. The folder `optional\weather\Williamtown` contains the Williamtown (Newcastle, NSW) TMYx 2011-2025 weather file the tool was developed with. It is used only if you point `epw` at it. Credit: Climate.OneBuilding.Org (see THIRD_PARTY_NOTICES.md).

Time. `month`, `day`, `hour` and `minute` are the local standard time of the weather file (no daylight saving). For a weather file outside Australian Eastern Standard Time, give the local standard time of that file's location.

Comfort mode and your weather. Comfort mode (slider below 1) uses your weather file (outdoor temperature, sun, wind, sky temperature, ground temperature and the comfort band follow your climate), except when the file is the Williamtown reference station (WMO 947760): then the stored reference weather is used. The status line says "COMBINED weather: reference (Williamtown)" or "COMBINED weather: your EPW (<city>)". The room is still the one reference room (see section 6 and LIMITS.md); only the weather follows your file.

Window JSON keys: `id`, `name`, `facing`, `tilt_deg`, `level_m`, `scale`, `width_f`, `beam_w`, `area_m2`, `plane`. Other keys are ignored with the message "Unknown parameter ignored: <key>".

Example: `{"id": "W1", "facing": 180, "tilt_deg": 90, "level_m": 1.0}`

## 3. Component outputs

| Output | Meaning |
|---|---|
| `ring` | Ring angle for each window, degrees. |
| `aperture` | Aperture for each window, percent closed. |
| `ids` | Window names, same order. |
| `status` | One line of status or an ERROR. |
| `a` | JSON: release, package_sha, comfort_band, comfort_room, north_deg, switches (each with its one-line limit), unknown_keys, classes, code, warning, hard_warnings. |

## 4. Day-ahead or all days

* "dayahead" plans three days: the day before, the clock day and the day after. This is the fast choice for the light slider.
* "all" plans the full year. It takes much longer and is an offline job. Start it, keep working, come back later.
* For slider below 1 the component requires days = "all". Otherwise it stops with "set days=all" and starts no plan.

## 5. Settings: `engine_config.json` and environment

| engine_config.json key | Environment variable | Meaning |
|---|---|---|
| `python` | `ALO_PYTHON` | Full path of the external CPython 3.9 or newer. |
| `epw` | `ALO_EPW` | Weather file (.epw). No default: set one (section 2, Weather). |
| `cache_dir` | `ALO_CACHE` | Cache folder. Default `%LOCALAPPDATA%\ActiveLightOptimizer`. |
| `eplus_ref` | `ALO_EPLUS_REF` | Folder holding the reference room data (`A04_felix_open`). |
| `nproc` | `ALO_NPROC` | Planning processes. Ships as 4; set it to your number of CPU cores. |

`ALO_HOME` points at the package folder. A variable always wins over the file. The older names `WO_PYTHON`, `WO_EPW`, `WO_CACHE`, `WO_EPLUS_REF`, `WO_NPROC`, `WO_CONFIG` and `WO_HOME` still work as a fallback; if both are set, the `ALO_` name wins. Relative paths in the file are read from the folder of the file.

Caches live only under `%LOCALAPPDATA%\ActiveLightOptimizer`, in `plans`, `scen`, `gh_bg` and `logs`. Deleting them is safe.

**How the node finds the engine**

The engine ships inside the plugin as a zip. The node unpacks it once to `%LOCALAPPDATA%\ActiveLightOptimizer\engine`. To find the settings file it looks here, in this order, and the first match wins:

1. the `config` input (path of an `engine_config.json`),
2. the variable `ALO_CONFIG`,
3. `ALO_HOME\engine_config.json`,
4. `%LOCALAPPDATA%\ActiveLightOptimizer\engine_config.json` (your own settings; they survive plugin updates),
5. the `engine_config.json` bundled in the plugin,
6. next to the component file.

The first file found is used. Its keys `python`, `epw`, `cache_dir`, `eplus_ref` and `nproc` apply, and the variables `ALO_PYTHON`, `ALO_EPW`, `ALO_CACHE`, `ALO_EPLUS_REF` and `ALO_NPROC` override the file key by key.

The engine folder (`lab\engine\engine.py`) is taken from `ALO_HOME`, then `home` in the config, then the folder of the config, then the installed plugin, then the component's own folder.

The node tells you what it used. `status` ends with `| engine: <folder> (from ...)`, and the `a` JSON has `engine` and `config`, each with its source (config input, ALO_CONFIG, ALO_HOME, user config, plugin package, component folder). A `ALO_HOME` left over from an old install overrides the plugin on purpose. If `status` shows `(from ALO_HOME)` and you did not mean it, delete that variable.

The neutral example `ActiveLightOptimizer_example.gh` holds the node, a Plane input, a `run` toggle (off), a `days` panel and output panels. Switch `run` on to start the first plan.

Fallback without the plugin (zip install, not the main route): unzip the engine anywhere, set `ALO_HOME` to that folder (or give the `config` input the path of `engine_config.json`) and paste the component source `lab\engine\gh_engine_component.py` into a Python 3 script component.

## 6. Switches and what they say

Visible in the component: slider, days, background. The others are set in `engine_config.json`, in the window JSON or in the building JSON. Each text below is the exact limit that comes with that switch.

**days**

For slider < 1 the component requires days = "all" and refuses otherwise.

**comfort_band** (default "standard", on)

Comfort is counted with the European adaptive standard (EN 16798-1:2019, category II: about 3 K above and 4 K below a temperature that follows the recent outdoor weather), only from 07:00 to 23:00. Night hours, for example sleeping in a bedroom, are not counted. The tool scores one room (the example flat's living room); the result is not a prediction for your room unless it is similar. Cold and heat-stress hours are degree-hours outside that range, not counts of unhappy occupants. Agreement with an independent implementation of the standard is within 0.5 % in whether an hour is comfortable; the first three weeks of the year are slightly less exact (running average starts from an assumption). The EnergyPlus status column uses the older 2007 symmetric rule and will differ in cold weather.

**comfort_band = "legacy"** (programming interface only, off)

legacy: the older 24-hour counting, kept only to reproduce numbers from before release. It scores roughly two to three times as much cold discomfort because it includes the night.

The band changes the reported comfort figures (kelvin-hours below the band), but in our 12-day test it did not change which windows get opened or shut, because both bands rank the days the same way. Do not expect a different plan from switching the band.

**comfort room** (always shown in `a` as `comfort_room`)

Comfort is computed for one reference room (A04_felix_open). Windows in other rooms use the same room; compare designs, do not read absolute room temperatures.

**building** (off unless you pass a building JSON in comfort mode)

O13 (doors, ventilation, MVHR) is a building-physics layer checked against EnergyPlus reference runs, not against real houses; off unless you describe the openings. Cross-ventilation is checked against EnergyPlus AirflowNetwork with limits (same pressure coefficients, constant cracks, door Cd 0.62 vs about 0.73). Daily-maximum temperature error is about 0.05 to 0.10 K (daily RMSE); the hourly room-air error is larger: 0.16 K for cross-ventilated rooms and 0.41 K for single-sided rooms modelled by the airflow network, and 0.3 to 0.95 K hourly for the simple ventilated reference rooms with the default wind model. Hours judged open agree with EnergyPlus only 75 to 93 % of the time; do not read airflow numbers as predictions. Light-building hot-day peak error with single-sided wind: north -0.78, east -1.80, south -1.66 K. Fan at or above 25 C and vent modulation are assumptions. The absolute result inherits the core's cold bias: use it to rank designs and relative changes (door open/closed, vent on/off).

**building.doors** (off unless listed)

Door mixing follows the Brown-Solvason model, checked against 18 EnergyPlus door-mixing references; opening a door lowers room temperature by about 0.3 K more than the reference in the medium building class.

**building.vents** (off unless listed): the text under "building" applies.

**vents[].cw_model** (default "default"; opt-in "eplus_exact", building JSON only)

cw_model=eplus_exact: single-sided wind effectiveness follows the EnergyPlus ZoneVentilation:WindandStackOpenArea formula, Cw = 0.55 - 0.25 x (wind incidence angle / 45 deg), zero beyond 90 deg. Checked against EnergyPlus on 9 ventilated reference rooms (public cases, the same ones used to develop it): it lowers the hourly room-air error on all 9 (by about 20 to 56 %) and the hot-day peak of the east and south light rooms; but the hot-day peak error of the north light room and of the heavy and medium rooms moves away from EnergyPlus, by up to 0.6 K (worse on 7 of 9). It is not the AirflowNetwork model: rooms modelled by AirflowNetwork still differ clearly. Not validated against a measured building.

**vents[].dt_min** (default -100, meaning no margin)

dt_min is a user parameter, not an EnergyPlus match. Opening windows earlier (a stricter rule) made the match to EnergyPlus worse.

**building.mv** (ventilation with heat recovery, off)

MVHR heavy-building peak is about 0.5 K under-delivered against the reference.

**building.fan** (ceiling fan, off)

Ceiling fan on/off is an assumption with a hard step at 25 C. Above 25 C it is UNVALIDATED.

Fixed and hidden (you cannot change them): the beam width (0.10 per window unless you set `beam_w`), the sun and pose settings, the comfort category (2) and the light planner. Building class defaults to "medium" and terrain to "suburbs".

## 7. Building recipe (doors and vents)

Skip this unless you want doors or vents. Pass the JSON as the `building` input, in comfort mode (slider below 1, days "all"). The component then shows a warning in `status`.

A minimal building with two rooms, one door and one vent:

```json
{
  "name": "two rooms",
  "o13": true,
  "class": "medium",
  "terrain": "suburbs",
  "zones": [
    {"id": "cell", "env": "<env value of data\\eplus_ref\\A04_felix_open\\case.json>", "windows": true},
    {"id": "back", "env": "<same format>", "windows": false}
  ],
  "partitions": [{"a": "cell", "b": "back", "area_m2": 21.6, "type": "stud", "azimuth_a": 180}],
  "doors": [{"a": "cell", "b": "back", "w": 0.9, "h": 2.1, "sched": {"open": [7, 22]}, "leak_m2": 0.015}],
  "vents": [{"zone": "cell", "area_m2": 1.0, "height_m": 1.0, "azimuth_deg": 0, "sched": "always", "dt_min": -100}]
}
```

* `env` is the room description. Copy the value of the `env` entry from the `case.json` file in `data\eplus_ref\A04_felix_open` (replace the placeholder text with that JSON object; it holds `floor_m2`, `volume_m3` and the glazing). The second room needs its own values in the same format.
* `sched` is "always", "never", a list of 24 numbers from 0 to 1, or `{"open": [7, 22]}` (open from 07:00 to 22:00).
* Doors need the zone ids in `a` and `b`. Width `w` and height `h` are in metres.
* Vents may also take `months`, `tin_min`, `tin_max`, `tout_min`, `tout_max`, `wind_max`, `cross`, `cw`, `cd` and `cw_model`. For the opt-in wind formula add `"cw_model": "eplus_exact"` and read its text in section 6. Do not set a numeric `cw` together with a `cw_model`: that is an error.
* Optional: `"mv": [{"zone": "cell", "ach": 0.5, "hr_eff": 0.75}]` and `"fan": {"air_speed": 0.6}`.
* `"o13": false` or no building at all switches the layer off.

Use this layer to rank designs (door open or closed, vent on or off), not to predict a real house.

## 8. If the second program cannot run (plans_file)

Option 3 is built in. On any computer that has CPython 3.9 or newer, run:

`python lab\engine\engine.py plan scene.json plans.json --days all --slider 0.5 --cache C:\some\folder`

`scene.json` holds `{"windows": [ ... ]}` in the window JSON format. Then give `plans.json` to the `plans_file` input. The component checks that the plans fit the current windows, days and mode, and shows a banner if not.

## 9. Run times

First run is the slow one. Plans run in the background and are cached on disk. The numbers are for an idle PC, this test PC (4 cores), cold cache, Python 3.9.12. Other PCs differ. Times are medians of 3 runs (source: T11); the window-count rows are single runs (source: T12).

nproc 4 (the shipped setting), light slider:

| Case | Time |
|---|---|
| 1 window, 12 days | 88.6 s |
| 3 windows, 12 days | 217.6 s |
| 1 window, day-ahead (3 days) | 74.5 s |
| 3 windows, day-ahead (3 days) | 162.9 s |
| Comfort, case C5, 2 days | 56.9 s |

Window count, 12 days, nproc 4 (T12):

| Windows | Time | Peak memory (all processes) | Disk cache |
|---|---|---|---|
| 1 | 89 s | 1126 MB | 11 MB |
| 3 | 216 s | 1474 MB | 28 MB |
| 5 | 416 s | 2057 MB | 42 MB |
| 15 | 1219 s (about 20 min) | 2721 MB | 114 MB |

nproc 1 (older anchors, 12 days, cold cache):

| Case | Time |
|---|---|
| nproc 1, light, case fixed_1 | 146 s |
| nproc 1, light, case fixed_3 | 397 s (range 373 to 464 s) |
| nproc 1, comfort, case fixed_3, slider 0.5, 2 days | about 162 s |

A full-year plan is an offline job. Other computers will differ.

## 10. Troubleshooting by message

Each message appears in `status`. `<...>` is filled in by the tool.

* **ERROR: interpreter not found: <path>: set ALO_PYTHON**: The external Python did not start. Check the path in `engine_config.json` or `ALO_PYTHON`. Variants: "(it did not run: exit N)" means the program started and failed, so reinstall Python; "(no answer within 3 s)" means it is too slow or blocked, so try again.
* **ERROR: interpreter too old (<x.y>): <path>: set ALO_PYTHON to CPython 3.9 or newer**: Install CPython 3.9 or newer and point `python` at it.
* **ERROR: cache folder not writable: <path> (also tried <LOCALAPPDATA path>): set ALO_CACHE to a writable folder**: The tool cannot write there. Set `cache_dir` or `ALO_CACHE` to a folder you may write to.
* **ERROR: no weather file set: set "epw" in engine_config.json, or the ALO_EPW environment variable, or the epw input, to the full path of a .epw file (optional Williamtown file: optional\weather\Williamtown\)**: No weather file is chosen for you. Set one as the message says. The optional Williamtown file is in the package folder `optional\weather\Williamtown`.
* **ERROR: no weather file set: no weather file is chosen automatically: <hint>** (shown by `check_install.cmd`): Same cause and fix as the previous "no weather file set" message: set `epw` in `engine_config.json`, or `ALO_EPW`.
* **ERROR: weather file not found: <path>: set the epw input or ALO_EPW**: Give the `epw` input, `epw` in the config, or `ALO_EPW` a real file.
* **ERROR: comfort reference data A04_felix_open not found in <dir>: set ALO_EPLUS_REF (comfort needs it; light mode does not)**: Point `eplus_ref` at the folder `data\eplus_ref` of the package, or use slider 1.
* **ERROR: <n> windows: the maximum is 15**: Use 15 windows or fewer.
* **ERROR: window <id>: plane normal/origin is not finite**, **ERROR: window <id>: plane normal has zero length**, **ERROR: window <id>: <key> is not a finite number**: A plane or number is empty or broken. Check the Rhino geometry and the wiring.
* **ERROR: window <k>: not valid JSON: <detail>**, **ERROR: window <k>: the JSON must be an object**, **ERROR: window <k>: give a JSON spec or a Plane, got <type>**: Fix the window item `k` as the message says.
* **ERROR: window ids are not unique: [...]**: Give each window its own `id`.
* **set days=all: slider < 1 (comfort) needs days = "all"; the comfort plan judges the whole year (a day subset is only an approximation). No plan was started.**: Set `days` to "all", or set the slider to 1.
* **ERROR: slider must be between 0 (comfort) and 1 (light): <v>** and **ERROR: slider must be a number 0 (comfort) to 1 (light): <v>**: Use a number from 0 to 1.
* **ERROR: days must be "dayahead", "all" or day numbers like "171,354": <v>** and **ERROR: days must be numbers 0 to 364 (0 = 1 Jan): <v>**: Fix the `days` text.
* **ERROR: invalid AEST clock month M day D h:mm (non-leap year)**: The date does not exist. Check month, day, hour and minute. (The word AEST in the message means Australian Eastern Standard Time, UTC+10, no daylight saving: the local standard time of the weather file.)
* **ERROR: building JSON is not valid (or the file cannot be read): <detail>** and **ERROR: building must be a JSON object**: Check the building text or file path (section 7).
* **ERROR scene: the scene text is not valid JSON (<detail>)**, **ERROR scene: the scene JSON must be an object {...}**, **ERROR scene: no scene given (empty text). Give a scene.json path or the scene JSON.**, **ERROR scene: scene file not found: <path>**, **ERROR scene: cannot read scene file <path> (<detail>)**, **ERROR scene: <path> does not hold a JSON object {...}**, **ERROR scene: a scene must be a dict, a JSON text or a scene.json path (got <type>)**: The `scene` input is not a readable scene. Wire the ALO Scene output, or give a correct scene.json path.
* **ERROR scene: the scene has no "<key>" entry**: A required entry (for example params, joints, bodies, targets) is missing from the scene. Build the scene with the ALO Scene node, or add the entry.
* **ERROR scene: units_to_m must be a number > 0 (metres per model unit), got <v>**: Give the number of metres in one model unit, for example 0.001 for millimetres.
* **ERROR scene: north_deg must be a number (degrees), got <v>**: Give the north angle as a plain number of degrees.
* **ERROR scene: exterior_dir must be three numbers [x, y, z], got <v>**: Give three numbers, or leave it empty to have it estimated.
* **ERROR scene: exterior_dir has zero length: give the direction pointing from the room out through the window**: Give a direction that is not [0, 0, 0].
* **ERROR scene: exit_point must be three numbers [x, y, z] (the window centre on the wall), got <v>**: Give three numbers for the window centre on the wall.
* **ERROR scene: params must be a list of {name, values}**: The parameters entry must be a list. Wire ALO Parameter nodes into the ALO Scene node.
* **ERROR scene: parameter <n> has no name**, **ERROR scene: parameter <name> needs a list of at least 2 numbers in "values"**, **ERROR scene: parameter <name> values must be strictly increasing (...)**, **ERROR scene: parameter <name> drives no joint: ...**: Give every parameter a name, at least two increasing values, and at least one joint that uses it.
* **ERROR scene: two parameters are called <name>: parameter names must be unique**: Rename one of the two ALO Parameter nodes.
* **ERROR scene: joints must be a non-empty list: the window needs at least one revolute joint**: Wire at least one ALO Joint node.
* **ERROR scene: two joints are called <name>: joint names must be unique**: Rename one of the two joints.
* **ERROR scene: bodies must be a non-empty list (housing, rings, blades ...)**: Wire at least one ALO Body node.
* **ERROR scene: two bodies are called <name>: body names must be unique**: Rename one of the two bodies.
* **ERROR scene: internal check failure (<type>: <detail>)**: The scene check itself failed. Check the scene for broken or missing values and try again; if it persists, keep the message for the developer.
* **ERROR: north must be a number of degrees**: The `north` input is empty or not a number. Give a number such as 0 or 90.
* **ERROR scene: the planner supports exactly 2 parameters (steering rotation first, opening drive second); this scene has N**: Use exactly two ALO Parameter nodes in 1.0.
* **ERROR scene: joint <name>: only revolute joints are supported in 1.0 (no sliding / prismatic joints)**: Sliding joints come in 1.1. Use hinged (revolute) joints.
* **ERROR scene: joint <name>: the axis has zero length (axis curve of length 0): draw the hinge axis with two different points**, **ERROR scene: joint <name> references unknown parameter <p> (parameters: ...)**, **ERROR scene: body <name> chain names unknown joint <j> (joints: ...)**: Fix the joint axis, the parameter name or the body chain as the message says.
* **ERROR scene: no body has a clear-opening loop (mouth): the window has no opening. Give the housing / rings their mouth polygons**, **ERROR scene: body <name>: mouth N has zero area (check that its points are not collinear or repeated)**: Give a body of role mouth a valid opening outline.
* **ERROR scene: targets must be a non-empty list of room faces (the surfaces the beam may land on)**: Give at least one ALO Room Target.
* **ERROR scene: no opening is clear at any pose: blockers cover every mouth (area 0 at all N poses)**: The blockers cover the whole opening at every pose. Check the blocker geometry and the joint gains.
* **ERROR scene: the mechanism could not be evaluated (<type>: <detail>). Check joint chains, axes and polygons**: Check the chains, axes and polygons of the scene.
* **WARNING scene: blocker part of body <b> is not long and thin enough (length / width = x.x, needs >= 3): ...**: The body is still produced, but the blocker is approximated coarsely. Split it into several simpler parts, each its own ALO Body (1.0 limit, see LIMITS.md).
* **WARNING scene: opening area 0 at N of M poses, e.g. ...**: The window is fully closed at those poses; check this is intended.
* **ERROR: engine folder not found (looked for engine.py in <dirs>): set ALO_HOME to the package folder or give the config input the path of engine_config.json**: With the plugin, the install is damaged or partly done. In `_PackageManager` uninstall and install again, then restart Rhino. If it still fails, delete `%LOCALAPPDATA%\ActiveLightOptimizer\engine` so the node unpacks it again. Otherwise set `ALO_HOME`, or fill the `config` input.
* **The node is missing from the ribbon**: Rhino was not restarted after the install, or Grasshopper was open. Close and reopen Rhino. In `_PackageManager` check that ActiveLightOptimizer is listed under Installed.
* **ERROR: engine could not be loaded from <dir>: <detail>** and **ERROR: engine_config.json is not valid JSON: <path>: <detail>**: A package file is broken or the config has a typo. Reinstall the package or fix the commas and quotes in the config.
* **ERROR: plans_file not found: <path>**, **ERROR: plans_file is not valid JSON: <path>: <detail>**, **ERROR: plans_file is not a plan file (expected {window id: plan with runs}): <path>**: Check the plan file (section 8).
* **ERROR: windows without a plan: [ids] (the plans are for [ids])**: The plan file is for other windows. Make a new one.
* **ERROR: <ExceptionType>: <message>**: Any other failure. Run `check_install.cmd`. If it passes, keep the message and `a` for the developer.

Warnings added to `status` after " | ":

* **BACKGROUND PLAN FAILED: <err>**: The planner stopped with that error. Fix the cause and set `run` to True.
* **background process exited (code N) without plans: relaunched**: The tool started the planner again by itself.
* **background planning gave up after 3 tries (see <status file>)**: Open that file in `%LOCALAPPDATA%\ActiveLightOptimizer\gh_bg`. Run `check_install.cmd`.
* **showing the PREVIOUS plans while the new ones are computed**: Normal after a change. The numbers are old until the new plan is ready.
* **the stored plan file was damaged and was removed: planning again**: Nothing to do.
* **cache folder <p> is not writable: using <q>**: The tool used the fallback folder. Fix `cache_dir` if you want your own.
* **KEY MISMATCH (plans_file): ...**: The plan file does not fit the current inputs. Make a new one.
* **building ignored: the O13 building layer acts only in comfort mode (slider < 1)**: Lower the slider and set days to "all", or drop the building.
* **NOT PLANNED for this day: [ids] (held pose)**: The listed windows have no plan for this day (for example, no sun). They keep their last pose.
* **WARNING opt-in with known limits active: ...**: You use building features. The full text is in `a`, key `hard_warnings`. See `LIMITS.md`.

## 11. Privacy

Nothing leaves your machine. The tool opens no network connection. Plans, caches and logs stay in the folders named above.

## 12. Your own window: the ALO nodes

The built-in window is the default. To plan for your own mechanism, build it from five nodes (category Active Light Optimizer) and wire the result into the `scene` input of the Active Light Optimizer node. The nodes only describe the window; they do not plan. The planning is done by the Active Light Optimizer node.

1. **ALO Parameter**: one movement parameter with its name, its minimum, maximum and number of steps. Release 1.0 needs exactly two: the steering rotation first (0 to 15 degrees), the opening drive second (0 to 100 percent). Other ranges give a WARNING, because the planner assumes these two.
2. **ALO Joint**: a hinge. Give its name, the parameter that drives it, the hinge axis as a line (origin to end) and a gain (the signed degrees of rotation per unit of the parameter). Revolute joints only. The sign of the gain is not guessed: check it in the pose preview.
3. **ALO Body**: a rigid part. Give its name, its role (mouth = a clear opening, blocker = a plate that shades), its geometry and its chain of joints, root joint first. A body with no chain does not move.
4. **ALO Room Target**: a room face the beam may land on (the room surfaces and, if you have them, hero curves and a room point).
5. **ALO Scene**: collects the parameters, joints, bodies and targets, plus the units, the north angle and the exterior direction (empty = estimated), and returns the `scene`, a status and a pose preview. Give `scene_path` to also write a scene.json file.

Pose preview. The ALO Scene node shows the window at all-minimum, a middle pose, all-maximum and one pose you choose. Check that the rings and blades open and steer the right way before you plan.

Scene check. Before planning, the scene is checked. Problems appear as "ERROR scene: ..." (the planner does not start; the messages are listed in section 10) or "WARNING scene: ..." (the plan runs). With a scene the status ends with "scene: custom (<hash8>)"; without one, "scene: base".

The `scene` input accepts the ALO Scene output, a JSON text, or the path of a scene.json file. The plan cache includes the scene, so changing the scene plans again.

Beam accuracy of custom windows. Beam accuracy was validated against an independent ray cast for the reference window only. Custom mechanisms built with the ALO nodes are validated for scene geometry, movement (pose preview) and plan consistency; their beam values are not independently validated in 1.0. An independent beam check of custom mechanisms is planned for 1.0.1. Blocker plates are sampled in sections, so the opening area can deviate by about 0.15 % (up to 0.5 %) from an exact calculation; thin or compact blockers deviate more. See `LIMITS.md`.

Limits in 1.0: two movement parameters, revolute joints only, and blocker parts must be long and thin (plates, blades, bars; length / width at least 3). One-parameter types (casement, awning, pivot, louvres), sliding joints and more than two parameters come in 1.1. See `LIMITS.md`.
