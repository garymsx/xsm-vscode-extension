# Standard import files

## File list

- [bios.xsm](./xsm/msx/bios.xsm)

  This file defines the BIOS for the MAIN ROM.  
  It also defines inline functions that can be called directly from DOS through inter-slot calls.

- [extbios.xsm](./xsm/msx/extbios.xsm)

  This file defines the BIOS for the SUB ROM.  
  It can also be called directly from DOS.

- [msxdos.xsm](./xsm/msx/msxdos.xsm)

  This file defines the function calls for MSX-DOS.

- [workarea.xsm](./xsm/msx/workarea.xsm)

  This file defines the work area that is referenced by the BIOS and BASIC.

- [unpackRLE.xsm](./xsm/unpack/unpackRLE.xsm)
- [unpackPRLE.xsm](./xsm/unpack/unpackPRLE.xsm)
- [unpackSRLE.xsm](./xsm/unpack/unpackSRLE.xsm)
- [unpackBPE.xsm](./xsm/unpack/unpackBPE.xsm)

  These files define functions to decompress RLE/PRLE/SRLE/BPE compressed data.