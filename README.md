# FTDI-based UART and JTAG Programmer for FPGAs
This JTAG programmer board is based on an FTDI chip and supports Lattice, AMD/Xilinx and Altera/Intel FPGAs. It allows those devices to be programmed and debugged over USB through the regular toolchains.

The board also provides a UART port for general-purpose serial communication between a PC application and any electronics project.

## Features
- Based on the FT2232H chip
- 1 JTAG port
  - Programs and debugs Lattice, AMD/Xilinx and Altera/Intel FPGAs through their respective toolchains
  - Standard 2 mm Xilinx 2x7 header connector
  - Also has a 6-pin 2.54 mm header for use with jumper wires
  - Voltage reference input pin configures the I/O voltage (2.5-5V)
- 1 UART port
  - Generic serial communication through virtual COM port
  - 6-pin 2.54 mm header, pinout compatible with FTDI's TTL-232R cables
  - Configurable 5V output or voltage reference input (*)
- Buffered I/O pins with surge protection
  - Outputs are current-limited
  - Inputs are 5V tolerant
- For Xilinx/AMD and Lattice, just the standard FTDI software drivers are needed
 
(*) These are mounting variations, so to be defined during PCB assembly

## Build result

## How to use
### First-time configuration
Install FTDI's VCP/D2XX drivers for your operating system [from their website](https://ftdichip.com/drivers/).

Also install [FT_PROG](https://ftdichip.com/utilities/#ft_prog). This utility will be used to initialize the EEPROM of the board with the correct settings.

When this is installed, connect the board to your PC. At this point, the green PWREN LED should be on: this means the PC has enumerated the device. If this is not the case, see the [Troubleshooting Guide](TROUBLESHOOTING.md).

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

>[!NOTE]
>The device description and vendor fields should only be customized if you intend to use it with Lattice software. For the other FPGA vendors, the software is more picky and doesn't recognize the programmer if you modify the fields after running the configuration utilities described below.

### Lattice
To use the programmer with Lattice devices, just open the programming software such as Diamond Programmer and select the 'HW-USBN-2B (FTDI)' option as the cable type.

No additional configuration is needed.

![image](https://github.com/user-attachments/assets/191b6716-d51f-4016-885b-d86d8948f95c)

>[!NOTE]
>This programmer is known to work with Diamond Programmer software - other tools, such as Radiant Programmer, have not been tested yet.


### AMD/Xilinx Vivado
Recent Vivado versions contain the `program_ftdi` tool that configures FTDI chips as AMD/Xilinx JTAG programmers. Running this utility (once) is required to make the programmer work with Vivado. You can execute it from the TCL console, e.g. in the Hardware Manager of Vivado. Run a command similar to this one:
```
program_ftdi -write -ftdi FT2232H -serial 0ABC01 -vendor "my vendor co" -board "my board" -desc "my product desc"
```
Note that the 'serial' parameter must match the serial number generated by FT_PROG. After programming, the programmer should show up in the Hardware Manager of Vivado.


>[!WARNING]
>Unfortunately, it seems that the `program_ftdi` is broken in Vivado 2024.1. If you see it complaining about missing DLLs while trying to program, try using an older version of Vivado, e.g. 2023.2, which is known to work. Alternatively, check out [dragonlock2's ftdi_dumps repository](https://github.com/dragonlock2/ftdi_dumps?tab=readme-ov-file) with EEPROM dumps of FTDI chips used in commercial products and program that one directly. Writing the EEPROM configuration can be done using the (Linux) utility `ftdi_eeprom` or my own FTDI EEPROM dumping tool.

>[!TIP]
>After running the Xilinx utility `program_ftdi`, you can still use the device for Lattice FPGAs (though it does not work simultaneously for Altera FPGAs...).

![image](https://github.com/user-attachments/assets/e66b6b8b-50de-415a-9c8b-a43718145bcb)

### Xilinx ISE
To use this programmer with Xilinx' older ISE toolchain, you'll need to flash the EEPROM to mimick a legacy Digilent SMT1 device (or any supported FTDI programmer based on a FT2232H chip). You will find the required EEPROM dump on [dragonlock2's ftdi_dumps repository](https://github.com/dragonlock2/ftdi_dumps?tab=readme-ov-file).

>[!TIP]
>Writing the EEPROM configuration can be done using the (Linux) utility `ftdi_eeprom` or my own FTDI EEPROM dumping tool.

Once the EEPROM is programmed, the cable appears as a Digilent USB JTAG cable:

![image](https://github.com/user-attachments/assets/c2d133c6-aa66-4827-b3aa-648a451eb14f)

>[!TIP]
>Using the Digilent SMT1 dump is the most versatile solution: the programmer works simultaneously for Xilinx ISE, Vivado and Lattice Diamond.

### Altera/Intel
To use the FTDI programmer with Quartus software, you need to configure the device as an 'Arrow USB Blaster'. You will find the required EEPROM dump on [dragonlock2's ftdi_dumps repository](https://github.com/dragonlock2/ftdi_dumps?tab=readme-ov-file).

>[!TIP]
>Writing the EEPROM configuration can be done using the (Linux) utility `ftdi_eeprom` or my own FTDI EEPROM dumping tool.

You also need to install the [Arrow USB Blaster driver](https://shop.trenz-electronic.de/en/Download/?path=Trenz_Electronic/Software/Drivers/Arrow_USB_Programmer). After this, your FTDI programmer should appear in the hardware list of Quartus Programmer and your FPGA is ready to be programmed.

![image](https://github.com/user-attachments/assets/fc0742db-86ff-43d0-8f19-78ab4b849e13)

## Troubleshooting
If you experience problems while using this board, check out the [Troubleshooting Guide](TROUBLESHOOTING.md).
