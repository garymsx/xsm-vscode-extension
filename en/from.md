# from function

The from function not only imports files, but also converts them into a format that is more convenient for the program to handle.

## Format

byte[] foo = from(file name, [, "converter name"][, "options"]);

- File name
  The specified file name is loaded and passed to the converter.
  If the file is a BMP (Indexed Color) file, the converted image data will be passed to the converter.

- Converter name  
  Specify the script name. The following converters are available:
  - sc1  
  Converts BMP data to MSX SCREEN1 pattern (& color) data.
  - sc2  
  Converts BMP data to MSX SCREEN2 pattern (& color) data.
  - pack  
  Reads the file as a normal binary and compresses it.

- Options  
  Custom options can be passed when creating a custom converter.

## About the Converters

## sc1
Converts BMP data to MSX SCREEN1 pattern generator table and color table format data.
The color table data corresponding to the patterns is concatenated after the pattern data.
For example, in the case of a 256x256 BMP:
- You can define 256 patterns.
- The pattern data is 8 bytes * 256 = 2048 bytes.
- If 256 patterns are defined, the color table used is 32 bytes.
- The color table starts from the 2048th byte and consists of 32 bytes.

### Options
The same RLE/PRLE as the pack converter can be specified.

## sc2
Converts BMP data to MSX SCREEN2 pattern generator table and color table format data.
The pattern generator table is the same as sc1, but the color table can be set for each pattern and each line.
For example, in the case of a 256x256 BMP:
- You can define 256 patterns.
- The pattern data is the same as sc1.
- The color table used is 8 bytes per pattern, so 8 * 256 = 2048 bytes.
- The color table starts from the 2048th byte and consists of 2048 bytes.

### Options
The same RLE/PRLE as the pack converter can be specified.

## pack
Reads the file as a normal binary and compresses it.
The compression method is specified as an option.
Please refer to the Wiki or other resources for the RLE mechanism.

### Options
- RLE  
  Compresses using RLE.
  The data is stored as follows:
  |Address| Meaning                          |
  |-------|----------------------------------|
  | +0    | Length of consecutive data (0-255)|
  | +1    | The repeated data is specified.  |
- PRLE  
  Compresses using PackBits RLE.
  The data is stored as follows:
  |Address| Meaning                                                                                     |
  |-------|---------------------------------------------------------------------------------------------|
  | +0    | Data length: 0-127 for consecutive data, -1 to -128 for non-consecutive data.               |
  | +1 to +n| For consecutive data, the repeated data is specified.<br/>For non-consecutive data, the data of length is stored. |

## Custom Converters
You can define your own converters.
Executing `xsm converter` from the command palette will generate a template for a custom converter.