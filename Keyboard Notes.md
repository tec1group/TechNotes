# A primer on Keyboard Handling on the TEC-1 & SC-1

## Basic Keyboard input -- Hardware Design

### Main Keypad

In hardware, the 74c923 triggers an NMI inteupt to the CPU on each keypress, and presents the pressed key data on Z80 I/O port 0, 
bits 0-4 (Valid values 0x00 to 0x13) Keys 0-9 and a-f generate the same hex value, keys -, +, GO and ADDR are values 0x10 to 0x13 respectively.

### Shift-Key

The shift key is independent of the 74c923, and effectively just sets bit 5 of port 0 low if pressed. 
This means pressing the shift key alone typically does nothing - it does not generate an NMI or affect code execution. Your code
must consider the state of bit 6 to decide what to do. 

The Shift-Key was not an original part of the design and early TEC's (TEC-1 and 1A PCBs) will not have it fitted.
See Issue 13 pages 9 and 10 for the Shift mod details.

## MONitor Differences

What actually happens when a key is pressed varies from one monitor to another.

### MON2

The NMI code at 0x0066 reads the keyboard value and stores it to memory at 0x08E0 (this is the keyboard buffer memory location). 
A value of 0xFF stored by the MONitor means 'no key pressed'. Any program code reading this location needs to reset it to 0xFF 
after reading the actual key value, in order ro "reset" the keyboard buffer to the not-pressed state.

The byte in memory includes the shift-key state on bit 5, with bit 5=0 meaning shift is pressed. 
Note this "negative logic" state. Bits 6 and 7 are undefined, but the NMI code does not strip them out, 
meaning they probably need to be masked off by the programmer to ensure an accurate key value is being used 
(i.e. AND with 0x1F), test bit 6 for Shift, etc.

This also means the MONitor assumes the Z80 data bass 'floats' at logic level 1 when any undefined bit is read. (port 0 bits 6 and 7 are not connected).

### MON-1B

The NMI code at 0x0066 reads the keyboard value, ANDs it with 0x1F (i.e. discards bits 5, 6 and 7) and stores it to the A and I registers. Both registers original contents are destroyed.

I have not fully examined the MON1 code, but it appears that the I register is intended to hold the keypress, at least initially, and the overwriting of the A register is an unintentional bug. I'm not sure if ultimately the keypress is stored somewhere in memory, or not.

MON1 Does not support the shift-key and pressing it does nothing due to the AND 0x1F instruction.

### JMON

JMON ignores the NMI, in favour of a polled approach to determinig if a key is pressed or not. The NMI code at 0x0066 is simply a RET instruction - effectively a 'do nothing' operation. The NMI interrupt routine code must still exist as NMI is still generated in hardware.

JMON first determines (at reset) if the LCD/DAT Board is present, or not & sets a flag in memory to remember which port to poll for keyboard data.

If the LCD is present, polls port 3 for keyboard status; if not, it polls port 0. When the port is read, bit 6 is tested. If bit 6=0, a 74c923 key is pressed. If bit 5=0, the Shift key is pressed. The code then reads they keyvalue from port 0 and directly does whatever it wants with it. i.e. only the 'key is pressed' status comes from port 3, and then only when the DAT board is fitted.

JMON needs a MOD to the TEC known as the 'resistor mod' to support polled keyboard status ONLY IF the DAT/LCD is *NOT* fitted. The mod is to fit a 4k7 resistor between pin 15 of the 4049 and pin 10 of the Z80. You can actually just put the resistor between pins 10 and 17 of the Z80, which may be easier, since pin 15 of the 4049 goes to pin 17 of the Z80!

Note: If the TEC does not have the Shift key fitted JMON may assume that shift is pressed. This is because D5 is left floating (without shift fitted) and hence bit 5 may be read as logic 1 or 0 depending on how the Z80 sees the High-Z bus state. The result is continuous or random keypress inputs. To fix this, fit the Shift-key resistor (A 4K7 resistor between data bus bit D5 and +5v.)

Since JMON uses Shift heavily, you probably want to install the Shift key in any case.

I am not certain if JMON then stores the keyvalue or provides any direct keyboard routines -- please refer to Jims notes && Issue 15 page 24 for further insight.

### SC-1

The Southern Cross SC-1 borrows from the TEC-1 design, employing the same 74c923 keyboard encoder, however it also differs in that an external keyboard buffer chip is added. Also, the keyboard I/O port is port 86h on the SC-1. The FN, AD, + and - keys are in the opposite value-order compared to the TEC-1's AD, GO, + and - keys...another subtle design difference.

The SC-1 does not have a SHIFT key, instead leveraging the Fn key as a 'command mode' key - the key pressed following Fn then executes one of a number of special functions.

Port 86H:
	Bits 0-4 are the same as the TEC, containing the scancode from the 74c923.
	Bit 5   Logic 1 = 'key pressed'.
	bit 6	Data IN2 pin
	bit 7	Data IN1 pin

The SC-1 uses a purely polled keyboard keyboard interface, much as JMON does, except it tests bit 5=1 to see if a key is being pressed (whereas JMON looks for bit 6=0).


## The 74c923 chip vs. CPU Clock speed and keyboard false triggers)

The 74c923 uses two capacitors to control the keyboard debounce and scanning speeds. These are on pin 6 (scan speed - 100nf) and pin 7 (debounce - 1uf).
As the TEC clock speeds increase, so these values may need adjustment to avoid missed or 'doubled' keypresses. Also, as the keyswitches wear,
an increased debounce value may help reduce false presses due to the worn switch contacts bouncing more. Note the higher the debounce cap value, the slower
the maximum keypress 'repeat' speed, so you may need to type slower!

The 74c923 datasheet shows a 1uf debounce cap provides 0.01 seconds of debounce. This may need to be increased to, say, 3.3uf for more relaible operation
at and about 4MHz. The datasheet says that 10uf provides 0.1 seconds and 100uf provides 1.0 seconds of debounce. some trial and error is needed with the
specific cap value that works best for your hardware (switches, clock fequency) & MON type.

Conversely, the 100nf cap results in the 74c923 scanning the keyboard matrix at about 1khz. A 1uf reduces the scan rate to about 100Hz, and a 0.01uf
increases to about 10khz (the actual value is a bit less than the round number, but the actual rate isn't all that critical). A higher value may be
helpful if you're using an external keyboard or third party keypad (especially if it's connected via long/unshielded cable) to reduce false inputs
triggered by noise.
