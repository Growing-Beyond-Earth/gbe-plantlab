# Sample Programs

Copy any of these onto your SD card, rename it `program.json`, and adapt it.
Each one is a complete, working program. New to this? Start with
[Programming Your PlantLab](../docs/PROGRAMMING_YOUR_PLANTLAB.md).

| File | What it does |
|---|---|
| `classroom_starter.json` | The standard day: lights 07:00–19:00 at 25 watts, fan around the clock. The best starting point |
| `sunrise.json` | Lights fade up in the morning and down in the evening instead of switching |
| `growth_phases_relative.json` | Different light recipes for seedling, growth, and flowering phases, counted from the day you load the program |
| `alternating_fortnight.json` | Two weeks of high light, two weeks of baseline, repeating — an A/B experiment |
| `ppfd_mix.json` | Targets a light *dose* (PPFD) instead of watts — for units that report Est PPFD |
| `breathing_pulse.json` | A slow "breathing" light effect — fun for demonstrations |
| `watering_every_3_days.json` | Waters every third day, four 8-minute pulses through the day |
| `auto_water_hold.json` | Waters only when the soil moisture sensor says the soil is dry — a measured 3-minute pour |

**About the watering samples:** they switch the PlantLab's **aux output**,
which powers whatever you connect to it. PlantLab kits don't include a
watering pump — to water automatically you'll need your own 12 V pump (a
small peristaltic pump works well) connected to the aux port.
