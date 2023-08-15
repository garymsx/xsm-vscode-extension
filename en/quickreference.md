# XSM Quick Reference

## Related Links
[Memory Map](memorymap.md) [from Function](from.md) [xsmconfig](xsmconfig.md)

## Features
- Can express machine code like C or JavaScript
- Implements non-existent instructions using a combination of multiple instructions (e.g. SUB HL, BC, SLA HL, etc.)
- Does not optimize for self-modification as it is intended to be ROM-able
- Variables are statically allocated (recursion must be manually handled)

## Literals

- Numbers
  ```
  0b11001000  // Binary
  0x1210      // Hexadecimal
  0o144       // Octal
  100         // Decimal
  0x64        // Hexadecimal
  ```

- Characters
  ```
  'A'
  '\n'   // \n is an escape sequence representing a line feed code (LF).
  ```

- Strings
  ```
  "ABC"  // The last character of a string is automatically set to a NULL character (\0).
  ```

- Verbatim Strings
  Verbatim strings can be written by adding @, which means that escape characters are not processed.
  ```
  @"Hello World\n"  // \n is not replaced with a line break.
  ```

- Non-NULL Terminated Strings
  By surrounding with \`～\`, you can write strings without a terminating NULL character (\0).
  ```
  `ABC`
  @`ABC`
  ```

- Escape Sequences
  ```
  \"  - " character
  \a  - bell
  \b  - backspace
  \t  - tab
  \n  - line feed
  \v  - vertical tab
  \f  - form feed
  \r  - carriage return
  \0  - NULL character (0)
  \x  - You can enter the character code directly using 00-ff following \x.
  \e  - You can send control codes to the screen following \e.
  \\  - backslash
  ```
  Note: Escape sequences are only limited to the use of MSX-DOS system calls, etc.

## Comments

- Line Comment
  ```
  A = 10; // Comment
  ```

- Block Comment
  ```
  /*
      Comment
  */
  ```

## Data
- Registers
  Registers are written in uppercase or lowercase. By default, they are treated as unsigned values and do not include a sign.

  |Register Type| Syntax |
  |---|---|
  |8-bit Register| ```A B C D E H L IXH IXL IYH IYL IT※1 RF※1``` |
  |16-bit Register| ```AF BC DE HL IX IY SP``` |
  |Flag Representation|```$C $NC $Z $NZ $PE $PO $P $M```|
  |Memory Reference Syntax| ``` *HL IX[n]```|
  |Signed Representation| ```(singed)A (singed)HL```|

  ※1 IT - I Register, RF - R Register.


- Variables
  |Variable Type|Description|
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
  You can set initial values at program loading. Note that unlike other languages, they are not initialized every time.
  ```
  char foo;
  byte foo = 1;
  word[2] foo = {1,2};
  byte[2][4] foo = {{1,2,3,4},{5,6,7,8}};
  byte[] foo = {1,2};
  char[] foo = "ABCDEFG";            // 8 bytes (7 characters + NULL character)
  char[][] foo = {"ABCDEFG", "ABC"}; // 16 bytes (length is based on the longest string)
  string foo = "ABCDEFG";
  ```

- Constants
  There are no types for constants. They are replaced with the expression where they are used.
  ```
  const foo = 0x0001;
  const foo = "ABC";
  ```

- Constants/Variable Constants
  There are no types for constants. They are replaced with the expression where they are used.
  ```
  const foo = 0x0001;
  const foo = "ABC";
  var   bar = 0;
  bar = bar + 1;   // The value can be changed or calculated.
  ```

- Virtual Type
  It is a mechanism for not storing data in the built file.  
  For example, if a program has a variable defined as 1000 bytes in a workspace with a size of 200 bytes,  
  the built file would be 1200 bytes.  
  By using the virtual type, variables can be mapped outside the program.  
  It is also possible to specify the address of the mapping destination directly.

  ```
  virtual word foo;              // The virtual type cannot be initialized.
  virtual char[256] foo;
  virtual(0xd000) char[256] foo; // It is also possible to directly map to memory.
  ```

  (TODO)You can initialize virtual data all at once with the following code. Address-specified virtual types are not included.
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

- Assignment Expression(=)

  It is replaced with the LD instruction, so you cannot write complex expressions.  
  In the XSM assignment expression, many patterns of assignment expressions that are unofficially supported by the Z80 are pseudo-supported.
  ```
  A = 1;            // Assignment to a register
  A = 5 + 2 * 8;    // Calculations can be written if the value is determined at assembly time
  *HL = 1;          // Assignment to the memory pointed by HL
  foo = A;          // Direct assignment to the memory
  A = foo.bar1;     // Reference to a structure
  A = foo[2];       // Arrays can be written if the address is determined at assembly time
  HL = &foo;        // Get the address of foo
  A = 00;           // When A = 00 is written instead of A = 0, XOR A is executed
  A = IX[0];        // Index registers can be treated like arrays
  HL = BC;          // Expands to LD H,B; LD L,C
  HL = IX[0];       // Expands to LD HL,(IX+1); LD L, (IX+0)
  A = (FOO)IX.bar1; // For structures up to 256 bytes, you can dynamically access IX by casting it to a structure.  
  ```

- Arithmetic Assignment Expression(+= -= *= /= %=)
  Complex calculations can be performed using these combinations.
  ```
  A += 2;
  A:$C += 2;   // with Carry (ADC) 
  A -= 3;
  A:$C -= 3;   // with Carry (SBC)
  HL -= BC;    // Expands to OR A; SBC HL,BC
  HL *= A;     // Multiplication, division, and modulo operations are supported by subroutines
  HL /= BC;
  HL %= D;
  ```

- Increment, Decrement(++ --)
  ```
  A++;
  HL--;
  ```

- Shift, Rotate(>> << >>> <<<)
  ```
  A<<4;          // Repeated 4 times      SLA
  (signed)A>>;   // Arithmetic Shift Right (SRA)
  A>>>;          // Rotate Right         (RRC)
  HL:$C>>>;      // with Carry           (RR)
  ```

- Logical Operation(&= ^= |=)
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

  A = foo(B,2);             // B = B is omitted and C = 2; D = 3 is set.
  A = bar("Hello World\n"); // The address of the string is passed.
  ```

