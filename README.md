# FTDI-based JTAG Programmer for FPGAs
This JTAG programmer board is based on an FTDI chip and supports Lattice, AMD (Xilinx), Intel (Altera) FPGAs, and probably more. It allows those devices to be programmed and debugged over USB through their regular toolchains.

The board also provides a separate UART port for general-purpose serial communication between a PC application and any other board.

If you work with a lot of different FPGAs, this solution can be used as a universal, cost-effective replacement for most of your vendor-specific JTAG cables.

## Features
- Based on the FT2232H chip
- 1 JTAG port
  - Programs and debugs Lattice, AMD/Xilinx, Altera/Intel, and probably also other FPGAs through their respective toolchains
  - Standard 2 mm Xilinx 2x7 header connector
  - Also has a 6-pin 2.54 mm header for use with jumper wires
  - Voltage reference input pin configures the I/O voltage (2.5-5V)
- 1 UART port
  - Generic serial communication through virtual COM port
  - 6-pin 2.54 mm header, pinout compatible with FTDI's TTL-232R cables
  - Configurable 5V output or voltage reference input (2.5-5V) (*)
- Buffered I/O pins with surge protection
  - Outputs are current-limited
  - Inputs are 5V tolerant
- For Xilinx/AMD and Lattice, only the standard FTDI software drivers are needed
 
(*) These are mounting variations, so to be decided during PCB assembly.

## Build overview
<img src="https://github.com/user-attachments/assets/7395aee2-f5fe-4fce-978e-9c705c561ec0" width="400" />

<img src="https://github.com/user-attachments/assets/c2324eec-d306-41d3-bb89-3800ac6dc31b" width="400" />

This board was designed using KiCad 7.0.2.

Navigate to the releases section to obtain the latest production files and schematic.

Files for 3D printing an optional enclosure are also provided. The enclosure is recommended to prevent accidental shorts during use.

<img src="https://github.com/user-attachments/assets/0eb81f71-52bd-4e76-bff5-49464992ebff" width="500" />


## How to use
### First-time configuration
Install FTDI's VCP/D2XX drivers for your operating system [from their website](https://ftdichip.com/drivers/).

