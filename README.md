# superacan-notes

All research notes about super acan.

# About the controller

The official super acan controller is hard to find and not good for game playing.
But you can use the SNES controller on super acan with a custom adapter.
The controller protocol is the same as SNES, but the button mapping is a little different.

| Acan | SNES |
|------|------|
|      |   B  |
|      |   Y  |
| START|SELECT|
|SELECT| START|
|   Up |   Up |
| Down | Down |
| Left | Left |
| Right| Right|
|      |   A  |
|      |   X  |
|   L  |   L  |
|   R  |   R  |

Not very sure and not tested but some game should support multitap (which never officially released, only mentioned on manual).

# About the 6650 chip

This should be a CIC chip and when power on the 68k CPU runs code which store inside 6619 chip (like Genesis TMSS).
The code basiclly doing some simple check on 6650 and checks a special string reading from 6650.
CPU communicate with 6650 with two memory location(register):

- 0xEB0D02 (R)
> Which address to access inside 6650.
- 0xEB0D00 (R/W)
> Data on that address.
