# A primer on Keyboard Handling on the TEC-1 & SC-1

## Basic Keyboard input -- Hardware Design

### Main Keypad

In hardware, the 74c923 chip triggers an NMI interrupt to the CPU on each keypress, and presents the pressed key data on Z80 I/O port 0, bits 0-4 (Valid values 0x00 to 0x13) Keys 0-9 and A-F generate the same hex value, keys -, +, GO and ADDR are values 0x10 to 0x13 respectively.

### Shift-Key

The shift key is independent of the 74c923, and effectively just sets bit 5 of port 0 low if pressed (and high if not pressed). This means pressing the shift key alone typically does nothing - it does not generate an NMI or affect code execution. Code must consider the state of bit 5 when reading the keyboard state, to detect the shift-key's state and device what meaning that has, if any. Note the shift-key, being active low, is also effectively the "opposite" of regular keys - a 0 meaning pressed.

The Shift-Key was not an original part of the design and early TEC's (TEC-1 and 1A PCBs) will not have it fitted. See Issue 13 pages 9 and 10 for the Shift key mod details.

## MONitor Differences

What actually happens when a key is pressed varies quite a lot from one monitor to another.

### MON-2

The NMI code at 0x0066 reads the keyboard value and stores it to memory at 0x08E0 (this is the keyboard buffer memory location). A value of 0xFF stored by the MONitor means 'no key pressed'. Any program code reading this location needs to reset it to 0xFF after reading the actual key value, in order ro "reset" the keyboard buffer to the not-pressed state.

The byte in memory includes the shift-key state on bit 5, with bit 5=0 meaning shift is pressed. Note this "negative logic" state. Bits 6 and 7 are undefined, but the NMI code does not strip them out, meaning they probably need to be masked off by the programmer to ensure an accurate key value is being used (i.e. AND with 0x1F), test bit 6 for Shift, etc.

This also means the MONitor assumes the Z80 data bass 'floats' at logic level 1 when any undefined bit is read. (port 0 bits 6 and 7 are not connected and so have no meaning).

### MON-1

The NMI code at 0x0066 reads the keyboard value, ANDs it with 0x1F (i.e. discards bits 5, 6 and 7) and stores it to the A and I registers. Both registers original contents are destroyed.

It appears that the I register is intended to hold the keypress, and the overwriting of the A register is an unintentional bug (John Hardy, MON-1 author, confirms this).

MON-1 Does not support the shift-key and pressing it does nothing due to the AND 0x1F instruction.

### JMON

JMON ignores the NMI, in favour of a polled approach to determining if a key is pressed or not. The NMI code at 0x0066 is simply a RET instruction - effectively a 'do nothing' operation. The NMI interrupt routine code must still exist as NMI is still generated in hardware.

JMON polls port 3 for keyboard status; when port 3 is read, bit 6 is tested. If bit 6=0, a 74c923 key is pressed. If bit 5=0, the Shift key is pressed. The code then reads they keyvalue from port 0 and directly does whatever it wants with it. i.e. only the 'key is pressed' status comes from port 3.

JMON needs a MOD to the TEC known as the 'resistor mod' to support polled keyboard status ONLY IF the DAT/LCD board is *NOT* fitted. The mod is to fit a 4k7 resistor between pin 15 of the 4049 and pin 10 of the Z80. You can actually just put the resistor between pins 10 and 17 of the Z80, which may be easier, since pin 15 of the 4049 goes to pin 17 of the Z80! If the DAT board is fitted, logic on the DAT board replaces the resistor mod. This actually also means that if *ANY* unused IO port is read, bit 6 indicates indicates the keypressed status!!!

Note 1: Be aware that without the resistor-mod (and no DAT board) strange keyboard things can happen - e.g. the TEC may see continuous random keypresses, or, may not see actual keyboard input at all (both due to the floating state of the D6 bit). TEC-1B REV.1 PCBs and newer include the JMON resistor as standard as it allows any software to implement a polled keyboard operation mode regardless of monitor used.

Note 2: If the TEC does not have the Shift key fitted JMON may assume that shift is pressed. This is because D5 is left floating (without shift fitted) and hence bit 5 may be read as logic 1 or 0 depending on how the Z80 sees the High-Z bus state. The result is continuous or random shifted-input values. To fix this, fit the Shift-key resistor (A 4K7 resistor between data bus bit D5 and +5v). Since JMON uses Shift heavily, you probably want to install the Shift key in any case.

JMON appears to use the RST 20h call to 'poll' the keyboard, and stores the read value in the A register and in memory at 0820h, as well as setting various CPU flags (ZF, CF) to indicate the overall state - e.g. new keypress, repeat-key, etc.


