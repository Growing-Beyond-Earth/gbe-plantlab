# PlantLab Programming Dictionary

Every object and field accepted in `program.json`, in one place. For learning
material, start with [Programming Your PlantLab](PROGRAMMING_YOUR_PLANTLAB.md)
and [Advanced PlantLab Programming](ADVANCED_PLANTLAB_PROGRAMMING.md).

Formats used below: times of day are `"HH:MM"` or `"HH:MM:SS"` (24-hour);
lengths of time are `"HH:MM:SS"`; dates are `"YYYY-MM-DD"`.

## Top level

| Field | Type | Required | Meaning |
|---|---|---|---|
| `loops` | array | one of these two | The program's loops, evaluated in order once per second; last match wins |
| `default_actions` | actions object | one of these two | Baseline applied when no loop matches (everything defaults to off without it) |
| `format_version` | number | no | Format this program is written to (currently 5). Firmware refuses a program declaring a newer version than it knows. Omit unless the program depends on a version-specific reading (declare 5 when using `target_ppfd`) |
| `program_name` | string | no | Your label; never read by the device |
| `settings` | object | no | Legacy wrapper around `default_actions` + `loops`; both wrapped and flat forms are accepted |
| `version` | string | no | Legacy decorative field; never read |

## Actions object

All fields optional; set only what you want to change. Also accepted as a
one-element array (legacy form).

| Field | Range | Meaning |
|---|---|---|
| `red` | 0–255 | Red LED duty (hardware ceiling 240) |
| `green` | 0–255 | Green LED duty (ceiling 82) |
| `blue` | 0–255 | Blue LED duty (ceiling 99) |
| `white` | 0–255 | White LED duty (ceiling 196) |
| `fan` | 0–255 | Fan speed, full range |
| `aux` | 0–255 | Auxiliary output (relays, valves, other aux devices), full range |
| `target_watts` | ≥ 2 | Scale the written light recipe to this electrical power; under 2 = no target |
| `target_ppfd` | 1–5000 | Scale the written recipe to this photon flux (µmol/m²/s). Works only on devices that report PPFD (see the Est PPFD column in the SD log) — ignored elsewhere and the raw colors run. Wins over `target_watts` if both present |

