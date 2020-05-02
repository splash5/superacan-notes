
# superacan-notes

All research notes about super acan.

# About the controller

The official super acan controller is hard to find and not good for game playing.
But you can use the SNES controller on super acan with a custom adapter.
The controller protocol is the same as SNES, but the button mapping is a little different.

| Acan | SNES |
|------|------|
|A     |B     |
|B     |Y     |
|START |SELECT|
|SELECT|START |
|Up    |Up    |
|Down  |Down  |
|Left  |Left  |
|Right |Right |
|X     |A     |
|Y     |X     |
|L     |L     |
|R     |R     |

Not very sure and not tested but some game should support multitap (which never officially released, only mentioned on manual).

# About the 6650 chip

This acts as a CIC chip but only contains at least 32 bytes RAM and 16 bytes ROM. (No CPU inside it I guess)
When powering on the 68k CPU runs code which store inside 6619 chip (like Genesis TMSS) and doing some check
about 6650 (like make sure that RAM exists and work properly) and finally reads a special string and compare it.
CPU communicate to 6650 with two memory location(register):

- 0xEB0D02 (W)
> Which address to access inside 6650.
- 0xEB0D00 (R/W)
> Data on that address.

Unlock procedure pseudo code:

    boolean unlock(void) {
      const uint8_t special[] = {0x9b, 0x65, 0x70, 0x90, 0x29, 0x43, 0x28, 0x20, 0x34, 0x39, 0x39, 0x31, 0x20, 0x43, 0x4d, 0x55};
      uint8_t count; 
      uint8_t mem[32];
    
      for (count = 0; count < 32; count++)
        mem[count] = um6650[0x5f - count];
    
      for (count = 0x5f; count <= 0x40; count--)
        um6650[count] = count;
    
      for (count = 0x5f; count <= 0x40; count--) {
        if (um6650[count] != count)
          return false;
      }
      
      for (count = 0; count < 32; count++)
        um6650[0x5f - count] = mem[count];
      
      for (count = 0; count < 16; count++) {
        if (um6650[0x2f - count] != special[count])
          return false;
      }
      
      um6650[0x09] = 0xff;
      uint8_t tmp = um6650[0x0c];  // ? 
      um6650[0x0c] = 0x00;
      
      return true;
    }
