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

To prevent the wrap around issue, address lines A14 and A15 need to be added to the 74ls138. The TEC-1B Rev.1 and above use a simple diode based OR gate to work aorund this; the OR gate ensures A14 and A15 must both be LOW (pin 5 of the 74LS138 is used as the control pin). Using a diode based OR gate works fine at TEC speeds and avoids having to fit another chip.

![TEC-1B Memory Decoder Mod](Memory%20Decoder%20Mod.jpg)

An alternate approach would be connecting A14 to pin 5 of the 74ls138 will expand the address 'wrap around' range to 8000h. Connecting A15 via an inverter to pin 6 of the 74ls138 will fully decode the full 64k and prevent any wrap-around. (disconnect pins 5 and 6 from ground/power, obviously).

To support larger size chips e.g. 8k 6264 RAM & 27c64 EPROM, simply pick higher address lines to decode. For example, to support the 8k chips (and to decode 8k blocks instead of 2k) - simply swap A11/12/13 for A13/14/15 instead. i.e. A13 to pin 1, A14 to pin 2 and A15 to pin 3. This automatically eliminates the 'wrap around' problem also!!

In a modern TEC 'redesign', it would be logical to move to 8k (or even higher size block) addressing. This was done in the Southern Cross SC-1, for example, and now also features as a jumper-selectable option on the TEC-1F PCB designed by Craig Jones.

The other problem with not decoding 2k blocks is, since all the MONitors look for 2k of RAM at 0800h, the monitor will need to be modified so that variables, stack etc. are placed where the RAM actually is - i.e. from 2000h upwards.

In an extreme case, a 32k ROM + 32k RAM design can eliminate the 74ls138 entirely, and simply use A15 as the 'chip select' signal - as-is for ROM and inverted, for RAM.

The TEC MONitors were generally written by hand and are full of hard coded addresses, meaning any such project would be a major undertaking. Check the TEC github - someone may have done it :)

Generally, any Z80 CPU hardware design needs ROM placed at 0000h since the power-on program counter address is 0000h - so the first CPU instruction is always fetched from 0000h. If you have RAM or "nothing" at this address, you can't guarantee what instruction the CPU will execute first. Also, the NMI, INT and restart vectors all point to addresses within the first 100h bytes (e.g. NMI at 00066h). In the TECs case, there needs to be the keyboard handler at 0066h (NMI vector).

Over the years there have been many clever designs to allow some form of 'bank switching' to enable the Z80 to see a full 64k of RAM - in the case of the TEC it is unlikely that such a feature would ever be needed (64k ought to be enough, anyone?) and so we will leave such advanced ideas for others to explore.

## Data Bus Conflict on ROM address space write design flaw

The first 2K of memory address space is of course ROM. As such, the EPROM is enabled onto the CPU data bus any time the bottom 2k of memory is addressed. Normally this address space is only READ from (given it's a ROM!!) however there is nothing stopping the EPROM also responding to memory WRITEs to this address space. This is because on the EPROM, /OE is always enabled and /CS is only driven by /MEMRQ. The state of the /RD and /WR lines is ignored. Hence, the Z80 and the ROM will BOTH try to output data to the bus in a memory write between addreses 0000h and 07ffh - which can happen during a crash (e.g. stack overflow) or just with a simple programming slip.

This was never fixed on the original TEC design (the TEC-1E and 1F are fixed) however it does not seem to cause any damage. If anyone ever tried to make a busmaster add-on (IE use the /BUSRQ and /BUSACK pins) this would be a potential problem.

To resolve this design flaw, connect pin 20 of the EPROM socket (/OE) to Z80 pin 21 (/RD) [disconnect pin 20 from GND obviously]. This means the EPROM is only enabled onto the bus during memory READ cycles and the bus is free during writes.

## Useful MONitor Memory Addreses

The various MONitors offer interesting items at various memory addresses. A brief list follows:

### MON-1 ROM Routines

See issue 10 for documnetation on the following ROM routines.

```
018Eh - Beep Routine. Makes a beep sound.
01B0h - Music Routine. Plays a musical note sequence based on note data at 0800h.
0270h - Display routine. Scrolls a series of letters across the 7-seg displays R to L based on character data at 0800h.
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
0DF0h - Stack Pointer (Close to top of First 2k of RAM.)
```

MON-1 stores it's variables & stack at the top of the first 2k of RAM - 0Fxxh - meaning if a program uses all 2k, you need to 'leave a hole' here. A bug with stack routines (e.g. one PUSH too many) will see the stack eventually overwrite user code, hence corrupting the program in RAM and almost certainly crashing the machine and/or performing unexpected operations.

### MON-2 RAM locations

```
0800h - 08FFh - MONitor variables and stack
0900h - User RAM start

08c0h - Stack Pointer. The stack grows downwards so is a maximum of C0 bytes in size
08D8h - Address Current MONitor Address in 4 nibbles over 4 bytes
08DFh - Mode Flag. Monitor data entry mode (0 = Address Mode, 1 = Data Mode)
08E0h - Keyboard Buffer: contains ffh if no key pressed, otherwise contains the scan code read from 74c923; scancode +14h if shift pressed as well
08E8h - Original Stack Pointer save MONitor saves the original stack pointer here when first initialised
```

MON-2 improves on MON-1 by moving its data into the first 100h bytes of RAM, meaning the memory map is contiguous and doesn't need the above 'hole'. This also means a bug that overwrites yser RAM is less likely to crash the MONitor.

Similarly, as the stack grows downwards it will reach ROM space first, meaning a stack bug-induced crash is less likely to overwrite user code.

### JMON RAM locations

```
0800h - 08FFh - MONitor variables and stack
0900h - User RAM start

0820h - Stack Pointer - Maximum 20h bytes
```