Values above a channel's ceiling behave as the ceiling; below 0 as 0. Every
field except the two targets also accepts a [`ramp`](#ramp-value) or
[`oscillate`](#oscillate-value) object in place of the number.

## Fields common to all loops

| Field | Type | Meaning |
|---|---|---|
| `type` | string | `"time"`, `"repeat"`, `"repeat_days"`, `"date_range"`, `"sensor"`, or `"api"`. Unknown types never fire (and warn) |
| `id` | string | The loop's stable identity (max 31 characters, unique per program). Used to remember duration timing and day anchors across reboots and program updates, and for named-loop replacement. Renaming = a new loop with fresh memory |
| `actions` | actions object | Applied while the loop is active |
| `loops` | array | Child loops, evaluated only while this loop is active (AND logic; max 6 levels deep) |

## `time` loop

| Field | Required | Meaning |
|---|---|---|
| `start` | no | Window opens (inclusive). Omitted = no lower bound |
| `end` | no | Window closes (exclusive). Omitted = no upper bound. May wrap midnight |
| `duration` | no | How long one occurrence runs from the window opening. With `start` but no `end`: that long, daily. With neither: once, from program load. `"00:00:00"` disables the loop |

No `start`, `end`, or `duration` = active all day, every day.

## `repeat` loop

| Field | Required | Meaning |
|---|---|---|
| `start` | no | First occurrence (default midnight) |
| `interval` | yes | Gap between occurrence *starts* (accepted older spelling: `every`) |
| `duration` | yes | Length of each occurrence, minimum 2 seconds (older spelling: `for`) |
| `until` | no | No occurrence starts at or after this time (default end of day) |

Occurrences never cross midnight. `duration` ≥ `interval` = always on.
Follows the wall clock (no stored state) — a clock correction can replay or
skip an occurrence; use a sensor loop with `hold` for exactly-once dosing.

## `repeat_days` loop

| Field | Required | Meaning |
|---|---|---|
| `start_date` | no | Day the cadence counts from (always fires). Omitted = counts from the day first seen; then `id` is required |
| `interval_days` | yes | Whole days between firing days; 1 = daily (older spelling: `every_days`) |
| `duration_days` | no | Days each occurrence lasts (default 1) (older spelling: `for_days`) |
| `end_date` | no | **Exclusive** — nothing fires on or after this day |

Selects whole days (midnight to midnight); nest `time`/`repeat` loops to pick
hours. Held off until the device clock is set.

## `date_range` loop

| Field | Required | Meaning |
|---|---|---|
| `start_date` | no | First day (inclusive). Omitted with no `duration_days`/`after_days` = no lower bound |
| `end_date` | no | Day after the last day (**exclusive**) |
| `duration_days` | no | Length in days, as an alternative to `end_date`; counts from the day first seen — `id` required |
| `after_days` | no | Delay the start this many days from first seen (`id` required); for chaining phases. Not meaningful alongside `start_date` |

At least one of the four must be present. Held off until the clock is set.

## `sensor` loop

| Field | Required | Meaning |
|---|---|---|
| `condition` | yes | See below |
| `check_interval` | no | How often to re-evaluate (default 1 minute; floor 1 minute) |

### Condition object

| Field | Required | Meaning |
|---|---|---|
| `sensor` | yes | One of the names below |
| `comparison` | yes | `>`, `>=`, `<`, `<=`, `==`, `!=` |
| `value` | yes | Numeric threshold (write it unquoted) |
| `hysteresis` | no | How far the reading must come back past `value` before releasing; only applies while already true. Meaningless for `==`/`!=` |
| `hold` | no | Once triggered, stay true exactly this long, then false until the next check that qualifies. Must be shorter than `check_interval`. Immune to clock steps |

### Sensor names

| Name | Unit | | Name | Unit |
|---|---|---|---|---|
| `temperature` | °C | | `light` | lux |
| `humidity` | % | | `soil_moisture` | % |
| `co2` | ppm | | `power` | W |
| `pressure` | hPa | | `voltage` | V |
| `fan_rpm` | RPM | | `current` | A |

A sensor loop's own `actions` run for as long as the condition holds — timing
belongs on the condition (`hold`) or in a child loop.

## `api` loop

| Field | Required | Meaning |
|---|---|---|
| `url` | yes | HTTP/HTTPS address, fetched with a plain GET (WiFi required; private-network addresses are blocked) |
| `check_interval` | yes | Poll interval (floor 1 minute, cap 24 hours) |
| `match_value` | yes | Text to match in the response (max 64 characters) |
| `match_mode` | no | `"substring"` (default), `"exact"`, or `"http_status"` (e.g. `"200"`, or `"2"` for any 2xx) |
| `timeout_seconds` | no | 1–60 (default 10) |
| `skip_on_error` | no | On a failed poll: `true` (default) keeps the previous result; `false` makes the condition false. A response that arrives but doesn't match is simply false, not an error |

`id` is required on api loops.

## `ramp` value

In place of a number: `{ "ramp": { "from": 0, "to": 180 } }`

| Field | Required | Meaning |
|---|---|---|
| `from` | yes | Value when the enclosing loop's window opens |
| `to` | yes | Value when it closes |

Runs across the enclosing window (once per occurrence inside a `repeat`).
Holds at `from` if there is no window or the window wraps midnight. Not
allowed on `target_watts`/`target_ppfd`.

## `oscillate` value

In place of a number:
`{ "oscillate": { "from": 20, "to": 200, "period": "00:05:00" } }`

| Field | Required | Meaning |
|---|---|---|
| `from`, `to` | yes | The two ends of the cycle (starts at `from`) |
| `period` | yes | One full there-and-back cycle; use 4 seconds or more |
| `phase` | no | Offset of the cycle, for staggering channels |
| `shape` | no | `"sine"` (default) or `"triangle"` |

Measured from local midnight, so the pattern repeats identically each day. Not
allowed on `target_watts`/`target_ppfd`.

## File facts

- Path: `/program.json` in the root of the SD card; read on insert and at
  power-on. The device keeps an internal working copy.
- A refused file (bad JSON, or too large — loading costs ~6× the file size in
  working memory) changes nothing: the previous program keeps running.
- Problems are reported in `PROGRAM_ERROR.TXT` on the card, on the console,
  and in `/logs/errors`.
- Evaluation is once per second; start times are inclusive, end times and end
  dates exclusive, throughout.
