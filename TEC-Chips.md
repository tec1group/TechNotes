# TEC-1 Chips

## Some notes on the various chips, their substitutes and other related design matters

In short, the TEC was designed to be built with parts that were cheap and readily avialble in the mid 1980's and could be assembled by hobbyists. Hence, cheap single sided PCBs and through-hole parts. PCB layout was done by hand and the part spacing was deliberately left 'broad' again for simplicity, ease of assembly and fault finding, and to a degree to give a more logical layout where the circuit Diagram and the hardware appeared visually similar. Remember this was a design built for learning, where efficiency of PCB space and speed of asembly were not important considerations.

Little thought was given to the world 40 years later in terms of part availability or future design expansion.

### Substituting the 74LS273 for other chips

The 74LS273 can be replaced by a 74LS374 or 74LS377 chip quite easily.

On the TEC-1B PCB, pin 1 of these chips is taken to +5v for the '273. For the '374 or '377, it is taken to GND. - there are two jumpers on the PCB for each chip, fit one *or* the other jumper, *never* both. Fitting both jumpers will short out the +5v regulator!!

No other changes are required to mix chips. Both the '374 and '377 can substitute for the '273; no software changes are required. On other model TECs and the SC-1 you may need to cut tracks to alter the pin 1 connection (or bend out pin one and run a jumper wire).

### Why the 4049?

In short, the 4049 chip just makes for a better oscillator. The 74xx family of chips are not nearly as good at being oscillators as 40xx chips, unless you go to the 74HCU family (eg 74HCU04 as per SC-1) which is designed for this purpose specifically.

You could probably build a working example from a 74xx chip, but getting it stable and reliable, espcially over a frequency range that can be adjusted by trimpot with the device operational, is a great challenge. It was simply just easier to use a more suitable part.

### Why not use a crystal or an oscillator module?

Mainly, cost. Back in the 80's, crystals were more expensive and rarer than they are today, also TE didn't stock them. Thats really all there was to it. The variable speed option was really a bit of a gimmick; Colin saw it as a 'learning aid' to slow down the machine to be able to see what it was doing at an easier to follow pace (e.g. observe the displays being scanned one by one, or the pitch of a note change). Hence, the oscillator module ad-on. Today I just drop in a 14 pin DIL oscillator module and be done with it!!

### 74xx families

The 74LS familiy was originally used because it was common and cheap. Any 74 family that works at TTL 5v and TEC clock speeds will do - LS, HC, AS, ALS, F etc. and you can mix and match as much as you like. At 4MHz and above however, the HC family becomes more suitable [HC == High speed, CMOS] and should be the preferred choice.

The SC-1's 74HCU04 however is critical for the oscillator to be stable - don't substitute here.

### EPROM

The 2716/2732 EPROM can be direclty replaced by the 28Cxx EEPROM equivilant, and is a drop in replacement.

The 28C64 is still readily available in 20201, and will drop in with a minor mod to wire up the extra 4 pins.

### Z80/Z80A vs Z80B etc.

Any of the Z80 family including the NEC D780C chip will work. Remember the really early Zilog chips may not support 4MHz (Needs Z80A). The Z80B is also fine - it just has a higher *maximum* clock speed, but even a 20MHz part will still only run at 4MHz in an original TEC, becuase the crystal oscilator (or 4049 clock) sets the actual clock speed. What's printed on the CPU chip is only it's maximum rating.

The original Z80 (the non-A version) also is NMOS and does not support the CPU clock being stopped. All CMOS chips support stopping the clock entirely.


## ROM and RAM chip substitutions

Almost any size Chip from the 62xx and 27xx/28xx family can be substituted in on the TEC. The 28xx EEPROMs work fine, but of course need to be programmed externally, the TEC can't program iself even with an EEPROM fitted, as it doen't support the correct write timing and Programming voltages.

The Atmel 28C64 for example can easily stand in for the 2716 or 2732. There is some wiring to do - simply ground A12, jumper VCC to the TEC's pin 24, tie WE high (bascially connect pins 28,27 and 26 all together, ground pin 2) and leave pin 1 not connected. Then drop the chip into the TEC with the extra 4 pins hanging down towards the speaker (i.e. pin 14 of the new chip - GND - goes into pin 12 of the TEC's 2716 socket). Program the bottom 4k with MON1B & MON2 or the bottom 2k with JMON. The rest of the chip's contents doesn't matter, but traditionally would be filled with FFs.

It is very cool to note that the designers effectively allowed 'backwards compatability' with the chip pin-outs so you can easily drop a bigger chip into an existing design with a minumum of re-wiring.

The 6264 (8k) RAM chip can also easily sub in for the 6116, again just ground the two higher order address pins (A11 and A12) so as to "emulate" a 2k chip. You can't use all 8k as-is due to the TEC address decoder only decoding 2k blocks, but this can be redesigned to support all 8k with aditional address deconing hardware...the TEC doesn't make this easy however as mapping 8k into a block starting at 2k in the Z80 addres space isn't easily decodable using a single chip. I'm sure the is a combination of gates that could do it but I have not attempted it personally.

An easier approach may be to map the 6264 from address space 0, and get 6k out of it, by disabling the 'first' 2k; this is easily done with an additional 74LS138; but the ROM CE must be used also to ensure the RAM is not active to the BUS during and ROM address space accesses...this could be done by connecting ROM CE to G1 of the additional 74LS138, and A13/A14/A15 to the A,B,C inputs of this '138; wire MRQ to G2A as per existing '138).
