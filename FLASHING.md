# Installing PlantLab Software with a USB Cable

**You rarely need this.** A PlantLab already running the new software updates
itself over WiFi or through the GBE Connect app. Use this guide only to:

- upgrade a PlantLab from the **original 2024 or 2025 MicroPython software**, or
- **recover** a unit that isn't starting properly.

> **Heads-up before you start:** a USB install is a factory-fresh start. The
> PlantLab will forget its WiFi settings (you'll re-enter them in the app —
> about a minute). Data already on the SD card or already uploaded online is 
> not affected.

## What you need

- The latest software file:
  **[download it here](https://growingbeyond.earth/update.php)** (a file named
  like `gbe-plantlab-1.0.0.uf2`)
- The USB A–to–Micro USB cable included in your kit (any USB cable that
  carries data will work; some charge-only cables won't)
- The USB C–to–USB A adapter from the kit, if your computer only has USB C
  ports
- A small screwdriver or similar pointed tool, to reach the BOOTSEL button
  through the hole in the clear acrylic lid covering the circuit board
- Any computer — Windows, Mac, or Chromebook

## Steps

1. **Unplug the PlantLab's power.**
2. **Connect the USB cable** between your computer and the Micro USB port on
   the Pico board (add the USB C adapter if your computer needs it).
3. Find the two buttons: **BOOTSEL** is the small white button on the Pico
   board — reach it with your pointed tool through the hole in the clear
   acrylic lid. **RESET** is the small black button next to the power jack.
4. **Hold BOOTSEL down, and while holding it, press and release RESET.**
   Keep holding BOOTSEL until a new drive named **RPI-RP2** appears on the
   computer, then let go.
5. **Drag the downloaded `.uf2` file onto the RPI-RP2 drive.** The drive
   disappears on its own when the copy finishes — that's normal, and it means
   it worked.
6. Unplug the USB cable and plug the PlantLab's power back in. You'll see a
   **rainbow pulse** from the lights right away — that's the new software
   starting up. (A magenta pulse a few seconds after power-on means the old
   software is still installed — repeat the steps above.)
7. Open the GBE Connect app nearby to set the WiFi and check that readings
   appear.

## If something doesn't look right

- **No RPI-RP2 drive appears:** you likely have a charge-only cable — try
  another one. Make sure BOOTSEL is held down *before* you press RESET, and
  keep holding it until the drive shows up.
- **The drive reappears after copying:** the copy didn't take — try again,
  and if it repeats, re-download the file.
- **Still stuck:** the install can be repeated as many times as you like; it
  cannot damage the PlantLab. Contact GBE support with the version number
  from your downloaded file.
