# Troubleshooting Guide
## Hardware issues
In case your board doesn't start up right away, or does not enumerate over USB, be sure to check following things:
### Solder joints
  - Inspect all pins of FTDI chip, are they properly soldered without shorts in-between? The pitch is rather small, so be sure to double-check this first.
  - Inspect the USB connector, especially the ground pin: this directly connects to a ground plane which has a higher thermal mass and it takes longer to properly solder this pin.
  - Inspect the crystal: if soldered by hand, this component might be a bit tricky to solder properly, as the pads are (mostly) underneath the device. If you have it available, reseat it using a heat gun.
### Check the power
Check there is 5V (on TP14), 3.3V (on J5, pin 2), VPHY 3.3V (TP16), VPLL 3.3V (TP15), 2.5V (on J5, pin 3) and 1.8V (on TP2).
  - It's good to check the current draw of the board: it should be well under 100 mA. If it's higher, something is definitely wrong!
  - If there is no 5V, the issue might be the USB connector soldering, the input filter (ferrite bead) or the USB cable. Before trying (to fry) another USB port, it may be wise to apply 5V from a lab supply instead and verify current draw first.
  - If there is no 3.3V or 2.5V: check regulators U1 and U8.
  - If there is no VPHY or VPLL, check the ferrite filter between 3.3V and the VPHY/VPLL rail.
  - If there is no 1.8V: check the FTDI chip itself, does it get hot, does it have shorts on its pins?
### Crystal behavior
If you have an oscilloscope, measure the OSCI/OSCO signals.
  - Do you see the crystal starting up? If yes, is it a stable wave (a centered sine-like wave on OSCI, and a more square-like wave on OSCO)? Does it have the right frequency (12 MHz)? It might stop running abruptly (after like 10 ms) if there was an enumeration failure, so be sure to also measure when plugging in the device.
  - If it is unstable, try tuning the load capacitors (check the datasheet of the crystal). FTDI suggests to use 27 pF capacitors as a default, but recommends verifying the crystal manufacturer's specification.

## Software issues
If the board starts up correctly (PWREN LED on) and is recognized by your operating system, driver issues might still throw a spanner in the works.
### FT_PROG doesn't see any devices
  - An issue I encountered was that FT_PROG couldn't load the FTD2XX.dll driver, but did not show any error message at first glance. However, you can see the issue when you go to the command line. 
If you see errors like
```Failed to load FTD2XX.DLL. Are the FTDI drivers installed?```
 it might indicate that the drivers are not correctly installed.

![image](https://github.com/user-attachments/assets/3cb0e7bc-4c6b-4b76-bfcc-057fb22ccd2d)

  - Comparing the loaded driver files on a PC where everything works fine with one where FT_PROG does not work, the difference was the the SysWOW64\ftd2xx.dll. This line was absent on the failing PC:
![image](https://github.com/user-attachments/assets/85e6d831-0830-48f8-b945-0272bcba0667)
  - While not ideal, the only working solution I found until now is to manually delete the drivers in the above list, and run the FTDI CDM driver setup again.
  - Other people have resolved similar issues by using another USB cable. Be sure to use a quality cable that is not only intended for charging!
### Vivado cannot run `program_ftdi`, it either segfaults, complains about missing DLLs or cannot find the command
  - Only Vivado 2022 and newer have the utility included
  - It seems things are broken in Vivado 2024; try using a lower version such as Vivado 2023.2
  - Alternatively, use the Linux tool `ftdi_eeprom` or my own FTDI EEPROM dumper tool to configure an FTDI device with a Xilinx 'fingerprint'.

### USB driver conflicts
If you used ESP32 development boards in the past, you might have used a tool like Zadig to replace USB drivers for FT2232H devices. If you have this installed, make sure that the correct (FTDI) driver is not automatically replaced for the JTAG programmer board, as can be seen here:

![thumbnail_image](https://github.com/user-attachments/assets/4f74fd7b-d4f5-47e7-9d5e-4f84b8ec60e5)