## Function Definition
- Format
  ```
  // :register defines the register used for return value. dispose defines the register used for call damage.
  // If dispose is defined, a warning will be shown if you use the register without initializing it after the function call.
  function foo(BC, string bar1): A dispose DE {
    ...
  }
  
  // Default Value
  function foo(C = 0x05); // Specifies the value when the parameter is omitted

  // BIOS or resident processes can be defined as follows:
  function foo(C) = 0xe000; // Specifies the address
  ```

## Labels and Jumps

- Label
  ```
  foo:
  ```

- Jump
  ```
  goto foo;
  ```

- Subroutine Call
  ```
  call foo;
  ```

- Return from Function or Subroutine
  ```
  // No return value or omitted
  return;
  // If a return value is set in the function, it has arguments.
  return 1;
  // If the function is defined as function foo(): A, return 1 is expanded as follows:
  LD A,1
  RET
  // If return A; is written, LD A,A is omitted.
  ```

- returni / returnn
  ```
  // Return for interrupt
  returni;
  returnn;
  ```

# Pseudo-Instructions

- unsafe
  Allows implicit use of A / HL registers.
  In the unsafe block, assignment expressions, arithmetic assignment expressions, conditional expressions, etc. are greatly expanded.
  Complex expressions can be written using A / HL registers, and expressions that can be achieved via assignment are omitted.
  Naturally, A / HL registers are destroyed.
  ```
  unsafe {
    *DE = B;
  }
  // This is expanded as follows. The original value of the A register is destroyed.
  LD A,B
  LD (DE),A
  
  unsafe {
    BC += DE;
  }
  // This is expanded as follows. The original values of the HL and DE registers are destroyed.
  LD H,B
  LD L,C
  ADD HL,DE
  LD B,H
  LD C,L

  // Furthermore, if a parameter BC (or DE) is given to unsafe, BC register is implicitly used.
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
  if(C >= 10 && C <= 20) ...                   // possible in an unsafe block
  if((signed)A >= -10 && (signed)A <= 10) ...  // If judging with signed, use (signed)
  if(HL >= 10 && HL <= 20) ...                 // This has overhead because it utilizes PUSH/SBC/etc. to achieve CP HL, xx.
  if($C) ...    // Carry flag
  if($NZ) ...   // Negate zero flag
  ```

  ```
  // Also, with the combination of const value, conditional compilation-like behavior is possible.
  const foo = 1;
  if(foo == 1) {        // No if statement is generated because the result is determined at compile time
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
  
  // Omission of expression leads to infinite loop.
  while() {
  }
  ```

- do
  Loop processing (conditional is checked at the end of the loop)
  ```
  do {
    A++;
  } while(A < 10)
  ```

- loop
  Loop processing using DJNZ. There are limitations, but it is the most compact in looping processing.
  ```
  loop(10) { // B register is implicitly used
    …      // if the size of the block exceeds 126 bytes, DEC B and JP instructions will be used
  }
  // Even if B register already has a value, please write as follows. LD B,B is omitted.
  loop(B)
  ```

- switch
  Switch branching based on the value of the A register.
  ```
  // Switch branching is done based on the A register. For example, switch(B) will execute LD A,B before branching.
  switch(A) { 
    case(0) {
      ...        // No need for break like in C. The next case will not be judged.
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
  Continue or exit a loop.
  ```
  while(A < 10) {
    if( xxxx ) continue; // Loop from the beginning
    if( xxxx ) break;    // Break the loop
  }
  ```

- on ～ goto / call / return
  Execute a jump instruction if the contents of the flag are valid.
  ```
  on $Z  goto foolabel;  // Jump instruction
  on $C  call foolabel;  // Subroutine call
  on $NZ return;          // Return
  ```

- using
  Provides safe PUSH and POP operations for registers.
  POP is automatically executed when exiting the using block.
  ```
  using(HL) {         // PUSH HL
    ...
    using(BC,DE) {    // PUSH BC; PUSH DE;
      ...
      if(...) return; // When condition is met, POP DE; POP BC; POP HL;
    }                 // POP DE; POP BC;
    ...
  }                   // POP HL
  ```

- try
  Exception handling. However, it cannot throw exceptions across functions.
  ```
  try {
    if(...) throw 1;  // A = 1 will be executed
    return;
  } catch(A) {
    ...               // Executed when throw is called
  } finally {
    ...               // Executed when exiting try
  }
  ```

- @ block
  Anonymous block. You can use continue and break in the block.
  ```
  @ {
      if(xxx) break;    // Exit condition
      continue;         // Loop
  }
  ```

