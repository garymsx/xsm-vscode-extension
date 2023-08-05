# XSM Language Support

Provides syntax highlighting and assembler for the Z80 structured assembly language XSM.

## All Machine Code
What a fancy term.

## Features
- Can express machine code using syntax similar to C or JavaScript.
- Implements non-existent instructions by combining multiple instructions (e.g. SUB HL, BC, SLA HL, etc.).
- Generates code that can be ROM-based.
- Variables are statically allocated (recursion must be handled manually).

## The Most Tedious "Hello, World"
Although it is verbose and unoptimized, this "Hello, World" program can help you learn the syntax of XSM.
```
org 0x100;
import MSXDOS from "xsm/msx/msxdos.xsm";

char[] hello = "Hello, World"; // null-terminated string

print(&hello);
return;

function print(DE) dispose DE {
    using(AF,BC,HL,IX,IY) {                  // push AF,BC,HL,IX,IY
        // loop_start:
        for(A = 0; A < sizeof(hello); A++) {
            using(AF,DE) {                   // push AF; push DE;
                unsafe E = *DE;              // E = A = *DE;
                // null char then exit
                if(E == 0) {                 // inc E; dec E; // set Z flag
                    break;                   // pop DE; pop AF; jr loop_end;
                }            
                // put char to console       // function(0x0005) _CONOUT(e, c = 0x02);
                                             // assignment to E is omitted
                MSXDOS._CONOUT(E);           // C = 0x02; call 0x0005;
            }                                // pop DE; pop AF;
            DE++;
        }                                    // A++;jr loop_start;
        // loop_end:
    }
}
// 1-line version
// MSXDOS._STROUT("Hello, World$"); // $ is null-terminated char
```

### Getting Started
1. Open the command palette and run `xsm init`.  
   This will create the xsmconfig.json file.

2. Create a .xsm file and write your program in it.

3. Open the command palette and run `xsm build`.  
   The .xsm file will be assembled and a .com file will be generated.

4. Transfer and execute the generated .com file on platforms like MSX-DOS or CP/M.

### Reference Manual, etc.
- [xsm-vscode-extension](https://github.com/garymsx/xsm-vscode-extension)