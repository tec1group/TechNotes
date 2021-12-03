# TEC-1 Hardware Memory Map, RAM and ROM notes.

The TEC-1's hardware memory map is created by the 74ls138 that is attached to address lines A11, A12 and A13

Because of this, the address decoder decodes 2k blocks. This was chosen because the 2716 & 6116 chips (both 2k in size) were common at the time of the design. Because A14 and A15 are not decoded, Address 4000h, 8000h, C000h 'wrap around' to 0000h, therefore any program accessing Addresses above 4000h risk overwriting memory by accident.

```
0000h-07ffh - 2k MONitor ROM
0800h-0fffh - 2k RAM
1000h-17ffh - 2k Exansion Port
1800h-1fffh - uncommitted
2000h-27ffh - uncommitted	
2800h-2fffh - uncommitted
3000h-37ffh - uncommitted
3800h-3fffh - uncommitted
```

Any uncommitted 2k block can be put to any purpose e.g. RAM Stack, NVRAM, E(E)PROM, TEC EPROM burner, etc.

To prevent this, address lines A14 and A15 need to be added to the 74ls138. Connecting A14 to pin 5 of the 74ls138 will expand the address 'wrap around' range to 8000h. Connecting A15 via an inverter to pin 6 of the 74ls138 will fully decode the full 64k and prevent any wrap-around. (disconnect pins 5 and 6 from ground/power, obviously).

To support larger size chips e.g. 8k 6264 RAM & 27c64 EPROM, simply pick higher address lines to decode. For example, to support the 8k chips (and to decode 8k blocks instead of 2k) - simply swap A11/12/13 for A13/14/15 instead. i.e. A13 to pin 1, A14 to pin 2 and A15 to pin 3. This automatically eliminates the 'wrap around' problem also!! This is the approach taken by the SC-1 and is a logical progression to support the newer,larger chips.

Any more modern TEC 'redesign' would be logical to move to 8k (or even higher size block) addressing. This was done in the Southern Cross SC-1, for example. The other problem with not decoding 2k blocks is, since all the MONitors look for 2k of RAM at 0800h, the monitor will need to be modifed so that variables, stack etc. are placed where the RAM actually is - i.e. from 2000h upwards.

In an extreme case, a 32k ROM + 32k RAM design can eliminate the 74ls138 entirely, and simply use A15 as the 'chip select' signal - as-is for ROM and inverted, for RAM.

The TEC MONitors were generally written by hand and are full of hard coded addresses, meaning any such project would be a major undertaking. Check the TEC github - someone may have done it :)

Generally, any Z80 CPU hardware design needs ROM placed at 0000h since the power-on program counter address is 0000h - so the first CPU instruction is always fetched from 0000h. If you have RAM or "nothing" at this address, you can't guarantee what instruction the CPU will execute first. Also, the NMI, INT and restart vectors all point to addresses within the first 100h bytes (e.g. NMI at 00066h). In the TECs case, there needs to be the keyboard handler at 0066h (NMI vector).

Over the years there have been many clever designs to allow some form of 'bank switching' to enable the Z80 to see a full 64k of RAM - in the case of the TEC it is unlikely that such a feature would ever be needed (640k ought to be enough, anyone?) and so we will leave such advanced ideas for others to explore.




## Useful MONitor Memory Addreses

The various MONitors offer interesting items at various memory addresses. A brief list follows:


### MON-1 ROM Routines

See issue 10 for documnetation on the following ROM routines.

```
018Eh - Beep Routine. Makes a beep sound.
01B0h - Music Routine. Plays a musical note sequence based on note data stored at 0800h.
0270h - Display routine. Scrolls a series of letters across the 7-seg displays from right to left based on character data stoed at 0800h.
0320h - Invaders game
03E0h - NIM game
0490h - Luna Lander game
```

### MON-1A additions:

```
05B0h - Sequencer routine. See Issue 11, page 29.
```

### MON-1 RAM locations

```
0800h - User RAM start
0DF0h - Stack Pointer Close to top of First 2k of RAM. Stack grows downwards until it overwrites user code.
```

MON1 stores it's variables & stack at the top of the first 2k of RAM - 0Fxxh - meaning if a program uses all 2k, you need to 'leave a hole' here. A bug with stack routines (e.g. one PUSH too many) will see the stack eventually overwrite user code, hence corrupting the program in RAM and almost certainly crashing the machine and/or performing unexpected operations.



### MON2 RAM locations

```
0800h - 08FFh - MONitor variables and stack
0900h - User RAM start
08c0h - Stack Pointer The stack grows downwards so is a maximum of C0 bytes in size.
08D8h - Address Current MONitor Address in 4 nibbles over 4 bytes
08DFh - Mode Flag Monitor data entry mode (0 = Address Mode, 1 = Data Mode)
08E0h - Keyboard Buffer	 ffh if no key pressed, otherwise scan code read from 74c923
08E8h - Original Stack Pointer save MONitor saves the original stack pointer here when first initialised
```

MON2 improves on MON1 by moving its data into the first 100h bytes of RAM, meaning the memory map is contiguous and doesn't need the above 'hole'. This also means a bug that overwrites yser RAM is less likely to crash the MONitor.

Similarly, as the stack grows downwards it will reach ROM space first, meaning a stack bug-induced crash is less likely to overwrite user code.



### JMON RAM locations

```
0900h - User RAM start
????h - Stack Pointer
```

## ROM and RAM chip substitutions

Almost any size Chip from the 62xx and 27xx/28xx family can be substituted in on the TEC. The 28xx EEPROMs work fine, but of course need to be programmed externally, the TEC can't program iself even with an EEPROM fitted, as it doen't support the correct write timing and Programming VCC voltages.

The Atmel 28C64 for example can easily stand in for the 2716 or 2732. There is some wiring to do - simply ground A12, jumper VCC to the TEC's pin 24, tie WE high (bascially connect pins 28,27 and 26 all together, ground pin 2) and leave pin 1 not connected. The drop the chip into the TEC with the extra 4 pins hanging down towards the speaker (i.e. pin 14 of the new chip - GND - goes into pin 12 of the TEC's 2716 socket). Program the bottom 4k with MON1B & MON2 or the bottom 2k with JMON. The rest of the chip's contents doesn't matter, but traditionally would be filled with FFs.

It is very cool to note that the designers effectively allowed 'backwards compatability' with the chip pin-outs so you can easily drop a bigger chip into an existing design with a minumum of re-wiring.

The 6264 (8k) RAM chip can also easily sub in for the 6116, again just ground the two higher order address pins (A11 and A12) so as to "emulate" a 2k chip. You can't use all 8k as-is due to the TEC address decoder only decoding 2k blocks, but this can be redesigned to support all 8k with aditional address deconing hardware...the TEC doesn't make this easy however as mapping 8k into a block starting at 2k in the Z80 addres space isn't easily decodable using a single chip. I'm sure the is a combination of gates that could do it but I have not attempted it personally.

An easier approach may be to map the 6264 from address space 0, and get 6k out of it, by disabling the 'first' 2k; this is easily done with an additional 74LS138; but the ROM CE must be used also to ensure the RAM is not active to the BUS during and ROM address space accesses...this could be done by connecting ROM CE to G1 of the additional 74LS138, and A13/A14/A15 to the A,B.C inputs of this '138; wire MRQ to G2A as per existing '138).

