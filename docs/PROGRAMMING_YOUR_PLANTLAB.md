# Programming Your PlantLab

Your PlantLab follows a program: a small text file named `program.json` that
tells it what to do at each time of day. This guide covers the basics — lights,
fan, and on/off timing. When you're ready for more (watering on a schedule,
reacting to sensors, multi-day experiments), see
[Advanced PlantLab Programming](ADVANCED_PLANTLAB_PROGRAMMING.md). Every
setting is listed in the
[PlantLab Programming Dictionary](PLANTLAB_PROGRAMMING_DICTIONARY.md).

## How programs get into the PlantLab

Save your file as `program.json` in the top folder of the SD card, then insert
the card. The PlantLab reads it right away — no restart needed. It also keeps
its own copy inside, so the program keeps running even if the card is removed.

If something in the file is wrong, the PlantLab writes a plain-language note
called `PROGRAM_ERROR.TXT` to the card telling you what to fix, and keeps
running its previous program — your plants are never left in the dark by a typo.

## Your first program

This turns the grow light on from 7:00 in the morning to 7:00 in the evening,
and runs the fan gently all day and night:

```json
{
  "loops": [
    {
      "type": "time",
      "id": "grow_light",
      "start": "07:00",
      "end": "19:00",
      "actions": { "red": 9, "blue": 26, "white": 98 }
    },
    {
      "type": "time",
      "id": "fan_always",
      "actions": { "fan": 96 }
    }
  ]
}
```

A program is a list of **loops**. Each loop says *when* (`start` and `end`)
and *what* (`actions`). Anything a loop doesn't mention stays off — with no
loop running, the lights and fan are simply off.

The parts of a loop:

- `"type": "time"` — this loop is controlled by the time of day.
- `"id"` — a name you choose, so you (and the PlantLab) can tell loops apart.
- `"start"` and `"end"` — 24-hour clock times, like `"07:00"` or `"19:30"`.
  Leave them both out and the loop runs all day, like `fan_always` above.
- `"actions"` — what to set while the loop is active.

## How times work

- Times use the 24-hour clock: `"07:00"` is 7 AM, `"19:00"` is 7 PM.
- A loop is on **from** its start time **up to, but not including**, its end
  time. `"end": "19:00"` means the lights go off exactly at 19:00:00.
- A window can cross midnight: `"start": "22:00", "end": "06:00"` is a
  nighttime loop.
- Time not covered by any loop = everything off.

## Lights: colors and brightness

The grow light has four channels: `red`, `green`, `blue`, and `white`. Each
takes a number from 0 (off) to 255 (as bright as that channel goes):

```json
{ "red": 40, "green": 0, "blue": 26, "white": 98 }
```

Each channel also has a built-in safety limit — asking for more than the limit
just gives you the limit:

| Channel | Safe maximum |
|---------|--------------|
| `red`   | 240 |
| `green` | 82 |
| `blue`  | 99 |
| `white` | 196 |

Because the limits differ, equal numbers do **not** mean equal light: green at
82 is at its ceiling, while red at 82 has plenty of headroom left.

## The fan

`fan` takes 0 (off) to 255 (full speed). Plants like moving air — a modest
constant speed such as `96` is a good starting point, and it's what the
standard program uses.

## Setting light by power: `target_watts`

Instead of guessing at brightness numbers, you can tell the PlantLab how much
electrical power the light should use, and it will adjust the brightness
itself:

```json
{
  "type": "time",
  "id": "grow_light",
  "start": "07:00",
  "end": "19:00",
  "actions": { "red": 9, "blue": 26, "white": 98, "target_watts": 25 }
}
```

The color numbers set the **recipe** — the balance of red, blue, and white —
and `target_watts` sets the **amount**. The PlantLab scales the whole recipe up
or down until the light draws 25 watts, and holds it there. This is exactly how
the standard PlantLab program works. (Targets below 2 watts are ignored and
the color numbers run as written.)

## Setting light by what plants receive: `target_ppfd`

Scientists measure grow light in **PPFD** — how many photons land on the
plants each second (µmol/m²/s). Two PlantLabs using the same watts can deliver
slightly different PPFD, so this is the more scientific target:

```json
{ "red": 9, "blue": 26, "white": 98, "target_ppfd": 200 }
```

As with watts, the colors set the recipe and `target_ppfd` sets the amount.

**This only works on a PlantLab that is reporting PPFD.** Most are — check the
`Est PPFD` column in the log file on the SD card, or your readings online. If
that column is blank, your unit isn't estimating light, `target_ppfd` will be
ignored, and the color numbers will run exactly as written — which can be much
brighter than you intended. When in doubt, use `target_watts`.

## Rules of the file (JSON)

`program.json` is written in a format called JSON. Four rules cover almost
every error:

1. Every name and every time goes in **double quotes**: `"start": "07:00"`.
   Numbers do not: `"fan": 96`.
2. Put a **comma between** items, but **not after the last one**.
3. Every `{` needs a matching `}`, and every `[` a matching `]`.
4. No comments — JSON has no way to write notes to yourself. Use the `id`
   fields to label things instead.

If the PlantLab refuses your file, `PROGRAM_ERROR.TXT` on the card points at
the problem, and a free online "JSON validator" will point at the same spot.

## A complete program to start from

Lights 12 hours a day at 25 watts, fan around the clock:

```json
{
  "program_name": "Classroom starter",
  "loops": [
    {
      "type": "time",
      "id": "grow_light",
      "start": "07:00",
      "end": "19:00",
      "actions": { "red": 9, "blue": 26, "white": 98, "target_watts": 25 }
    },
    {
      "type": "time",
      "id": "fan_always",
      "actions": { "fan": 96 }
    }
  ]
}
```

Copy it, change the times and the recipe, and you're programming your PlantLab.