Also install [FT_PROG](https://ftdichip.com/utilities/#ft_prog). This utility will be used to initialize the EEPROM of the board with the correct settings.

Now you can choose to either initialize the EEPROM using FT_PROG or directly flash the device using an existing EEPROM dump.

>[!TIP]
>The most versatile solution is to flash the EEPROM with the Digilent SMT1 dump. The programmer then works simultaneously for Vivado, ISE, Diamond and Quartus (details in the [Xilinx ISE section](#xilinx-ise)).
>
>However, for the very first startup of your board, please try to write the following settings using the FT_PROG utility. This way, you can make sure that your hardware and drivers are working properly.

When FT_PROG is installed, connect the board to your PC. At this point, the green PWREN LED should be on: this means the PC has enumerated the device. If this is not the case, see the [Troubleshooting Guide](TROUBLESHOOTING.md).

In FT_PROG, scan and parse the connected devices. Configure your board as follows:
- Port A
  - Set the hardware mode to "245 FIFO"
  - Set the driver to D2XX Direct
- Port B
  - Set the hardware mode to "RS232 UART"
  - Set the driver to Virtual COM Port
 
<img src="https://github.com/user-attachments/assets/a16dd61f-4555-4021-bcf2-6a7931f90843" width="400">
<img src="https://github.com/user-attachments/assets/0578e1ab-40e9-4639-bb36-360b7a9544a5" width="404">

Finally, program this configuration into the EEPROM. Depending on the vendor of the FPGA you want to program, a few additional steps might be needed. These are described below.

### Lattice
To use the programmer with Lattice devices, just open the programming software such as Diamond Programmer and select the 'HW-USBN-2B (FTDI)' option as the cable type.

Diamond programmer can use any FTDI device configured in D2XX mode. No additional configuration is needed.

![image](https://github.com/user-attachments/assets/191b6716-d51f-4016-885b-d86d8948f95c)

>[!NOTE]
>This programmer is known to work with Diamond Programmer software - other tools, such as Radiant Programmer, have not been tested yet.


### AMD/Xilinx Vivado
For AMD/Xilinx tools, the software looks for a Xilinx signature in the FTDI device EEPROM. You have two options to add this: either write a recognized EEPROM dump to the device (recommended), or use AMD's `program_ftdi` utility.

The recommended way is to use the Digilent SMT1 dump; refer to the [Xilinx ISE section](#xilinx-ise) on how to do this.

If you really want to do it AMD's way: recent Vivado versions (starting from 2022) contain the `program_ftdi` tool that configures FTDI chips as AMD/Xilinx JTAG programmers. Running this utility (once) will make the programmer work with Vivado. You can execute it from the TCL console, e.g. in the Hardware Manager of Vivado. Run a command similar to this one:
```
program_ftdi -write -ftdi FT2232H -serial 0ABC01 -vendor "my vendor co" -board "my board" -desc "my product desc"
```
Note that the 'serial' parameter must match the serial number generated by FT_PROG. After programming, the programmer should show up in the Hardware Manager of Vivado.

![image](https://github.com/user-attachments/assets/e66b6b8b-50de-415a-9c8b-a43718145bcb)

>[!WARNING]
>Unfortunately, it seems that the `program_ftdi` is broken in Vivado 2024.1. If you see it complaining about missing DLLs while trying to program, try using an older version of Vivado, e.g. 2023.2, which is known to work.


### Xilinx ISE
To use this programmer with Xilinx' older ISE toolchain, you'll need to flash the EEPROM to mimick a legacy Digilent SMT1 device. You will find the required EEPROM dump on [dragonlock2's ftdi_dumps repository](https://github.com/dragonlock2/ftdi_dumps?tab=readme-ov-file).

Note that this dump has the advantage that you can make the programmer work simultaneously for Vivado, ISE, Diamond and Quartus, turning your programmer in an (at least) 4-in-1 solution!

>[!TIP]
>Writing the EEPROM configuration can be done using the (Linux) utility `ftdi_eeprom` or my own [FTDI EEPROM cloning tool](https://github.com/anthony-bernaert/ftdi-cloner).

Once the EEPROM is programmed, the cable appears as a Digilent USB JTAG cable:

![image](https://github.com/user-attachments/assets/c2d133c6-aa66-4827-b3aa-648a451eb14f)

### Altera/Intel
For Quartus software to work with FT2232HL devices, one option is to add a library file to your Quartus installation directory. You'll find a ready-made DLL (if you're on Windows) [here](https://github.com/anthony-bernaert/jtag-mpsse-blaster). This is a fork of an open-source solution that adds FTDI MPSSE support to Quartus Programmer/Signal Tap. Linux is also supported, though I did not test this.

After installing the .dll/.so file, the JTAG programmer board should show up in the available hardware list of Quartus and your FPGA is ready to be programmed:

<img src="https://github.com/user-attachments/assets/405c9fbc-ca07-49d9-83ff-b391de7fe68e" width="700" />

If you don't like modifying your Quartus installation manually, an alternative solution is to configure the USB device as an 'Arrow USB Blaster'. You will find the required EEPROM dump on [dragonlock2's ftdi_dumps repository](https://github.com/dragonlock2/ftdi_dumps?tab=readme-ov-file).

>[!TIP]
>Writing the EEPROM configuration can be done using the (Linux) utility `ftdi_eeprom` or my own [FTDI EEPROM cloning tool](https://github.com/anthony-bernaert/ftdi-cloner).

In this case, you also need to install the [Arrow USB Blaster driver](https://shop.trenz-electronic.de/en/Download/?path=Trenz_Electronic/Software/Drivers/Arrow_USB_Programmer). After this, your FTDI programmer should appear in the hardware list of Quartus Programmer and your FPGA is ready to be programmed.

![image](https://github.com/user-attachments/assets/fc0742db-86ff-43d0-8f19-78ab4b849e13)

## Troubleshooting
If you experience problems while using this board, check out the [Troubleshooting Guide](TROUBLESHOOTING.md).
