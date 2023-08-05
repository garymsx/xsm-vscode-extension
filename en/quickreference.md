# XSM Quick Reference

## Related Links
[Memory Map](memorymap.md) [from Function](from.md) [xsmconfig](xsmconfig.md)

## Features
- Allows expression of machine code in the form of C or JavaScript
- Implements non-existing instructions by combining multiple instructions (e.g. SUB HL,BC, SLA HL)
- Does not perform self-modification to optimize for ROM usage
- Variables are statically allocated (recursion is not supported)

## Literals

- Numbers  
  ```
  0b11001000  // binary
  0x1210      // quaternary
  0o144       // octal
  100         // decimal
  0x64        // hexadecimal
  ```

- Characters  
  ```
  'A'
  '\n'   // \n is an escape sequence that represents the line feed code (LF).
  ```  

- Strings  
  ```
  "ABC"  // The last character of the string is set as the null character (\0).
  ```

- Verbatim strings  
  Using @ allows you to write strings that are not escaped.
  ```
  @"Hello World\n"  // \n is not converted to a line break.
  ```
- Non-null-terminated strings  
  By surrounding with \`~\`, you can write strings without the null character (\0).
  ```
  `ABC`
  @`ABC`
  ```

- Escape sequences  
  ```
  \"  - Inserts a "
  \a  - Bell
  \b  - Backspace
  \t  - Tab
  \n  - Line feed
  \v  - Vertical tab
  \f  - Form feed
  \r  - Carriage return
  \0  - Null character (0)
  \x  - Allows direct input of a character code by specifying a 16-bit hexadecimal number after \x
  \e  - Allows sending control codes to the screen by specifying a character after \e
  \\  - Backslash symbol
  ```
  * Note that escape sequences are only allowed when using MSX-DOS system calls.

## Comments

- Line comment  
  ```
  A = 10; // comment
  ```

- Block comment  
  ```
  /*
      comment
  */
  ```

## Data
- Registers  
  Can be written in uppercase or lowercase.
  By default, arithmetic is performed without considering sign.

  | Register Type | Syntax |
  |---|---|
  |8-bit registers| ```A B C D E H L IXH IXL IYH IYL IT※1 RF※1``` |
  |16-bit registers| ```AF BC DE HL IX IY SP``` |
  |Flag representation|```$C $NC $Z $NZ $PE $PO $P $M```|
  |Representation of memory reference| ``` *HL IX[n]```|
  |Rpresentation with sign| ```(singed)A (singed)HL```|

  ※1 IT - I register, RF - R register.


- Variables  

  | Data Type | Content |
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

    - Initial values  
  
    You can set initial values at the time of program loading. Please note that it is not initialized every time like other languages.
    ```
    char foo;
    byte foo = 1;
    word[2] foo = {1,2};
    byte[2][4] foo = {{1,2,3,4},{5,6,7,8}};
    byte[] foo = {1,2};
    char[] foo = "ABCDEFG";            // 8 bytes with 7 characters + null character 
    char[][] foo = {"ABCDEFG", "ABC"}; // 16 bytes (2 X 8) aligned to the length of the longest string
    string foo = "ABCDEFG";
    ```

- Constants  
    There is no data type. It is replaced when used in an expression.
    ```
    const foo = 0x0001;
    const foo = "ABC";
    ```

- Constants / Mutable Constants  
    There is no data type. It is replaced when used in an expression.  
    ```
    const foo = 0x0001;
    const foo = "ABC";
    var   bar = 0;
    bar = bar + 1;   // The value and calculation can be changed
    ```

- Virtual types  
    This is a mechanism that does not have data in the built file.  
    For example, if a program defines 1000 bytes of variables in a 200-byte program,
    the built file will be 1200 bytes.  
    You can use the virtual type (virtual) to map variables outside of the program.  
    It is also possible to directly specify the address of the mapping destination.  

    ```
    virtual word foo;              // virtual types cannot have initial values
    virtual char[256] foo;
    virtual(0xd000) char[256] foo; // Can also be mapped directly to memory
    ```

    (TODO)The following description allows you to initialize virtual data in bulk. Addresses of virtual type are not applicable.
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

  Replaced with the mnemonic LD instruction, so complex expressions cannot be written.  
  In the assignment expressions of XSM, many patterns of assignment expressions that are not supported by the Z80 are pseudo-supported.  
  ```
  A = 1;            // Assign to register
  A = 5 + 2 * 8;    // You can write a calculation expression if the value is determined at assembly time
  *HL = 1;          // Assign to the memory pointed to by HL
  foo = A;          // Assign directly to memory
  A = foo.bar1;     // Reference to structure
  A = foo[2];       // If the address is determined at assembly time, you can write an array
  HL = &foo;        // Get the address of foo
  A = 00;           // A = 0 will not execute XOR A
  A = IX[0];        // The index register can be treated like an array
  HL = BC;          // Expand to LD H,B; LD L,C;
  HL = IX[0];       // Expand to LD HL,(IX+1); LD L, (IX+0);
  A = (FOO)IX.bar1; // For structures up to 256 bytes, you can dynamically access it by casting IX to a structure
  ```

- Compound Assignment (+= -= *= /= %=)  
  Use these combinations to perform calculations.
  ```
  A += 2;
  A:$C += 2;   // With carry (ADC) 
  A -= 3;
  A:$C -= 3;   // With carry (SBC)
  HL -= BC;    // Expand to OR A; SBC HL,BC;
  HL *= A;     // Multiplication, division, and modulo division are supported by subroutines
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
  A<<4;          // 4 repetitions      SLA
  (signed)A>>;   // Signed shift       SRA
  A>>>;          // Rotation           RRC
  HL:$C>>>;      // With carry         RR
  ```

- Logical Operators (&= ^= |=)  
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
  A = bar("Hello World\n"); // The address of the string is passed.
  ```

