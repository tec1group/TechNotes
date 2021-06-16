TEC-1 Chips

Some notes on the various chips, their substitutes and other related design matters.



Substituting the 74LS273 for other chips

The 74LS273 can be replaced by a 74LS374 or 74LS377 chip quite easily. On the TEC-1B, pin 1 of these chips is taken to +5v for the 273. For the 374 or 377, it is taken to GND. No other changes are required. Both the 374 and 377 can substitute for the 273; no software changes are required. On other model TECs and the SC-1 you may need to cut tracks to alter the pin 1 connection (or bend out pin one and run a jumper wire).


Why the 4049?

In short, the 4049 chip just makes for a better oscillator. The 74xx family of chips are not nearly as good at being oscillators as 40xx chips, unle you go to the 74HCU family (eg 74HCU04 as per SC-1) which is designed for this purpose specifically.

You could probably build a working example from a 74xx chip, but getting it stable and reliable, espcially over a frequency range that can be adjusted with the device operational, is a great challenge. Easier just to use a more suitable part.


Why not use a crystal or an oscillator module?

Mainly, cost. Back in the 80's, crystals were more expensive and rarer than they are today, also TE didn't stock them. Thats really all there was to it. The variable speed option was really a bit of a gimmick; Colin saw it as a 'learning aid' to slow down the machine to be able to see what it was doing at an easier to follow pace (e.g. observe the displays being scanned one by one, or the pitch of a note change). Hence, the oscillator module ad-on. Today I just drop in a 14 pin DIL oscillator module and be done with it!!


74xx families

The 74LS familiy was originally used because it was common and cheap. Any 74 family that works at TTL 5v and TEC clock speeds will do - LS, HC, AS, ALS, F etc. and you can mix and match as much as you like. The SC-1's 74HCU04 howeve ris critical for the oscillator to be stable - don't substitute here.


EPROM

The 2716/2732 EPROM can be direclty replaced by the 28Cxx EEPROM equivilant, and is a drop in replacement. The 28C64 is readily available and will drop in with a minor mod to wire up the extra 4 pins.


Z80/Z80A

Any of the Z80 family including the NEC D780C chip will work. Remember the really early Zilog chips may not support 4MHz (Needs Z80A).
