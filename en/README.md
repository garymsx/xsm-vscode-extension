# XSM Language Support (Z80 Assembler)

Provides syntax highlighting and assembler for the XSM structured assembly language for Z80.

## All Machine Code
What a fancy term.

### Reference Manuals, etc.
- [xsm-vscode-extension](https://github.com/garymsx/xsm-vscode-extension)

### Updates
- 0.0.11
  - Changed the format of the options for the `from` converter (\[variable=value, ...], format)
- 0.0.9
  - Added placement address to `import`
  - Added output size to `module`
- 0.0.5
  - Added `build` to xsmconfig.json  
    You can specify the build targets.
    ```
    	"build": [
          "main.xsm", "sub.xsm", ...
    	],
    ```

## Features
- Allows expressing machine code using syntax similar to C or JavaScript.
- Implements non-existent instructions by combining multiple instructions (e.g. SUB HL, BC, SLA HL).
- Outputs code that can be ROMized.
- Variables are statically allocated (recursive calls need to be handled manually).

## The Most Tedious Hello, World
This is a verbose and unoptimized "Hello, World" that can help you learn the XSM syntax.
```
org 0x100;
import MSXDOS from "xsm/msx/msxdos.xsm";

char[] hello = "Hello, World"; // null-terminated string

print(&hello);
return;

function print(DE) dispose DE {
    using(AF,BC,HL,IX,IY) {                  // push AF; push BC; push HL; push IX; push IY;
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
        // loop_end:                         // pop IY;pop IX;pop HL;pop BC;pop AF;
    }
}
// 1-line version
// MSXDOS._STROUT("Hello, World$"); // $ is terminated char
```

### Getting Started
1. Open the command palette and run `xsm init`.  
   This will create the xsmconfig.json file.

2. Create a .xsm file and write your program.

3. Open the command palette and run `xsm build`.  
   The .xsm file will be assembled and a .com file will be generated.

4. Transfer the generated .com file to an MSX-DOS or CP/M system and run it.