## Function Definition

- Format  
  ```
  // : The register to be used for return is defined as the response, and the register to be broken by the call is defined.
  // If dispose is defined, a warning will be displayed if you try to use the register without initializing it after the function call.
  function foo(BC, string bar1): A dispose DE {
    ...
  }
  
  // Default Values
  function foo(C = 0x05); // Specify the value when the parameter is omitted

  // Define when calling BIOS or resident processing.
  function foo(C) = 0xe000; // Specify the address
  ```

## Labels and Jumps

- Labels  
  ```
  foo:
  ```
- Jumps  
  ```
  goto foo;
  ```

- Subroutine Call  
  ```
  call foo;
  ```

- Return from Function or Subroutine  

  ```
  // If there is no return value or it is omitted
  return;
  // If the function has a return value, it takes an argument.
  return 1;
  // If defined as function foo(): A, return 1 will be expanded as follows.
  LD A,1
  RET
  // When it becomes return A;, LD A,A is omitted
  ```

- returni / returnn  
  ```
  // Return (returni; or returnn;) for interrupt
  returni;
  returnn;
  ```

# Pseudo Instructions

- unsafe   

  Allows the implicit use of A / HL registers.  
  Within the unsafe block, assignment expressions, compound assignment expressions, conditional expressions, etc. are greatly expanded.  
  Complex expressions that are made possible by passing through A / HL registers can be omitted instead of being assigned explicitly.  
  Of course, since A / HL registers will be destroyed, please note that it will not be usable anymore.
  ```
  unsafe {
    *DE = B;
  }
  // This will be expanded as follows, the original value of the A register will be destroyed.
  LD A,B
  LD (DE),A
  
  unsafe {
    BC += DE;
  }
  // This will be expanded as follows, the original value of the HL register will be destroyed.
  LD H,B
  LD L,C
  ADD HL,DE
  LD B,H
  LD C,L

  // Furthermore, if you give an argument BC (or DE) to unsafe, you can implicitly use BC register.
  unsaf(BC) {
    HL += 123;
  }
  LD BC,123
  ADD HL,BC
  ```

## Control Statements

