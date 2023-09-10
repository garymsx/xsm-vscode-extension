# from function

The from function not only simply imports files, but also converts them into a format that is easier to handle in the program.

## Format

byte[] foo = from(file name [, "converter name"][, "options"]);

- File name
  The specified file name is loaded and passed to the converter.
  If it is a BMP (Indexed Color) file, the converted image data is passed to the converter.

- Converter name
  Specify the script name. The following converters are available:
  - sc1
    Converts BMP data to MSX SCREEN1 pattern (& color) data.
  - sc2
    Converts BMP data to MSX SCREEN2 pattern (& color) data.
  - sp1x16
    Converts BMP data to MSX Sprite Mode 1, size 16 pattern (& color) data.
  - pack
    Reads the file as a normal binary and compresses it.

- Options
  Specify in the format of variable name = value, variable name2 = value, etc.

## About Converters

## pack
Reads the file as a normal binary and compresses it.
Specify the compression method with options.
Please refer to the Wiki or other sources for the mechanism of RLE.

### Options
  - pack
    Specifies the compression format.

    - RLE
      Performs RLE compression.
      The format is as follows:
      |Item    |Size  | Meaning                                      |
      |:-------|-----:|:---------------------------------------------|
      |Data length| 1  | Length of consecutive data, 0 to 255.        |
      |Value   | 1    | Specifies the data to be repeated.           |
      |        |      | Repeated until the end of the data.          |

    - PRLE
      Performs PackBits RLE compression.
      The format is as follows:
      |Item    |Size    | Meaning                                                                 |
      |:-------|-------:|:------------------------------------------------------------------------|
      |Data length| 1    | Data length, 0 to 127 represents consecutive data, -1 to -128 represents discontinuous data. |
      |Value   | 1 to 128 | For consecutive data, specifies the data to be repeated.<br/>For discontinuous data, the data of the length is stored. |
      |        |        | Repeated until the end of the data.                                     |

    - BPE
      Performs Bit Pair Encoding compression.
      The format is as follows:
      |Item        |Size    | Meaning                                                                 |
      |:-----------|-------:|:------------------------------------------------------------------------|
      |Number of pair dictionaries| 1 | Number of pair dictionaries.                                             |
      |Pair dictionary|        | The following structure is repeated for the number of pair dictionaries. |
      | - Pair target| 1      | Code value to be replaced.                                               |
      | - Pair 1   | 1      | Code value of the first pair.                                            |
      | - Pair 2   | 1      | Code value of the second pair.                                           |
      |Data length | 2      | Length of the data section.                                              |
      |Data        | 1 to n | The data of the length is stored.                                        |
      |            |        | Repeated until the end of the data.                                      |

## sc1
Converts BMP data to MSX SCREEN1 pattern generator table and color table format data.
The color table data corresponding to the pattern is concatenated after the pattern data.
For example, in the case of a 256x64 BMP:
- 256 pattern definitions can be made.
- The pattern data is 8 bytes * 256 = 2048 bytes.
- If pattern is defined, the color table to be used is 32 bytes.
- The color table starts from the 2048th byte and is 32 bytes long.

### Options
- pack
  Specifies the options to be passed to the pack converter.

## sc2
Converts BMP data to MSX SCREEN2 pattern generator table and color table format data.
The pattern generator table is the same as sc1, but the color table can be set for each pattern and each line.
For example, in the case of a 256x128 BMP:
- 512 pattern definitions can be made.
- The pattern data is 8 bytes * 512 = 4096 bytes.
- The color table to be used is 8 bytes per pattern, so 8 * 512 = 4096 bytes.
- The color table starts from the 4096th byte and is 4096 bytes long.

### Options
- pack
  Specifies the options to be passed to the pack converter.

## sp1x16
Converts BMP data to MSX Sprite Mode 1 pattern table data.
Defines 16x16 sprites by combining four sets of continuous 8x8 size data (8 bytes) each.
After the sprite data ends, the color code corresponding to the sprite pattern number is stored.

### Options
- pack
  Specifies the options to be passed to the pack converter.

## Custom Converter
You can define your own converter.
When you execute `xsm converter` from the command palette, a template for a custom converter will be generated.