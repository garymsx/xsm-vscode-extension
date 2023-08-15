# from function

The from function not only simply imports files, but also converts them into a format that is easier to handle in the program.

## Format

byte[] foo = from(file name [, "converter name"][, "option"]);

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

- Option
  You can pass options when creating a custom converter.

## About Converters

## sc1
Converts BMP data to MSX SCREEN1 pattern generator table and color table format data.
The color table data corresponding to the pattern is concatenated after the pattern data.
For example, in the case of a 256x256 BMP:
- 256 patterns can be defined
- Pattern data is 8 bytes * 256 = 2048 bytes
- If 256 patterns are defined, the color table used is 32 bytes
- The color table starts from the 2048th byte and is 32 bytes long

### Option
  Same RLE/PRLE as the pack converter can be specified.

## sc2
Converts BMP data to MSX SCREEN2 pattern generator table and color table format data.
The pattern generator table is the same as sc1, but the color table can be set for each pattern and each line.
For example, in the case of a 256x256 BMP:
- 256 patterns can be defined
- Pattern data is the same as sc1
- The color table used is 8 bytes per pattern, so 8 * 256 = 2048 bytes
- The color table starts from the 2048th byte and is 2048 bytes long

## sp1x16
Converts BMP data to MSX Sprite Mode 1 pattern table data.
It defines 16x16 sprites by combining four sets of continuous 8x8 size data (8 bytes) as one set.
After the sprite data ends, the color code corresponding to the sprite pattern number is stored.

### Option
  Same RLE/PRLE as the pack converter can be specified.

## pack
Reads the file as a normal binary and compresses it.
Specify the compression method as an option.
Please refer to the Wiki or other sources for the mechanism of RLE.

### Option
  - RLE
    Compresses using RLE.
    Here, the data is stored as follows:
    |Address| Meaning                                      |
    |------:|:---------------------------------------------|
    | +0    | Length of consecutive data, 0 to 255.         |
    | +1    | The data to be repeated is specified here.    |
  - PRLE
    Compresses using PackBits RLE.
    Here, the data is stored as follows:
    |Address | Meaning                                                                 |
    |-------:|:------------------------------------------------------------------------|
    | +0     | Data length, 0 to 127 represents consecutive data, -1 to -128 represents discontinuous data. |
    | +1 to +n | For consecutive data, the data to be repeated is specified.<br/>For discontinuous data, the data of the length is stored. |

## Custom Converter
You can define your own converter.
When you execute `xsm converter` from the command palette, a template for a custom converter will be generated.