# Advanced PlantLab Programming

This guide continues where
[Programming Your PlantLab](PROGRAMMING_YOUR_PLANTLAB.md) leaves off: the aux
output, repeating schedules, sensor-driven behavior, multi-day experiments,
smoothly changing light, and how loops combine. Every field is listed in the
[PlantLab Programming Dictionary](PLANTLAB_PROGRAMMING_DICTIONARY.md).

## How loops combine

The PlantLab re-evaluates the whole program once per second:

1. It starts from `default_actions` if the program has one (see below), or
   from everything-off if not.
2. It goes through the loops **in order**. Every loop whose condition is true
   applies its actions.
3. **The last matching loop wins** for any setting they both touch.

So put general rules first and exceptions after them: a "fan at 96 all day"
loop followed by a "fan at 200 when hot" loop does what you'd expect; in the
other order the all-day loop would immediately overwrite the override.

### Nesting = AND

A loop can carry child loops in a `loops` field. Children are only considered
while the parent is active — that's how you say "during these hours AND this
condition":

```json
{
  "type": "time",
  "id": "day_window",
  "start": "09:00",
  "end": "17:00",
  "actions": { "fan": 96 },
  "loops": [
    {
      "type": "sensor",
      "id": "day_heat_boost",
      "condition": { "sensor": "temperature", "comparison": ">", "value": 28 },
      "actions": { "fan": 200 }
    }
  ]
}
```

Children override their parent (last match wins applies here too). Nesting
works up to 6 levels deep; deeper loops are never evaluated and the PlantLab
warns about them.

### `default_actions`

`default_actions` is the baseline applied when no loop says otherwise. Current
firmware doesn't require it — everything defaults to off — but it's a tidy way
to express "fan always, unless something changes it":

```json
{
  "default_actions": { "fan": 96 },
  "loops": []
}
```

You may also see older programs with everything wrapped in a `"settings"`
object, or with `actions` written as a one-element array. Both still work;
new programs are simpler without them.

## The aux output

`aux` (0–255) switches the auxiliary port — relays, solenoid valves, and other
add-on devices. Most aux devices are effectively on/off, so `255` is the usual
"on". The PlantLab keeps a running total of aux on-time (its odometer), which
is how watering amounts get recorded.

```json
{
  "type": "time",
  "id": "afternoon_watering",
  "start": "14:00",
  "duration": "00:05:00",
  "actions": { "aux": 255 }
}
```

That's `duration` doing something new: with a `start` but no `end`, the loop
runs for exactly that long — five minutes at 14:00, **every day**. A
`duration` with no `start` at all runs once, from the moment the program
loads, and never again (that's what the app's temporary-control feature uses).

## Repeating through the day: `repeat`

One `repeat` loop replaces a whole list of time loops:

```json
{
  "type": "repeat",
  "id": "hourly_mist",
  "start": "08:00",
  "interval": "02:00",
  "duration": "00:00:30",
  "until": "18:00",
  "actions": { "aux": 255 }
}
```

Aux on for 30 seconds at 08:00, 10:00, 12:00, 14:00, and 16:00. `interval` is
the gap between the *starts* of occurrences; `until` cuts the series off (no
occurrence starts at or after it). Keep `duration` at 2 seconds or more — the
program is evaluated once per second, and a shorter pulse can be missed.

Prefer this over writing one loop per occurrence: it's shorter, it can't
drift out of step, and it uses a tiny fraction of the memory.

## Reacting to sensors: `sensor`

```json
{
  "type": "sensor",
  "id": "heat_alarm",
  "condition": { "sensor": "temperature", "comparison": ">", "value": 30 },
  "actions": { "fan": 255 }
}
```

Available sensors: `temperature` (°C), `humidity` (%), `co2` (ppm),
`pressure` (hPa), `light` (lux), `soil_moisture` (%), `power` (W),
`voltage` (V), `current` (A), `fan_rpm` (RPM). Comparisons: `>`, `>=`, `<`,
`<=`, `==`, `!=`.