### SC-1

The Southern Cross SC-1 borrows from the TEC-1 design, employing the same 74c923 keyboard encoder chip, however it also differs in that an external keyboard buffer chip is added. The buffer chip allows the keyboard port to be polled rather than interrupt driven - the Z80's NMI line is not used for keyboard input. The SCMON Monitor also supports a purely software scanned keyboard modification, eliminating the (hard to get) 74c923 chip, ut does require (another) rewrite of the keyboard code.

The keyboard I/O port is accesible via port 86h on the SC-1. The FN, AD, + and - keys are in the opposite value-order compared to the TEC-1's AD, GO, + and - keys...another subtle design difference.

| Key | TEC | SC |
| ---- |:----:| ----:|
| 0..F | 0..F | 0..F |
|  + | 10 | 12 |
|  - | 11 | 13 |
| Fn | -- | 10 |
| AD | 13 | 11 |
| GO | 12 | -- |

The SC-1 does not have a SHIFT key, instead leveraging the Fn key as a 'command mode' key - the key pressed following Fn then executes one of a number of special functions.

Port 86H:
	Bits 0-4 are the same as the TEC, containing the scancode from the 74c923.
	Bit 5   Logic 1 = 'key pressed'.
	bit 6	Data IN2 pin
	bit 7	Data IN1 pin

The SC-1 uses a purely polled keyboard keyboard interface, much as JMON does, except it tests bit 5=1 (of port 86h) to see if a key is being pressed (whereas JMON looks for bit 6=0).

The SC-1 also has a series of system calls which are accesed via a RST 30h interface. These include keyboard routines which are hardware independent; the system calls support the TEC-1F and SC-1 with either software scanned or hardware(74c923) keyboards. This provides a universal approach and takes all the hard work out of programming the keyboard. 

See the SC-1 Users Manual (available from the SC-1 GitHub repository) for further documentation on the various available system calls.


## The 74c923 chip vs. CPU Clock Speed (and keyboard false triggers)

The 74c923 uses two capacitors to control the keyboard debounce and scanning speeds. These are on pin 6 (scan speed - 100nf) and pin 7 (debounce - 1uf).

As the TEC clock speeds increase, so these values may need adjustment to avoid missed or 'doubled' keypresses. Also, as the keyswitches wear, an increased debounce value may help reduce false presses due to the worn switch contacts bouncing more. Note the higher the debounce cap value, the slower the maximum keypress 'repeat' speed, so you may need to type slower!

The 74c923 datasheet shows a 1uf debounce cap provides 0.01 seconds of debounce. This may need to be increased to, say, 3.3uf for more relaible operation at about 4MHz. The datasheet says that 10uf provides 0.1 seconds and 100uf provides 1.0 seconds of debounce. Some trial and error is needed with the specific cap value that works best for your hardware (switches, clock fequency) & MON type.