- if  

  Conditional branching
  ```
  if(A == 0) {...}
  if(A != 0) {...} else {...}
  if(C >= 10 && C <= 20) ...                   // Possible in unsafe
  if((signed)A >= -10 && (signed)A <= 10) ...  // Use (signed) when judging with signed value
  if(HL >= 10 && HL <= 20) ...                 // CP HL,xx is realized by using PUSH/SBC, so there is overhead.
  if($C) ...    // Carry flag
  if($NZ) ...   // Inverse of zero flag
  ```  

  ```
  // Also, when used in conjunction with const values, it allows for conditional compilation-like functionality.
  const foo = 1;
  if(foo == 1) {        // IF there is a zero, the result will be determined at build time and no if statement will be generated
    // This block will be built
  } else {
    // This block of code will not be built
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

  Loop processing (check is made at the end of the loop) 
  ```
  do {
    A++;
  } while(A < 10)
  ```

- loop  

  Loop processing using the DJNZ instruction. Although there are limitations, it is the most compact loop processing.
  ```
  loop(10) { // B register is implicitly used.
    ...      // If the processing in the loop is more than 126 bytes, it will be replaced by DEC B and JP instructions.
  }
  // Even if a value is already in the B register, please write as shown here. LD B,B is omitted.
  loop(B)
  ```

- switch  

  Branches according to the value of the A register.
  ```
  // Case is based on the A register. For example, if switch(B) is used, LD A,B is executed in advance.
  switch(A) { 
    case(0) {
      ...        // The case statement does not check the next case.
    }
    case(1,2) {  // Multiple conditions are allowed
      ...
    }
    default {
      ...
    }
  }
  ```

- continue / break  

  Continues or exits a loop.
  ```
  while(A < 10) {
    if( xxxx ) continue; // Skip to the beginning of the loop
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

  Provides a safe PUSH and POP for registers.  
  Automatically performs a POP when exiting the using block.  
  ```
  using(HL) {         // PUSH HL
    ...
    using(BC,DE) {    // PUSH BC; PUSH DE;
      ...
      if(...) return; // POP DE; POP BC; POP HL;
    }                 // POP DE;POP BC;
    ...
  }                   // POP HL
  ```

- try  

  Exception handling. However, exceptions cannot be thrown beyond functions.
  ```
  try {
    if(...) throw 1;  // A = 1 will be executed
    return;
  } catch(A) {
    ...               // Processing when a throw occurs
  } finally {
    ...               // Processing that is always executed when exiting the try block
  }
  ```

- @ Block  

  Anonymous block. You can use continue and break in the block.
  ```
  @ {
      if(xxx) break;    // Exit condition
      continue;         // Continue the loop
  }
  ```

## let(TODO)
When you need to execute a calculation and get the result, you can use the let statement.  
This instruction allows you to execute calculations using registers other than B/C/IX and does not store the used registers.  
```
// Regular expression
B = 1;
C = 5;
byte x = 10;
let A = B + C * x;   // A = 51;

// Get the address
B = 1;
byte[2][2] data = {{0,1},{2,3}};
let HL = &data[B][0]; // Get the address of data[1].[0] 2-dimensional array cannot use 16-bit indices if the size of the array is within 256 bytes.
A = *HL;              // A = 2;

// Indirect reference
byte data2 = 123;
word ptr = &data2;
let A = *ptr;         // A = 123;

// Indirect reference for 16 bits
word data3 = 1234;
word ptr = &data3;
let HL = *ptr;        // HL = 1234;

// Cast structure
// The cast is in the format ((structure name) register) only.
let A = ((FOO)BC).bar;

```

In the let statement, if the destination of the assignment is 8-bit, it is judged as an 8-bit expression, and the used registers change.  
Even if it is not used due to optimization, please take it as a rough guide for the destroyed registers.  
| Expression  | Calculation results | Intermediate results | Variable reference | Index   |
|-----|---------|---------|---------|-------------|
|8bit | A       | D       | HL      |E            |
|16bit| HL      | DE      | IY      |DE           |

## Internal Functions

- Data definition functions (bin/qtr/hex)  
  ```
  byte[] foo  = bin("00000000"         // Binary notation
                   + "11,11,11,11");    // Non-numeric characters are ignored
  byte[] foo = qtr("0123,0123");       // Quaternary notation
  byte[] foo = hex("FF,FF");           // Hexadecimal notation
  ```

- File import functions (from)  
  Imports the image of a file and uses it as an initial value for a variable.
  You can also convert the image to a more convenient format for the program by executing the converter during the import.
  Please refer to [About from Function](from.md) for details.
  ```
  byte[] foo = from("data.bin");           // Store the file as-is in foo
  byte[] foo = from("tile.bmp", "sc1");    // Convert a BMP (Indexed Color) image to SCREEN1 VRAM format
  ```

- Split functions (high/low)  
  ```
  word foo = 0x1234;
  A = high(foo);     // A will be 0x12
  A = low(foo);      // A will be 0x23
  ```

- Size acquisition function (sizeof)  
  ```
  word[10] foo;
  A = sizeof(foo);  // A will be 20
  ```

- Length acquisition function (length)  
  ```
  word[10] foo;
  A = length(foo);  // A will be 10
  ```

- Offset function (offset)  
  Obtains the offset value from the beginning of a structure member. This is a function that becomes necessary when using an array for a structure.  
  ```
  // Example: Getting the address of foo[1].foo2
  struct FOO {
    byte  bar1;
    byte  bar2;
  };
  FOO[2] foo;
  HL = &foo;                          // Get the address of foo[0]
  unsafe(BC) HL += sizeof(foo);       // Get the address of foo[1]
  unsafe(BC) HL += offset(foo.bar2);  // Get the address of foo[1].bar2
  ```

- Character code conversion function (half)  
  ```
  string foo = half("あいうえお"); // Converts to MSX hiragana ASCII code.
  ```

- Bit check functions (set/res)  
  ```
  // Functions that can be used in if statements.
  if(set(A,0)) { ... } // The condition is true if bit 0 of the A register is ON
  if(res(B,7)) { ... } // The condition is true if bit 7 of the B register is OFF
  ```

- Getting the type name  
  ```
  // Gets the type name as a string. Can be used to judge inline functions, etc.
  inline xxx(param) {
      if(typename(param) == "A")      info("This is a register.");  // Register
      if(typename(param) == "string") info("This is a character."); 
      if(typename(param) == "number") info("This is a number.");     // Literal number
      if(typename(param) == "byte")   info("This is a byte variable.");   // Variable
      if(typename(param) == "word")   info("This is a word variable.");   // Variable
  } 
  ```

- Change character codes  
  ```
  // Changes the character code mapping.
  charmap('A',0xa1);
  charmap('B',0xa2);
  …
  charmap('Y',0xa6);
  charmap('Z',0xba);

  char[] foo = "AB";  // 0xa1,0xa2 is output.
  ```

- Message functions (error/warn/info)  

  ```
  Functions that output logs. Can be used for inline function errors, etc.
  error("message"); 
  warn("message");
  info("message"); 
  ```

## Other Instructions

General instructions can be used as usual, but some formats have been changed or added.  
Only those that have been added or changed are described in detail.

- dvr  
  Obtains both divide and remainder. Only the following combinations are supported.
  ```
  dvr A,D;        // The quotient is stored in the B register and the remainder is stored in the A register.
  dvr HL,D;       // The quotient is stored in the BC register and the remainder is stored in the HL register.
  dvr HL,DE;      // The quotient is stored in the BC register and the remainder is stored in the HL register.
  ```

- push  
  ```
  push HL;
  push BC,DE,HL;  // Push multiple at once
  ```

- pop  
  Pops registers in reverse order starting from the end of the written order.  
  This is done to align with the push instruction.
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
  move buf2, buf;    // Copy buf to buf2 using the following:
                     // DE - Destination
                     // HL - Source
                     // BC - Size
  move buf2, buf, 5; // Size can also be specified
  ```
  This is an initialization instruction based on LDIR. BC/DE/HL registers are destroyed.
  
- clear  
  ```
  byte[5] buf = {0,1,2,3,4};
  clear buf, 0;
  clear buf, 0, 5; // Size can also be specified
  ```
  This is an initialization instruction based on LDIR. BC/DE/HL registers are destroyed.

## System Constants, System Labels

- __virtual / __virtual_size
  Gets the start address and size of the virtual data area.
  ```
  clear __virtual, 0, __virtual_size; // Initializes virtual types in bulk
  ```

- __heap  
  ```
  HL = __heap;   // Points to the address of the end of the program +1. You can use memory freely after this address.
  ```

## Preprocessor
Strictly speaking, it is not a preprocessor, but it is called a preprocessor because it is a set of instructions that are not part of the program.

- module  
  By default, the output file is a .com file, but you can specify a different name with module.
  ```
  module foo.bin;
  ```

- org  
  Specifies the address where the program is to be placed. The default is 0x0100.  
  Only one declaration is allowed per program.  
  ```
  org 0x100;            
  org 0x4000, 0x8000;   // The second parameter is the address where virtual type variables are placed.
  // For example, when creating a ROM file, variables cannot be placed on the same page, so you need to specify an address that will be on a different page.
  ```

- import  
  References an external file and adds it to the calling module.
  ```
  import "foo.xsm";
  import "foo.xsm", 0x8000, virtual;  // Treats the program as if it were loaded at the specified address. The binary is not generated at build time.
  import foo from "foo.xsm", 0x8000;  // Read with address specified. The program must be transferred to that address.
  // Transfer is done with the move instruction.
  move 0x8000, foo;
  ```

- include  
  Imports an external file and inserts it there.  
  Please use import if that is enough.
  ```
  include "foo.xsm";
  ```

## Annotations
Used to provide optimization hints or control the compiler.

- @jp/@jr  
  Forces the use of jp or jr for internally generated jump instructions in if statements, for statements, etc.
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
  if(A == 0) { // A warning will be displayed because it destroys A due to *BC = D, but it will not be displayed.
    ...
  }

  ```