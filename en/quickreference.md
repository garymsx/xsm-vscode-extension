# XSM Quick Reference

## Link
- [Memory Map](memorymap.md)
- [from Function](from.md)
- [xsmconfig](xsmconfig.md)
- [imports](imports)

## Features
- Allows machine code to be written in a format similar to C or JavaScript
- Implements non-existent instructions by combining multiple instructions (e.g. SUB HL, BC, SLA HL)
- Designed for ROM usage, so it does not support self-modifying code for optimization
- Variables are statically allocated (recursion must be handled manually)

## Literals

- Numbers  
  ```
  0b11001000  // Binary
  0x1210      // Quaternary
  0o144       // Octal
  100         // Decimal
  0x64        // Hexadecimal
  ```

- Characters  
  ```
  'A'
  '\n'   // \n represents the newline character (LF) as an escape sequence.
  ```  

- Strings  
  ```
  "ABC"  // The last character of the string is set to the NULL character (\0).
  ```

- Verbatim Strings  
  Verbatim strings can be written by prefixing the string with @, which prevents escape sequences from being processed.
  ```
  @"Hello World\n"  // \n will not be converted to a newline.
  ```
- Non-NULL Terminated Strings  
  Non-NULL terminated strings can be written by enclosing the string in \`～\`. The NULL character (\0) is not appended to the string.
  ```
  `ABC`
  @`ABC`
  ```

- Escape Sequences  
  ```
  \"  - Inserts a " character
  \a  - Bell
  \b  - Backspace
  \t  - Tab
  \n  - Newline
  \v  - Vertical tab
  \f  - Form feed
  \r  - Carriage return
  \0  - NULL character (0)
  \x  - Allows direct input of a character code using the following 16-bit hexadecimal values: 00 to ff
  \e  - Allows sending control codes to the screen using the following characters after \e
  \\  - Backslash
  ```
  *Note: Escape sequences are only valid when using MSX-DOS system calls.*

## Comments

- Line Comment  
  ```
  A = 10; // This is a comment
  ```

- Block Comment  
  ```
  /*
      This is a comment
  */
  ```

## Data
- Registers  
  Registers can be written in uppercase or lowercase. By default, signed values are not used.

  |Register Type| Notation |
  |---|---|
  |8-bit Register| ```A B C D E H L IXH IXL IYH IYL IT※1 RF※1``` |
  |16-bit Register| ```AF BC DE HL IX IY SP``` |
  |Flag Representation|```$C $NC $Z $NZ $PE $PO $P $M```|
  |Memory Reference Notation| ``` *HL IX[n]```|
  |Signed Representation| ```(singed)A (singed)HL```|

  ※1 IT - I Register, RF - R Register.


- Variables  

  |Type|Description|
  |---|---|
  |byte|Unsigned 1-byte numeric type|
  |sbyte|Signed 1-byte numeric type|
  |word|Unsigned 2-byte numeric type|
  |sword|Signed 2-byte numeric type|
  |char|Character type (1 byte)|
  |string|String reference type (2 bytes)|
  
    - Arrays  
    ```
    byte[2] foo;
    byte[2][4] foo;
    ```

    - Initial Values  
  
    Initial values can be set during program loading. Note that variables are not initialized every time like in other languages.
    ```
    char foo;
    byte foo = 1;
    word[2] foo = {1,2};
    byte[2][4] foo = {{1,2,3,4},{5,6,7,8}};
    byte[] foo = {1,2};
    char[] foo = "ABCDEFG";            // 8 bytes: 7 characters + NULL character 
    char[][] foo = {"ABCDEFG", "ABC"}; // 16 bytes: 2 x 8 bytes, aligned to the length of the longest string
    string foo = "ABCDEFG";
    ```

- Constants  
    Constants do not have a specific type. They are replaced with their values when used in expressions.
    ```
    const foo = 0x0001;
    const foo = "ABC";
    ```

- Constants/Variable Constants  
    Constants do not have a specific type. They are replaced with their values when used in expressions.  
    ```
    const foo = 0x0001;
    const foo = "ABC";
    var   bar = 0;
    bar = bar + 1;   // The value can be changed and calculated
    ```

- Virtual Type  
    Virtual types are used to map variables outside of the built file.  
    For example, if a program is 200 bytes and defines a 1000-byte variable in the workspace, the built file will be 1200 bytes.  
    By using the virtual type, variables can be mapped outside of the program.  
    It is also possible to directly specify the mapping address.  

    ```
    virtual word foo;              // Virtual types cannot have initial values
    virtual char[256] foo;
    virtual(0xd000) char[256] foo; // It is also possible to directly map to memory
    ```

    (TODO) The following statement can be used to initialize virtual data in bulk. Address-specified virtual types are not affected.
    ```
    clear _virtual, 0, _virtualSize;
    ```

## Structures

- Definition  
    ```
    struct FOO {
        byte bar1;
        word  bar2;
    } 
    ```
- Declaration and Initialization  
    ```
    FOO foo = {1,2};
    FOO[] foo = {
          {1,2}
        , {3,4}
    };
    ```

## Expressions

- Assignment (=)  

  Assignments are replaced with the LD instruction, so complex expressions cannot be written.  
  XSM's assignment expressions support many patterns of assignments that are not supported by the Z80.  
  ```
  A = 1;            // Assign to register
  A = 5 + 2 * 8;    // Calculations can be written if the value is determined at assembly time
  *HL = 1;          // Assign to the memory pointed by HL
  foo = A;          // Assign directly to memory
  A = foo.bar1;     // Reference a structure
  A = foo[2];       // Arrays can be written if the address is determined at assembly time
  HL = &foo;        // Get the address of foo
  A = 00;           // A = 0 is equivalent to XOR A
  A = IX[0];        // Index registers can be treated like arrays
  HL = BC;          // Expands to LD H,B; LD L,C
  HL = IX[0];       // Expands to LD HL,(IX+1); LD L, (IX+0)
  A = (FOO)IX.bar1; // If the structure is within 256 bytes, IX can be cast to the structure to access it dynamically
  ```

- Compound Assignment (+= -= *= /= %=)  
  Calculations can be performed using these combinations.
  ```
  A += 2;
  A:$C += 2;   // With carry (ADC)
  A -= 3;
  A:$C -= 3;   // With carry (SBC)
  HL -= BC;    // Expands to OR A; SBC HL,BC
  HL *= A;     // Multiplication, division, and modulo are supported by subroutines
  HL /= BC;
  HL %= D;
  ```

- Increment, Decrement (++ --)  
  ```
  A++;
  HL--;
  ```  

- Shift, Rotate (>> << >>> <<<)  
  ```
  A<<4;          // Repeated 4 times      SLA
  (signed)A>>;   // Signed shift         SRA
  A>>>;          // Rotate               RRC
  HL:$C>>>;      // Rotate with carry    RR
  ```

- Logical Operation (&= ^= |=)  
  ```
  A &= 0b00001111;  // AND
  A ^= 0b00001111;  // XOR
  A |= 0b00001111;  // OR
  ```  

- Function Call  
  ```
  // Definition
  function foo(B, C, D = 3) ...
  function bar(HL) ...

  A = foo(B,2);             // B = B is omitted, C = 2; D = 3 is set.
  A = bar("Hello World\n"); // The address of the string is passed
  ```

## Function Definition

- Format  
  ```
  // :Register specifies the register to be used for the return value, and Dispose specifies the registers that will be destroyed by the function call.
  // If Dispose is defined, a warning will be issued if the registers used after the function call are not initialized.
  function foo(BC, string bar1): A dispose DE {
    ...
  }
  
  // Default Value
  function foo(C = 0x05); // Specifies the value when the parameter is omitted

  // BIOS and resident routines can be defined as follows:
  function(0xe000) foo(C); // Specifies the address
  ```

## Control Statements

- if  

  Conditional branching
  ```
  if(A == 0) {...}
  if(A != 0) {...} else {...}
  if(C >= 10 && C <= 20) ...                   // Possible within unsafe block
  if((signed)A >= -10 && (signed)A <= 10) ...  // Use (signed) to compare signed values
  if(HL >= 10 && HL <= 20) ...                 // Overhead is added because it uses PUSH/SBC to implement CP HL,xx.
  if($C) ...    // Carry flag
  if($NZ) ...   // Negation of zero flag
  ```  

  ```
  // Conditional compilation is possible by combining with const values.
  const foo = 1;
  if(foo == 1) {        // The if statement is not generated because the result is determined at build time
    // This block is built
  } else {
    // This block is not built
  }
  ```  

- for  

  Loop processing
  ```
  for(A = 0; A < 100; A++) {
    …
  }
  ```

- while  

  Loop processing
  ```
  while(A < 10) {
    A++;
  }
  
  // An infinite loop is created if the expression is omitted.
  while() {
  }
  ```

- do  

  Loop processing (condition is checked at the end of the loop)
  ```
  do {
    A++;
  } while(A < 10)
  ```

- loop  

  Loop processing using DJNZ. There are some limitations, but it is the most compact way to implement loop processing.
  ```
  loop(10) { // B register is implicitly used
    ...      // If the loop body exceeds 126 bytes, it will be replaced with DEC B and JP instructions.
  }
  // Even if the B register already has a value, please write it like this. LD B,B will be omitted.
  loop(B)
  ```

- switch  

  Branches based on the value of the A register.
  ```
  // A register is used for case evaluation. For example, switch(B) will execute LD A,B in advance.
  switch(A) { 
    case(0) {
      ...        // No need for break like in C switch statements. The next case is not evaluated.
    }
    case(1,2) {  // Multiple conditions can be written
      ...
    }
    default {
      ...
    }
  }
  ```

- continue / break  

  Continues or exits the loop.
  ```
  while(A < 10) {
    if( xxxx ) continue; // Continue from the beginning of the loop
    if( xxxx ) break;    // Exit the loop
  }
  ```

- on ～ goto / call / return  

  Executes a jump instruction if the flag is true.
  ```
  on $Z  goto foolabel;  // Jump instruction
  on $C  call foolabel;  // Subroutine call
  on $NZ return;          // Return
  ```

- using  

  Provides safe PUSH and POP operations for registers.
  Automatically executes POP when exiting the using block.
  ```
  using(HL) {         // PUSH HL
    ...
    using(BC,DE) {    // PUSH BC; PUSH DE;
      ...
      if(...) return; // POP DE; POP BC; POP HL; when the condition is met
    }                 // POP DE;POP BC;
    ...
  }                   // POP HL
  ```

- try  

  Exception handling. However, exceptions cannot be thrown across functions.
  ```
  try {
    if(...) throw 1;  // A = 1 is executed
    return;
  } catch(A) {
    ...               // Executed when throw is called
  } finally {
    ...               // Always executed when exiting the try block
  }
  ```

- @ Block  

  Anonymous block. Allows the use of continue and break within the block.
  ```
  @ {
      if(xxx) break;    // Exit condition
      continue;         // Loop
  }
  ```

## let (Not Implemented)

  The let statement allows the execution of a calculation expression and obtaining the result.  
  This instruction does not use the B/C/IX registers for calculations and does not save the used registers, so please be careful.  
  ```
  // Normal expression
  B = 1;
  C = 5;
  byte x = 10;
  let A = B + C * x;   // A = 51;

  // Address acquisition
  B = 1;
  byte[2][2] data = {{0,1},{2,3}};
  let HL = &data[B][0]; // Get the address of data[1][0]; 16-bit values cannot be used as indices if the array size is within 256 bytes.
  A = *HL;              // A = 2;

  // Indirect reference
  byte data2 = 123;
  word ptr = &data2;
  let A = *ptr;         // A = 123;

  // Indirect reference for 16-bit values
  word data3 = 1234;
  word ptr = &data3;
  let HL = *ptr;        // HL = 1234;

  // Structure casting
  // Casting is only possible in the format ((struct)register).
  let A = ((FOO)BC).bar;

  ```

  The let statement determines whether to use an 8-bit or 16-bit expression based on the assignment destination.  
  The used registers may not be saved due to optimization, but please use them as a guide to the registers that will be destroyed.  
  | Expression  | Result | Intermediate Result | Variable Reference | Index      |
  |-----|---------|---------|---------|-------------|
  |8-bit | A       | D       | HL      |E            |
  |16-bit| HL      | DE      | IY      |DE           |

## Internal Functions

- Data Definition Functions (bin/qtr/hex)  
  ```
  byte[] foo  = bin("00000000"         // Binary notation
                   + "11,11,11,11");    // Non-numeric characters are ignored
  byte[] foo = qtr("0123,0123");       // Quaternary notation
  byte[] foo = hex("FF,FF");           // Hexadecimal notation
  ```

- File Import Function (from)  
  Imports the image of a file and uses it as the initial value of a variable.
  It is also possible to execute a converter to convert the image into a format that is easier to handle in the program.
  Please refer to [About the from Function](from.md) for more details.
  ```
  byte[] foo = from("data.bin");           // Stores the file as is in foo
  byte[] foo = from("tile.bmp", "sc1");    // Converts a BMP (Indexed Color) image to the SCREEN1 VRAM format
  ```

- Separation Functions (high/low)  
  ```
  word foo = 0x1234;
  A = high(foo);     // A is assigned 0x12
  A = low(foo);      // A is assigned 0x23
  ```

- Size Acquisition Function (sizeof)  
  ```
  word[10] foo;
  A = sizeof(foo);  // A is assigned 20
  ```

- Element Count Acquisition Function (length)  
  ```
  word[10] foo;
  A = length(foo);  // A is assigned 10
  ```

- Offset Acquisition Function (offset)  
  Obtains the offset value from the beginning of a structure member. This is necessary when using an array in a structure.

  ```
  // Example: Obtaining the address of foo[1].foo2
  struct FOO {
    byte  bar1;
    byte  bar2;
  };
  FOO[2] foo;
  HL = &foo;                          // Get the address of foo[0]
  unsafe(BC) HL += sizeof(foo);       // Get the address of foo[1]
  unsafe(BC) HL += offset(foo.bar2);  // Get the address of foo[1].bar2
  ```

- Half-Width Conversion Function (half)  
  ```
  string foo = half("あいうえお"); // Converts to MSX's hiragana ASCII codes.
  ```

- Bit Check Functions (set/res)  
  ```
  // Functions that can be used in if statements.
  if(set(A,0)) { ... } // Condition is true if bit 0 of A is ON.
  if(res(B,7)) { ... } // Condition is true if bit 7 of B is OFF.
  ```

- Get Type Name  
  ```
  // Gets the type name as a string. Can be used for inline function conditionals.
  inline xxx(param) {
      if(typename(param) == "A")      info("It is a register");  // Register
      if(typename(param) == "string") info("It is a character"); 
      if(typename(param) == "number") info("It is a number");     // Literal number
      if(typename(param) == "byte")   info("It is a byte type");   // Variable
      if(typename(param) == "word")   info("It is a word type");   // Variable
  } 
  ```

- Character Code Conversion  
  ```
  // Changes the character code mapping.
  charmap('A',0xa1);
  charmap('B',0xa2);
  …
  charmap('Y',0xa6);
  charmap('Z',0xba);

  char[] foo = "AB";  // -> 0xa1,0xa2 is output.
  ```

- Message Functions (error/warn/info)  

  ```
  Functions that output logs. Can be used for inline function errors.
  error("message"); 
  warn("message");
  info("message"); 
  ```

## Other Instructions

These instructions can be used like regular mnemonics, but some formats have been changed or added.  
Only the instructions that have been added or modified are described in detail.

- dvr
  Performs division and remainder calculations at the same time. Only the following combinations are supported.
  ```
  dvr A,D;        // Quotient is stored in the B register, remainder is stored in the A register.
  dvr HL,D;       // Quotient is stored in the BC register, remainder is stored in the HL register.
  dvr HL,DE;      // Quotient is stored in the BC register, remainder is stored in the HL register.
  ```

- push  
  ```
  push HL;
  push BC,DE,HL;  // Multiple registers can be pushed together
  ```

- pop  
  Pops the registers in reverse order of the push instruction. This is done to maintain the order of the push and pop instructions.
  ```
  pop HL;
  pop BC,DE,HL;  // pop HL; pop DE; pop BC;
  ```

- ldi / ldir / ldd / lddr  
- cpi / cpir / cpd / cpdr  
- nop  
- halt  
- di  
- ei  
- im  
- ex / exx  
  ```
  ex AF,AF;
  ex DE,HL;
  exx;
  ```
  
- daa  
- cpl  
  ```
  cpl A;
  ```
  
- neg  
  ```
  neg A;
  ```
  
- ccf  
- scf  
- rst  
- in  
- ini / inir / ind / indr  
- out  
- outi / otir / outd / otdr  
- bit  
  ```
  bit A,7;
  bit 7,A; // Both have the same meaning.
  ```
  
- set  
  ```
  set A,7;
  set 7,A; // Both have the same meaning.
  ```
  
- res  
  ```
  res A,7;
  res 7,A; // Both have the same meaning.
  ```

- move  
  ```
  byte[5] buf = {0,1,2,3,4};
  byte[5] buf2;
  move buf2, buf;    // Copy buf to buf2, the following registers are used:
                     // DE - Destination
                     // HL - Source
                     // BC - Size
  move buf2, buf, 5; // Size can also be specified
  ```
  This instruction internally executes LDIR. BC/DE/HL registers are destroyed.
  
- clear  
  ```
  byte[5] buf = {0,1,2,3,4};
  clear buf, 0;
  clear buf, 0, 5; // Size can also be specified
  ```
  This instruction is an initialization instruction based on LDIR. BC/DE/HL registers are destroyed.

## System Constants, System Labels

- __virtual / __virtual_size
  Returns the starting address and size of the virtual data area.
  ```
  clear __virtual, 0, __virtual_size; // Bulk initialization of virtual types
  ```

- __heap  
  ```
  HL = __heap;   // Points to the address after the end of the program. Memory can be freely used from this address onwards.
  ```

## Preprocessor
Strictly speaking, it is not a preprocessor, but for convenience, we call it a preprocessor.

- module  
  By default, the output file is created as a .com file, but you can use the "module" keyword to specify a different name.  
  If a size is specified, the file will be padded with zeros until it reaches that size.

  ```
  module foo.bin;
  module foo.rom, 0x4000; // Create a ROM file with a size of 4KB.
  ```

- org  
  Specifies the address where the program will be located. The default is 0x0100.  
  Only one declaration is allowed per program.  
  ```
  org 0x100;            
  org 0x4000, 0x8000;   // The second parameter specifies the address where virtual variables will be located.
  // For example, when creating a ROM file, variables cannot be placed in the same page, so a different address is specified.
  ```

- import  
  References an external file and adds it to the calling module.
  ```
  import "foo.xsm";
  import foo from "foo.xsm";           // Named import. The imported module can be accessed using foo.name.
  import "foo.xsm", 0x8000, virtual;   // Treats the program as if it is loaded at the specified address. No binary is generated at build time.
  import foo from "foo.xsm", 0x8000;   // Loads the program with the specified address. The program needs to be transferred to that address.
  import "foo.xsm", 0x4000, 0x10000;   // The second address specifies the placement address.
  // Transfer is done using the move instruction.
  move 0x8000, foo;
  ```
  The following three are predefined imports.
  ```
  import "xsm/msx/bios.xsm";       // BIOS
  import "xsm/msx/extbios.xsm";    // Extended BIOS
  import "xsm/msx/workarea.xsm";   // BASIC and BIOS work areas
  ```

- include  
  Includes an external file at the current position.
  If import is sufficient, please use import.
  ```
  include "foo.xsm";
  ```

## Annotations
Annotations provide optimization hints and control over the compiler.

- @jp/@jr  
  Forces the use of either jp or jr for internally generated jump instructions in if statements, for statements, etc.
  ```
  @jp
  if(A == 0) {
    ...
  }

  @jr
  for(A = 0; A < 10; A++) {
    ...
  }
  ```

- @ignorewarn  
  Suppresses warning messages.

  ```
  unsafe *BC = D;
  @ignorewarn
  if(A == 0) { // A warning is issued because A is destroyed by *BC = D, but it is suppressed.
    ...
  }

  