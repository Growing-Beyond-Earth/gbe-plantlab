# What's New in Your PlantLab

Your PlantLab has received a major upgrade. We rewrote all the code that runs the PlantLab, this time in C instead of Python, to make everything more stable, secure, and simple.

This note covers what's new and the few things that look different, so nothing takes you by surprise.

## The phone app now works with your PlantLab

The biggest change: your PlantLab now talks to the GBE Connect app over Bluetooth. Stand near the device, open the app, and you can:

- see live sensor readings,
- change the light, fan, and schedule without touching the SD card,
- set the device's name and location,
- enter or change WiFi settings,
- install software updates.

The app finds nearby PlantLabs, and syncs the time and sensor data automatically when you connect.

## Your WiFi settings moved — don't panic

If you look on the SD card and the old `wifi_settings.json` file is **gone, that's a new security measure, and your WiFi still works.** During the upgrade, the PlantLab copied your WiFi name and password into its internal memory, and then securely erased the file from the card. The WiFi password no longer sits in plain text on a removable card.

To change WiFi later, use the app.

## No data lost when WiFi drops

The new software sends all its data to the cloud, even when WiFi comes and goes. Whenever the PlantLab can't reach the internet, it keeps saving its sensor readings to the SD card. The moment a connection comes back, it gradually uploads everything it saved, and your online graphs fill in automatically.

## No WiFi at all? Your phone is the messenger

If your classroom has no usable WiFi, the app can carry the data instead. While you're connected over Bluetooth, the PlantLab hands its saved sensor readings to your phone, and your phone passes them along to the cloud using its own internet connection. This happens automatically, but it can take a few seconds to several minutes, depending on how much time has elapsed since your last check-in.

If you do not have WiFi, **you should check in with the app at least a few times per week**. Each visit:

- uploads the readings saved since last time (the app shows progress — give it a few minutes if it's been a while),
- keeps your online graphs current,
- notifies you of any waiting firmware updates.

## Firmware updates arrive on their own

New PlantLab firmware now arrives over the air, over WiFi automatically, or through the app during a check-in. Updates are secure (the PlantLab installs only genuine GBE software), and the device tests each update before committing: if anything goes wrong, it switches back to the previous version by itself. Your settings, program, and data all survive updates.

## The log file on the SD card looks a little different

- Logs are now **one file per month** — `log_2026-09.csv` instead of one ever-growing file.
- The columns are simpler: date, time, the sensor readings, power, fan speed, and estimated light level.
- If you see a file ending in `_old.csv`, that's your previous log, renamed and kept — not deleted — when the format changed.
- A new **Est PPFD** column reports the light reaching your plants in the units scientists use (µmol/m2/s). This is not measured by the sensor, but is estimated based on your light settings and power usage. Some PlantLab units are not yet able to report Est PPFD.

## Friendlier error messages

If a `program.json` has a mistake, the PlantLab no longer leaves you guessing. It writes a plain-language note called `PROGRAM_ERROR.TXT` to the SD card saying what's wrong and what to do — and it keeps running the previous program in the meantime, so your plants are never left in the dark by a typo. The file deletes itself once a good program loads.

## Small things you might notice

- **The clock sets itself** from the internet when WiFi is available, or from the app during a check-in.
- **The PlantLab runs without an SD card.** It keeps its own internal copy of the program, so pulling the card doesn't stop the schedule. (You'll want the card in for data logging.)
- **Programs behave the same.** Anything that ran on the old software runs identically now, including the same light-safety limits.
