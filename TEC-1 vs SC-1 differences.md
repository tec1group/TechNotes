# TEC-1 and SC-1 differences

This is a list of differences between the TEC-1 and SC-1 computer, that would matter for a programmer trying to port code from one to the other.

## MONitor

The various TEC MONitor ROMs and the SCMON share very few similaritie, and were developed independently.

The SCMON has a set of callable routines (to scan the display, read keyboard etc) which are standardized via a RST 30h interface and available across MONitor versions.

The TEC MONitros have no such option although JMON did start ot offer some routines via RSTs e.g. RST 20h for keyboard input.

## Memory Map

The TEC-1 prior to the 1F model uses 2k addressing; wheras the 1F and the SC1 use 8k addressing.

Please see https://github.com/tec1group/TechNotes/blob/main/Memory%20Map%20notes.md for detailed meory maps.

Programmers will need to adjust use of memory, especially the code starting address (0800h, 0900h or 2000h) to suit the target platform. This may involve re-writing JP instructions and where varaibles are stored, for example.

## IO MAP

The TEC uses IO ports 0,1 and 2 interrnally (also 3, 84h and 4 in JMON).
The SC1 uses ports 84h, 85h and 86h.

See https://github.com/tec1group/TechNotes/blob/main/IO%20Port%20Maps.md for detailed IO maps.

There is a fairly straight forward relationship between port definitions, however the specific bit(s)of each port are redefined.

TEC == SC

port 2 == 85h == 7 seg segment select
port 1 == 84h == 7 seg digit & speaker select
port 0 == 86h == keyboard input

Note the mapping of display segments is different in the sC vs. the TEC and so a different table of which segementto light for any given character is needed.

Keyboard handling is entirely different and disussed at https://github.com/tec1group/TechNotes/blob/main/Keyboard%20Notes.md


