<h1>TEC-1 I/O ports Definitions</h1>

For the standard TEC-1x family (1/1A/B/C/D etc) per TE issues 10-15:

If we define 'standard' monitors, as MON1 family & MON 2, then ports 0-2 are defined. With the DAT board (and JMON), ports 3, 4 & 84h are additonally defined, as follows.

NB: Many TEC add ons used various ports (3 & 4 mostly) but the code can usually be changed easily to support any port avaiallbe in hardware as the MONitors don't touch these ports in any way.

Port 0 - Keyboard data input:<br>
 - bits 0-4 -- keypressed value from the 74c923<br>
 - bit 5 - low = Shift pressed (Mon2 +)<br>
 - bit 6 - low = Key pressed - used by JMON with 4K7 resistor mod only<br>
 - bit 7 - not connected<br>

Port 1 - 7-seg Display select & speaker (& JMON tape out) output
 - bits 5-0 select seach 7-segment display, bit 5 = left, bit 0 = right
 - bit  6 - not connected
 - bit  7 - speaker output & tape output (tape --- JMON with DAT board only)

Port 2 - 7-seg segment select output<br>
 - bit 0	segment a<br>
 - bit 1	segment f<br>
 - bit 2	segment g<br>
 - bit 3	segment b<br>
 - bit 4	decimal point<br>
 - bit 5	segment c<br>
 - bit 6	segment e<br>
 - bit 7	segment d<br>

Segments are defined as per the FND-500 7-segment display specification.
	
Port 3 - DAT board serial/inputs/keypresss sense/tape input<br>
 - bit 0	serial input<br>
 - bit 1	IN port 1<br>
 - bit 2	IN port 2<br>
 - bit 3	IN port 3<br>
 - bit 4	IN port 4<br>
 - bit 5	IN port 5<br>
 - bit 6	Keypress sense - LOW = a 74c923 key is being pressed<br>
 - bit 7	Casette input<br>

Port 4 - DAT board LCD control port output

	As per the HD44780 16x2 LCD Spec - Instruction Register

(Most of the software written for the 8x8 assumes it is on Port 3 & 4)

Ports 5/6/7 - uncommitted
(With the DAT / LCD boards taking up ports 3 & 4, any software written for the 8x8 should aim for using Ports 5 & 6)

Port 84h - JMON/DAT board LCD data port input/output

	As per the HD44780 16x2 LCD Spec - Data Register

In JMON, the LCD picks up A7 as the 'register select' line, hence the port 04h/84h mapping.



TEC-1 I/O Address Space & Expansion

In a standard TEC only 8 I/O ports are decoded, so the I/O space 'wraps' every 8 ports. Adding A3 to pin 5 of the 74ls138 (disconnect pin 5 from GND and connect to A3 instead) expands this to 16 ports.

To decode ports 8-15, add another 74ls138, wired exactly as per the existing, except connect A3 to pin 6 instead of +5v & Keep pin 5 Grounded).

For >16 ports, A4 could also be used if needed to decode up to 32 ports. Connect A4 to pin 6 via an inverter to the first two 74ls138's, add two more 74ls138's and connect A4 directly (not inverted) to the second two '138's on pin 6 also. (Disconnect pin 6 from +5v on all 4 chips).


For Southern Cross SC-1:<br>
Ports 80h to 83h - uncommitted - available on the I/O port expansion connector<br>
Port 84h - 7-seg segment select output<br>
Port 85h - 7-seg Display select & speaker (& JMON tape out) output<br>
Port 86h - Keyboard data input<br>
Port 87h - ???<br>

Note that the segment order port-bit is different in the SC-1 vs. the TEC. Any Lookup tables can be re-written easily to adapt one design to the other.

From Craig Jones, Designer of the SC-1:

"The SC-1 uses ports 80 to 83 in the "I/O" port at the top of the board. The I/O port uses a 16 pin IDC to connect to the relay board or up to 2 8x8 displays. I didn't conceive of needing any more than 16 I/O ports so I used A6 and A7 one active low and one active high. This way just by connecting them the other way around I get the other 8 ports. I use port 87 to enable/disable the single stepper."

Note that the SC-1 has the keyboard and 7-segment ports in a different order compared to the TEC.

The SC-1 decodes Address lines A6 and A7 for I/O meaning it has the ability to expand to 32 I/O ports - ranges 40h-4Fh and 80h to 8Fh. The onboard 74hc138 decodes 80h to 87h.

Note the SC-1's address model conflicts with the DAT board's use of 84h, meaning you cant fit a TEC/JMON type LCD to the SC-1 and use JMON in it without additional code rewrites. To fix this, use a different address line instead of A7 - e.g. use of A6 on the DAT board would put the LCD at 04h/44h.
