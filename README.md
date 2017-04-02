﻿# rdos
An Arduino-based “hard drive” for the Tandy 100/102

This is based off my failed project “Arduino-tpdd”.

The projects would not work with the pre-existing clients (TS-DOS or Teeny).  But it would work with the small BASIC test programs that I created.

So I decided to write my own client(s) to communicate with my project.  This would not offer the full functionality of TS-DOS, but it will allow someone to "boot" the system from the Arduino, save and load files, and, maybe, do some renaming, deleting.

# Hardware explanation

Our base is a microcontroller.  I needed more memory than the Arduino Uno, so I went with the Arduino Mega to get enough SRAM.  The SD shield plus SoftwareSerial libraries just wouldn't fit in the Uno's small memory.

The microSD shield was a no-brainer.  The only thing with that is that I had to solder the SPI headers on since the microSD shield was made for the Uno and the usual pin outs were not compatible with the Mega.  The SPI header, though, lined up. [MicroSD Shield](https://www.sparkfun.com/products/12761)

The biggest pain was the RS-232 shifter.  RS-232 operates from 3.3V to 12V.  So we need something to 1. shift DOWN the voltage (because sending more than 5V down the Arduino's TTL lines would fry it) and 2. to shift UP the TTL voltage to something that my T102 would like to see.  After a few failures, I went with the [RS232 to 5V TTL Converter](http://www.serialcomm.com/serial_rs232_converters/rs232_rs485_to_ttl_converters/rs232_to_5v_ttl_converter/rs232_to_5v_ttl.product_general_info.aspx)


Arduino pins used:
* 7 - Drive activity light
* Digital ground - Drive activity light (gnd)
* 62 (A8) - Shifter TX
* 63 (A9) - Shifter RX
* Analog 5V and ground for the power for the shifter.
* SPI header for SD shield
* TBA button to "boot" RLOAD to the Tandy using LOAD "COM:98N1D

Unavailable pins:
* 10-13
* 50-53
* 8 (card select)

Notes:
The Mega puts the SPI pins on 50-53 instead of 10-13.  So you need to solder the SPI header to the MicroSD Shield in order to move the pins.  As a result, pins 50-53 on the Mega are unavailable.  Also, because of how the MicroSD Shield is made, pins 10-13 are unavailable as well - if you use the stacking headers.  You can probably get around this by not using the stacking headers for pins D8-D13 and connecting stuff directly to the Mega and not through the MicroSD Shield.

Not all pins on the Mega and Mega 2560 support change interrupts, so only the following can be used for RX:
10, 11, 12, 13, 50, 51, 52, 53, 62, 63, 64, 65, 66, 67, 68, 69
So, since 10-13 and 50-53 are unavailable, I had to put the shifter on pins 62 and 63.

# Software explanation

Much of the code is based on a [DeskLink port to Linux](http://www.bitchin100.com/).
The protocol is (reverse engineered) documented [here](http://bitchin100.com/wiki/index.php?title=TPDD_Base_Protocol)

Commands:
+ Type 00 - Directory Reference - Used by RLIST to list the files, and RSAVE/RLOAD to save and load files.  Option "request previous directory block", and "end directory reference" will not be supported
+ Type 01 - Open file - Used by RSAVE/RLOAD to save and load files
+ Type 02 - Close file - Used by RSAVE/RLOAD to save and load files
+ Type 03 - Read file - Used by RLOAD to load files
+ Type 04 - Write file - Used by RSAVE to save files
+ Type 05 - Delete file - Used by RMAN to delete files
+ Type 06 - Format Disk - Unsupported
+ Type 07 - Drive Status - Unsupported
+ Type 08 - DME Req - Unsupported
+ Type 0C - Drive Condition - Unsupported
+ Type 0D - Rename file - Used by RMAN to delete files
+ Type 23 - TS-DOS Mystery Command - Unsupported
+ Type 31 - TS-DOS Mystery Command - Unsupported

File names were padded, internally, to be 6 bytes, dot, 2 bytes.  This will be removed and will use the full 24 bytes available.
File name case will NOT be forced to upper case.
Directories will be supported.
Checksum will not be supported (since the communications is much better).

+ RLIST.BA - List the files in the current directory
+ RLOAD.BA - Load the file into the Tandy
+ RSAVE.BA - Save the file into the SD card
+ RMAN.BA - Delete/rename files