Conditions are re-checked every `check_interval` (default: one minute).
Sensors themselves are read every 10 seconds, so intervals shorter than a
minute gain nothing.

### Stop it flickering: `hysteresis`

A reading sitting right at the threshold turns things on and off every check.
`hysteresis` makes the condition sticky:

```json
{
  "type": "sensor",
  "id": "cooling",
  "condition": { "sensor": "temperature", "comparison": ">", "value": 28, "hysteresis": 1 },
  "actions": { "fan": 200 }
}
```

The fan comes on above 28 °C and stays on until the temperature falls below
27 °C. The turn-on point is always exactly the `value` you wrote; the band only
applies on the way back out.

### A measured pulse: `hold`

For watering, on-time *is* the amount delivered, so a pour should never be cut
short — even though watering itself pushes the moisture reading back up:

```json
{
  "type": "sensor",
  "id": "auto_water",
  "check_interval": "00:30:00",
  "condition": { "sensor": "soil_moisture", "comparison": "<", "value": 88, "hold": "00:03:00" },
  "actions": { "aux": 255 }
}
```

"Every half hour, if the soil is below 88, water for exactly three minutes."
Each check that finds the condition true starts one full pulse; nothing
interrupts a pulse in progress. `hold` must be shorter than `check_interval`,
and note the interval defaults to one minute — a three-minute hold needs the
interval set explicitly, as above.

One structural rule worth internalizing: **a sensor loop's own actions run for
as long as its condition holds** — they are not limited by anything nested
inside it. For a timed response, put the timing on the condition (`hold`) or
in a child loop, and let the sensor loop be a pure gate.

## Multi-day scheduling

Both loop types below work on calendar days, and neither runs until the
PlantLab's clock has been set (a cadence on a wrong clock would silently land
on the wrong days). Dates are written `"YYYY-MM-DD"`.

### A recurring cadence: `repeat_days`

```json
{
  "type": "repeat_days",
  "id": "watering_days",
  "start_date": "2026-09-01",
  "interval_days": 3,
  "end_date": "2026-12-15",
  "loops": [
    {
      "type": "repeat",
      "id": "watering_pulses",
      "start": "08:00",
      "interval": "04:00",
      "duration": "00:08:00",
      "until": "20:01",
      "actions": { "aux": 255 }
    }
  ]
}
```

Fires on September 1, 4, 7, … and on each firing day waters at 08:00, 12:00,
16:00, and 20:00. (Note `until` is 20:01: no occurrence starts *at or after*
`until`, so 20:00 exactly would exclude the last pulse.) `duration_days` makes each occurrence span several days —
`"interval_days": 28, "duration_days": 14` is two weeks on, two weeks off.
`end_date` is **exclusive**: nothing fires on or after that date.

Omit `start_date` and the cadence counts from the day the loop is first seen —
"every 3 days, starting now" needs no date at all. A loop doing that **must
have an `id`**, because the id is what the start day is remembered against.

A day loop selects *days*, not hours — putting `actions` directly on it
applies them midnight to midnight. Usually you want hours, so nest a `time` or
`repeat` loop inside, as above.

### One continuous span: `date_range`

```json
{
  "type": "date_range",
  "id": "germination",
  "duration_days": 14,
  "loops": [
    {
      "type": "time",
      "start": "06:00",
      "end": "22:00",
      "actions": { "white": 150, "target_watts": 20 }
    }
  ]
}
```

A `date_range` is a stretch of days — pin it with `start_date`/`end_date`, or
let it start from "now" with `duration_days` (length) and `after_days` (delay
before it begins). `after_days` is how you chain phases: seedling recipe for 14
days, then a `date_range` with `"after_days": 14` takes over. Like
`repeat_days`, a range counted from "now" needs an `id`.

