# XSM Language Support(Z80 Assembler)

Provides syntax highlighting and assembler for the Z80 structured assembly language XSM.

## Machine Language
What a beautiful phrase.

### Reference Manuals, etc.
- [xsm-vscode-extension](https://github.com/garymsx/xsm-vscode-extension)

### Updates
- 0.0.9
  - Added placement address to `import`
  - Added output size to `module`
- 0.0.6 - 0.0.8  
  Bug fixes
- 0.0.5
  - Added "build" to xsmconfig.json  
    You can now specify the build target.
    ```
    	"build": [
        "main.xsm", "sub.xsm", ...
    	],
    ```
  - Bug fixes
    - Fixed an issue where __heap was not being referenced correctly.

## Features
- Machine code can be expressed using syntax similar to C or JavaScript.
- Non-existent instructions can be achieved by combining multiple instructions (e.g. SUB HL, BC, SLA HL, etc.).
- Outputs code that can be ROMized.
- Variables are statically allocated (recursive calls must be handled manually).

## The Most Boring Hello, World in the World
This is a verbose and unoptimized "Hello, World" that allows you to learn XSM syntax.
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

2. Create a .xsm file and write your program.

3. Open the command palette and run `xsm build`.  
   The .xsm file will be assembled and a .com file will be generated.

4. Transfer the generated .com file to MSX-DOS, CP/M, or other compatible systems and run it.