Conversely, the 100nf cap results in the 74c923 scanning the keyboard matrix at about 1khz. A 1uf reduces the scan rate to about 100Hz, and a 0.01uf increases to about 10khz (the actual value is a bit less than the round number, but the precise scan rate isn't all that critical). A higher value may be helpful if you're using an external keyboard or third party keypad (especially if it's connected via long/unshielded cable) to reduce false inputs triggered by noise, and/or RFI interference coming from the TEC.


## Keyboard Code - reading keystrokes in software

### MON-1

MON-1 uses an interrupt driven design where the I and A CPU registers are altered by the NMI code at 0066h each time a key is pressed. The I register was intended to be the keyboard buffer register, and the changing of A was an unintentional bug that some TE magazine examples (e.g. Quick Draw) happen to rely upon.

The original design for keyboard input as described in Issue 11, was based around executing a HALT instruction at the point where a keypress was to be obtained. The instruction after HALT is executed with the A and I registers now containing they key code after the key is pressed. Hence, either register can be immediately examined to determine what key was pressed. Hence:
```
.....
HALT
CP 01
JP Z, PRESSED_1
CP 02
JP Z, PRESSED_2
....
```
The use of HALT as a keyboard input routine is used in programs such as 'Quick Draw' (Issue 11) and the 8x8 display training programs (Issue 11).

The fact that the A and I registers are altered at random by keypresses can crash or confuse your code - for this reason MON-1 is not a great platform and MON-2 or JMON should be used instead. It is more good luck than good design which sees the MONitor itself not crash randomly due to this flaw.

HALT can also be used to introduce a deliberate pause or "Press any key to continue" type function, again many TE sample programs use this to good effect.

The major downside of this design, is tha the whole computer is in the HALT state until a key is pressed - no further instructions can execute until a key is pressed. not exactly useful if you want to do something else (e.g. scan the display) while waiting for a keypress.

MON-1 itself uses a simple trick to see if a key is pressed without using HALT, so it can scan the displays while waiting for a keypress - see the following MON-1 code:
```
loop:           ld      a, 0ffh
                ld      i, a            ; set I and A to ffh
                call    (scan displays)
                ld      a, i
                cp      0ffh            ;  if A has now changed (via I), a key has been pressed
		ret nz			;  return with key value in A
		jp loop
```
All that takes place here, is using the I register as a keyboard buffer. First, set I to an invalid value - 0FFh. Later, test the value of I. If I is still equal to 0FFh, then no key has been pressed. If however I has changed, it must be a keypress has occurreed, and the new value in I holds the value of the key pressed. This means also that the code inside (scan displays) cannot alter the I register - it is reserved exclusively as a keyboard buffer. The I register is defined in ROM by the keyboard interrupt handler code at 0066h and cannot be altered.


### MON-2

MON-2 takes a similar interupt driven approach, with one main difference. The NMI code at 0066h stores the keyboard value read in memory at 08E0h, and no registers are altered. In other words, the use of the I register as a keyboard buffer has been replaced with memory location 08E0h. This means many MON-1 based routines in the early TE mags won't run properly under MON-2 since I and A no longer hold the key pressed value.

The HALT based approach will still pause until a key is pressed, but a ld a,08E0h instruction (and possibly ld i,a) will need to be inserted immediately after to fetch the key value.

In MON-2, to do other things like scan the displays while checking for a keypress, the following code is used:
```
loop:		call KEYB
		call (scan displays)
		jp loop
		

KEYB:		push af
                push hl
               	ld hl,08e0h		; 08e0h is the keyboard buffer address
                ld a,0ffh
                cp (hl)                 ; if the buffer still holds $FF, 
                jr z,POP_HLAF           ; no key pressed and return
                ld a,(hl)               ;  a = (KEYDATA)
                and 1fh                 ;  Mask off the high 3 bits
                bit 5,(hl)              ;  test if the Shift bit 5 is on
                jr nz,NOSHIFT           ;  if (shift) is on, then
                add a,14h               ;    add $14 to the contents of A

NOSHIFT:	(take action based on the read key value, which is in A)
		.....
                ld (hl),ffh             ; Reset the Keyboard buffer back to $FF

POP_HLAF:	pop hl
		pop af
		ret
```		
This code is a bit more elaborate as it processes the SHIFT key, however the principal is broadly similar: if the byte in memory at 08e0h is equal to ffh, no key is pressed. If it's different, process it to determine the pressed key + SHIFT, take action, and finally "clear" the keypress by writing ffh to 08e0h.


### JMON

Execute a RST 20h instruction

Test the Carry flag. If set, a key is being pressed, and the key's value is in the A register and also placed in memory at 0820h.
Test the Zero flag.  If set, it means the key is newly detected; if Zero flag is clear the key is 'repeating' (i.e. being held down)

Note: The port (00h) input value read is ANDed with 1Fh, so Shift is not read / processed by RST 20h. Shift state must be checked separately.

## Keyboard input in your code

So, the question becomes - how to best support keyboard input in your code?

### TEC-1

Recall that the various monitors use different approaches:

- MON-1: check I register
- MON-2: Check byte at 08e0h
- JMON: Execute RST 20h, test flags, read (0820h) etc.

This means that you can either use an approach that is specific to a given monitor (one of the above) or you can write your own code that doesn't use the MONitor's routines, hence is monitor-independent.

My personal advice and approach - fit the JOMN resistor mod and use a polled keyboard approach. This works in any monitor and is basically universal. The advantage of a polled keyboard being - you can do other things while testing if a key is preing pressed. The disadvantage is that your code will be bigger, you need to debug it, and clever features like JMONs keyboard auto-repeat are quite complex to write yourself (vs. just using the built in code that already does it for you).

### SC-1

It is highly recommended to use the System Calls in SCMON (RST 30h interface) to access the keyboard as this provides complete hardare independence; however, if you want to do it the 'old' way:

The SC-1 already uses the polled approach, so just change the IO port and data-bit being tested. I tend to use a conditional-define for this purpose so I can compile for one machine or the other, but you can easily change your code to just check IO port 86h instead of 00h, and mask off/bit-test the right bits. This only wrks for a 74c923 equipped SC-1 but not the software scanned keyboard.