### Anchors: how "starting now" is remembered

The first day a from-"now" loop is seen, the PlantLab stamps that date against
the loop's `id` and stores it. The anchor survives reboots, power cuts, and
re-loading the same program — your experiment doesn't restart because the
power blinked. Two consequences:

- **Set the clock before loading the program**, or the anchor stamps against
  whatever day the clock showed.
- **Renaming an `id` creates a new loop** with a fresh anchor. That's the
  deliberate way to restart a schedule — and a reason not to rename ids
  casually. The same applies to `duration` timing on time loops: the `id` is
  the loop's identity, not just a label.

## Light that changes smoothly

Any action value can be time-varying instead of a fixed number.

### `ramp` — sunrise and sunset

```json
{
  "type": "time",
  "id": "sunrise",
  "start": "06:00",
  "end": "08:00",
  "actions": { "white": { "ramp": { "from": 0, "to": 180 } } }
}
```

White climbs steadily from 0 at 06:00 to 180 at 08:00. A ramp runs across its
loop's window, so inside a `repeat` it runs once per occurrence. It needs a
window: inside a sensor loop or a midnight-wrapping window it just holds at
`from` (the PlantLab warns about this).

### `oscillate` — a slow cycle

```json
{
  "type": "time",
  "id": "light_show",
  "start": "07:00",
  "end": "19:00",
  "actions": {
    "red": { "oscillate": { "from": 20, "to": 200, "period": "00:05:00" } },
    "blue": { "oscillate": { "from": 200, "to": 20, "period": "00:07:00" } },
    "white": 30
  }
}
```

The value swings from `from` to `to` and back over one `period` (use 4 seconds
or more). `phase` offsets a channel's cycle; `shape` is `"sine"` (default) or
`"triangle"`. Different periods on different channels is what makes motion
look organic.

**Never put a ramp or oscillate on `target_watts` or `target_ppfd`** — the
power adjustment is itself a feedback loop, and a moving target keeps it
chasing forever. Targets take plain numbers.

## Checking the internet: `api`

An `api` loop polls a web address with a GET request and matches the response:

```json
{
  "type": "api",
  "id": "iss_overhead",
  "url": "https://api.example.com/iss/visible",
  "check_interval": "00:05",
  "match_value": "true",
  "match_mode": "substring",
  "actions": { "blue": 99 }
}
```

`id`, `url`, `check_interval`, and `match_value` are all required.
`match_mode` is `"substring"` (default), `"exact"`, or `"http_status"`.
`skip_on_error` (default `true`) keeps the previous answer through network
hiccups; set it `false` to treat a failed poll as "no". Requests are GET-only
and need WiFi, and addresses on private networks are blocked.

## Versioning a program: `format_version`

If a program depends on a newer feature that older firmware would *misread*
(rather than just skip), declare the format version so old firmware refuses it
cleanly. The important case is `target_ppfd`, added in format 5: firmware
older than that ignores the field and runs the raw colors at full brightness.
A program using `target_ppfd` should carry:

```json
{
  "format_version": 5,
  "loops": [
    {
      "type": "time",
      "id": "grow_light",
      "start": "07:00",
      "end": "19:00",
      "actions": { "red": 9, "blue": 26, "white": 98, "target_ppfd": 200 }
    }
  ]
}
```

## When something doesn't work

- **The file was refused** (`PROGRAM_ERROR.TXT` says so): it wasn't valid
  JSON, or was too large to load. The previous program keeps running and your
  file is untouched on the card.
- **It loaded with warnings**: it's running, but some parts will never do
  anything — almost always a misspelled field or loop type, which the PlantLab
  skips rather than guesses at. The warnings name each one.
- **Size**: loading costs about six times the file's size in working memory.
  If a program grows past a few kilobytes, it's usually a schedule written out
  step by step — say it once with `repeat`, `ramp`, or `oscillate` instead.
