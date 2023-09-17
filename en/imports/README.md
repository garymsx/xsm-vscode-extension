# Standard import files

## File list

- [bios.xsm](./xsm/msx/bios.xsm)

  This file defines the BIOS for the MAIN ROM.  
  It also defines inline functions that can be called directly from DOS via inter-slot calls.

- [extbios.xsm](./xsm/msx/extbios.xsm)

  This file defines the BIOS for the SUB ROM.  
  It can also be called directly from DOS.

- [msxdos.xsm](./xsm/msx/msxdos.xsm)

  This file defines the function calls for MSX-DOS.

- [workarea.xsm](./xsm/msx/workarea.xsm)

  This file defines the work area that is referenced by the BIOS and BASIC.