# TEC-1 Chips

## Some notes on the various chips, their substitutes and other related design matters

In short, the TEC was designed to be built with parts that were cheap and readily avialble in the mid 1980's and could be assembled by hobbyists. Hence, cheap single sided PCBs and through-hole parts. PCB layout was done by hand and the part spacing was deliberately left 'broad' again for simplicity, ease of assembly and fault finding, and to a degree to give a more logical layout where the circuit Diagram and the hardware appeared visually similar. Remember this was a design built for learning, where efficiency of PCB space and speed of asembly were not important considerations. Cost was a driving factor, hence single-sided PCB (Double Sided PCBs were 50% more, plate thru holes, even more again.)

Little (well, absolutely no) thought was given to the world 40 years later in terms of part availability or future design expansion.

### 8212 vs 74LS273

The original TEC used an 8212 latch chip. The 8212 is an Intel part made in the 1970's as part of the 8008,8085,8080 microprocessor families and became end of life only a few years after the TEC was designed. As the chip was designed for Intel CPUs, it was both costly and also physically large as it was considered a custom item.

The 74LS273 is of course a generic 74xxx family chip and not a "special" part. It was both cheaper and physically smaller. It is 100% TEC software compatible and (with the phyical pinout/size differences allowed for) also TEC hardware compatible. TE were very lucky to be able to swap in a generic chip with only a quick PCB change and no software re-writes.

### Substituting the 74LS273 for other chips

The 74LS273 can be replaced by a 74LS374 or 74LS377 chip quite easily.

On the TEC-1B PCB, pin 1 of these chips is taken to +5v for the '273. For the '374 or '377, it is taken to GND. There are two jumpers on the PCB for each chip, fit one *or* the other jumper, *never* both. Fitting both jumpers will short out the +5v to ground!!

No other changes are required to mix chips. Both the '374 and '377 can substitute for the '273; no software changes are required. On older model TECs and the SC-1 you may need to cut tracks to alter the pin 1 connection (or bend out pin 1 and run a jumper wire).

### Why the 4049?

In short, the 4049 chip just makes for a better oscillator. The 74xx family of chips are not nearly as good at being oscillators as 40xx chips, unless you go to the 74HCU family (eg 74HCU04 as per SC-1) which is designed for this purpose specifically. Theres a lot of design science behind all of it which I won't go into; the short version is the 74xx is more unstable and tends to result in either not a good range of ftrequency adjustment, unsable frequency, or glitches when being adjusted that tend to crash the CPU, whereas the 4049 has none of these problems.

You could probably build a working example from a 74xx chip, but getting it stable and reliable, espcially over a frequency range that can be adjusted by trimpot with the device operational, is a great challenge. It was simply just easier to use a more suitable part.

### Why not use a crystal or an oscillator module?

Mainly, cost. Back in the 80's, crystals were more expensive and rarer than they are today, also TE didn't stock them. Thats really all there was to it -- it was about saving money and simplifying supply logistics by not stocking an odd, unique part and having to setup purchasing accounts etc. with another supplier. The variable speed option was really a bit of a gimmick; Colin saw it as a 'learning aid' to slow down the machine to be able to see what it was doing at an easier to follow pace (e.g. observe the displays being scanned one by one, or the pitch of a note change) but that was really a justification for using the cheaper parts, rather than a crystal.

Hence, the oscillator module ad-on eventually came out some years later. By then, crystals had gotten cheaper and more readily avilable in 'common' frequencies. 3.5795MHz is a common frequency used in numerous devices, making them cheap and available enough to source in smaller quantities (TE would buy maybe 100 at a time -- most electronic distributors would have minimum order terms of 5,000 units at a time which was excessive --at $2 a crystal, it was not reasonable for TE to "tie up" $10,000 or so in just one inventory component used in only one kit that would in all likelyhood sell a couple of hunded units!). Today I just drop in a 14 pin DIL oscillator module as a clock source (the TEC still needs the 4049 inverter) and be done with it!!

Note the TE oscillator module also divides the crystal frequency by two - so you need an 8MHz crystal for a 4MHz Z80 clock - or remove the 'divide by two' option by removing the 7473 and connecting pin 5 to pin 9 instead. The divide by two section was included so that the module worked at ~1.79MHz and was thus compatible with the original Z80 (not Z80A) chips.

### 74xx families

The 74LS familiy was originally used because it was common and cheap. Any 74 family that works at TTL 5v and TEC clock speeds will do - LS, HC, AS, ALS, F etc. and you can mix and match as much as you like. At 4MHz and above however, the HC family becomes more suitable [HC == High speed, CMOS] and should be the preferred choice if buying new parts today.

