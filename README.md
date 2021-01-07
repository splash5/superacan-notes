

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

Booting procedure pseudo code:

    boolean booting_check(void) {
      const uint8_t special[] = {0x9b, 0x65, 0x70, 0x90, 0x29, 0x43, 0x28, 0x20, 0x34, 0x39, 0x39, 0x31, 0x20, 0x43, 0x4d, 0x55};
      const uint8_t *um6650_addr_reg = (uint8_t far *)0xeb0d02;
      const uint8_t *um6650_data_reg = (uint8_t far *)0xeb0d00;
      
      uint8_t count; 
      uint8_t mem[32];

      // backup values inside 6650
      for (count = 0; count < 32; count++) {
      	*um6650_addr_reg = 0x5f - count;
      	mem[count] = *um6650_data_reg;
      }
    
      // filling test pattern
      for (count = 0x5f; count <= 0x40; count--) {
        *um6650_addr_reg = count;
        *um6650_data_reg = count;
      }
    
      // check if previous writing succeed
      for (count = 0x5f; count <= 0x40; count--) {
        *um6650_addr_reg = count;
        
        if (*um6650_data_reg != count)
          return false;      
      }
      
      // restore values back to 6650
      for (count = 0; count < 32; count++) {
        *um6650_addr_reg = 0x5f - count;
        *um6650_data_reg = mem[count];
      }
      
      // check if special string matches
      for (count = 0; count < 16; count++) {
        *um6650_addr_reg = 0x2f - count;
        
        if (*um6650_data_reg != special[count])
          return false;      
      }
      
      // unknown purpose
      *um6650_addr_reg = 0x09;
      *um6650_data_reg = 0xff;
      
      *um6650_addr_reg = 0x0c;
      uint8_t tmp = *um6650_data_reg;	// tmp = 0xff
      *um6650_data_reg = 0x00;
            
      return true;
    }
Based on these info, I built a tiny PCB with a AT28C64B which stores the special string at 0x20 to 0x2f, and filled 0x40 to 0x5f with 0x40 to 0x5f accordingly. And It did pass the checking and the game boots up!

I put EAGLE schematic, board and gerber file in [here](pcb/eagle/UM6650C).If you want to build by yourself, you need to program AT28C64B with [this file](pcb/eagle/UM6650C/UM6650C.bin) first. After programming the AT28C64B, solder it on PCB and close the solder jumper next to C5 capacitor (mark with a tiny arrow). I also shared gerber on [oshpark](https://oshpark.com/shared_projects/0XdS5ExD) so you can order PCB from there.

