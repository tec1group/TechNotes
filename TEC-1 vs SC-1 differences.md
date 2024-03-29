# TEC-1 and SC-1 differences

This is a list of differences between the TEC-1 and SC-1 computer, that would matter for a programmer trying to port code from one to the other or a hardware designer, designing a peripheral device.

Whilst the hardware differences are noticeable (code won't run unaltered) the changes to adapt code are also fairly stright forward as the platforms are quite similar (roughly 95% the same) at the conceptual level.

The TEC-1F alo brings the TEC-1 much closer to the SC-1 allowing the two platforms to share even greater commonality such as a common memory map, bitbanged serial IO option, scanned keyboard capability (avoiding the need for the now very rare 74C923 IC), whilst retaining classic TEC compatability where needed.

I strongly urge anyone planning to develop for the TEC-1/SC-1 to use the TEC-1F as a starting point for future design.

## MONitor

The various TEC MONitor ROMs and the SCMON share very few similarities, and were developed quite independently.

The SC-1's SCMON has a set of callable routines (to scan the display, read keyboard etc) which are standardized via a RST 30h interface and available across MONitor versions.

The TEC MONitors have no such option and are completely closed boxes (except for the game restarts in MON-1) although JMON did attempt to offer some routines via standardized RSTs e.g. RST 20h for keyboard input.

## Memory Map

The TEC-1 prior to the 1F model uses 2k addressing; whereas the 1F and the SC1 use 8k addressing.

See https://github.com/tec1group/TechNotes/blob/main/Memory%20Map%20notes.md for detailed memory maps and further discussion.

Programmers will need to adjust use of memory, especially the code starting address (0800h, 0900h or 2000h) to suit the target platform. This may involve re-writing JP instructions and where variables are stored, for example.

#### MONitor variables

Most MONitors use some RAM for internal variables. If this memory is overwritten by software, the MONitor may behave strangely if your program exits back to MONitor, or after the machine is reset.
````
MON2 - 0800-08FFh
JMON - 0800-08FFh
SCMON - 3F00h-3FFFh
`````

The most obvious change here, is porting MON1 to MON2 or JMON requires re-writing code at 0900h instead of 0800h.

#### Stack

The stack location varies however the various MONitors manage this for you and are usually safe to ignore. If your code requires a large stack however, you may need to consider relocating this to .e.g. near top of memory as MON2 and JMON both have limited stack space set aside by default.
````
JMON  0820h - Stack Pointer. Maximum 20h bytes
MON2  08c0h - Stack Pointer. Maximum C0h bytes
MON1  0DF0h - Stack Pointer.
SCMON (varies) - Just below top of RAM; typically 3E00h or 3FEEh. Exact location depends on MONitor version.
````
## IO MAP

The TEC uses IO ports 0,1 and 2 interrnally (also 3, 4 and 84h in JMON).
The SC1 uses ports 84h, 85h and 86h.

See https://github.com/tec1group/TechNotes/blob/main/IO%20Port%20Maps.md for detailed IO maps.

There is a fairly straight forward relationship between port definitions, however the specific bit(s)of each port are redefined.

````
TEC    == SC
port 2 == 85h => 7 seg segment select
port 1 == 84h => 7 seg digit & speaker select
port 0 == 86h => keyboard input
````

Note the mapping of display segments is different in the SC vs. the TEC and so a different table of which segements to light for any given character is needed.

Keyboard handling is entirely different and discussed at https://github.com/tec1group/TechNotes/blob/main/Keyboard%20Notes.md

The overlap of JMON's use of port 84h for the LCD is a direct conflict with the SC1, however since the SC1 has no LCD display, this should not present an issue. Any LCD hardware implemented on the SC1 would simply use a differnet port and therefore would also need a code (IO port use) rewrite.

#### The following code snippet shows the differences required for HEX characters dispalyed on the 7 segment displays on either platform

````asm
HEXADECIMAL TO 7 SEGMENT DISPLAY CODE TABLE

TEC-1
.DB 0EBH,28H,0CDH,0ADH  ;0,1,2,3
.DB 2EH,0A7H,0E7H,29H ;4,5,6,7
.DB 0EFH,2FH,6FH,0E6H ;8,9,A,B
.DB 0C3H,0ECH,0C7H,47H  ;C,D,E,F

SC-1
.DB 3FH,06H,5BH,4FH ;0,1,2,3
.DB 66H,6DH,7DH,07H ;4,5,6,7
.DB 7FH,6FH,77H,7CH ;8,9,A,B
.DB 39H,5EH,79H,71H ;C,D,E,F
````

## Keyboard

THe keyboard is the most nonstandard part of any TEC/SC, with numerous variations and options avilable:

- TEC-1, No SHIFT key
- TEC-1A, With SHIFT key
- TEC-1B with JMON resistor
- TEC with DAT Board
- SC-1 with 74C923
- SC-1 with scanned keyboard (no 74C923)
- TEC-1F with scanned keyboard (no 74C923)
- The SC-1 does not have a SHIFT key

See https://github.com/tec1group/TechNotes/blob/main/Keyboard%20Notes.md for detailed notes on keyboard use.

## Programmers Checklist

Use the following guide to work through re-writing your code for either platform.

- Adjust code starting address to 0800h, 0900h or 2000h. Recalculate absolute JMP addresses. Relocate any RAM based buffers, variables and tables.
- Adjust keyboard input routine
- Adjust 7 segment display scanning routine
- Adjust any routines using the speaker
- Remove any use of MON specific ROM routines

In modern software development, a series of conditional defines can be used in conjunction with a master define, to allow the code to be built quickly on any target platform.

For example:

If you code includes a flag such as:
````
#DEFINE TEC
````
then later in your code you can do:
````
#IFDEF TEC
  (TEC code)
#ELSE
  (SC code)
#ENDIF
````
Simply comment out the #DEFINE to compile for the other platform.