The SC-1's 74HCU04 however is critical for the oscillator to be stable - don't substitute here.

### Z80/Z80A vs Z80B etc.

Any of the Z80 family including the NEC D780C chip will work. Remember the really early Zilog chips may not support 4MHz (Needs Z80A). The Z80B is also fine - it just has a higher *maximum* clock speed, but even a 20MHz part will still only run at (up to) 4MHz in an original TEC, because the crystal oscilator (or 4049 clock) sets the actual clock speed. What's printed on the CPU chip is only it's maximum rating.

The original Z80 (the non-A version) also is NMOS and does not support the CPU clock being stopped. All CMOS chips support stopping the clock entirely.

## ROM and RAM chip substitutions

Almost any size static RAM chip from the 62xx and 27xx/28xx family can be substituted in on the TEC. Obviously pin wiring adjustments need to be made but in terms of bus timing and address decode, but fundamentally any static RAM following the '2716'/'6116' RAM timing conventions will work.

The TEC when clocked at up to 4MHz works well with chips offering a 150ns or faster access time. Faster chips work fine e.g. 120ns, 100ns, 80ns etc. Generally, the TEC works with almost any chips on the market, excluding perhaps the most vintage of early 1970's parts of 250ns++ access times.

### 2716 EPROM

The 2716 or 2732 EPROM can be directly replaced by the 28Cxx EEPROM equivilant, and is a 'drop in' replacement with a couple of wiring mods on the extra pins for the larger capacity parts.

The 28C64 is still readily available in 2021. The TEC-1F PCB can accept a 2864 directly. The 28xx EEPROMs work fine, but of course need to be programmed externally; the TEC can't program itself even with an EEPROM fitted, as it doen't support the correct write timing and programming voltages.

The Atmel 28C64 can easily be substituted in as follows - seat the chip into the TEC-1 ROM socket with the extra 4 pins hanging down towards the speaker (i.e. pin 14 of the new chip - GND - goes into pin 12 of the TEC's 2716/32 socket). This leaves 4 extra pins 'overhanging' at the pin-one end. Simply ground A12, jumper VCC to the TEC's pin 24, tie WE high (bascially connect pins 28, 27 and 26 all together, ground pin 2) and leave pin 1 not connected. Program the bottom 4k with MON1B & MON2 or the bottom 2k with JMON. The top 4k of the chip's contents doesn't matter, but traditionally would be filled with FFs.

It is very cool to note that the memory designers effectively allowed 'backwards compatability' with the chip pin-outs so you can easily drop a bigger chip into an existing design with a minumum of re-wiring. You can see how the evolution of 2716->32->64 (etc) have kept the pinouts as compatible as possible.

### 6116 RAM

The 6264 (8k) RAM chip can also easily sub in for the 6116, again just ground the two higher order address pins (A11 and A12) so as to "emulate" a 2k chip. You can't use all 8k as-is due to the TEC address decoder only decoding 2k blocks, but this can be redesigned to support all 8k with aditional address decoding hardware (Or use a TEC-1F PCB which has 8k support)...the TEC doesn't make this easy however as mapping 8k into a block starting at 2k in the Z80 address space isn't easily decodable using a single chip. I'm sure there is a combination of gates that could do it but I have not attempted it personally.

An easier approach may be to map the 6264 from address space 0, and get 6k out of it, by disabling the 'first' 2k; this is easily done with an additional 74LS138; but the ROM CE must be used also to ensure the RAM is not active to the BUS during and ROM address space accesses...this could be done by connecting ROM CE to G1 of the additional 74LS138, and A13/A14/A15 to the A,B,C inputs of this '138; wire MRQ to G2A as per existing '138).

### 74c923 Keyboard Encoder

The 74c923 keyboard encoder is no longer readily available and is proving to be an issue with the long term future for new TEC builds. Buy up when and where you can as there is no 'drop in' equivilant. Limited stock of the 74c923 was still available via Rockby Electronics (Clayton, Vic) as of this writing (July 2022).

As a work-around, the TEC-1F offers a scanned keyboard option, however this is not software-compatible with any of the original TEC MONitors.

### 7805 Voltage regulator & power supply

The 7805 is still readily available (it is so common it is unlikely to ever go off the market), however there is no need to use the TECs built in power supply - any source of regulated 5 volts able to deliver around300mA will do. A typical NMOS Z80 equipped TEC seems to draw a little over 200mA when idle sitting at the MONitor prompt, with no peripherals atttached. CMOS Z80's, quite a bit less. An LM340T5 is a direct replacement for a 7805.

Do not use the 78L05 -- this can only deliver ~100mA and will fail in the TEC as it will be overloaded.
