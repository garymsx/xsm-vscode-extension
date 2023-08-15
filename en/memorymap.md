# Memory Configuration
The following information may be useful when switching pages or ROMizing.

## Overall

The program is built with the following memory configuration.

|Area                |Explanation                                                         |
|:-------------------|:-------------------------------------------------------------------|
|                    |↑ Address 0x0000                                                     |
|Binary Area         |This area contains programs, data, and string literals.               |
|                    |↓ The following is not included in the binary file.                  |
|Virtual Data Area   |This area is referred to by data types with the virtual modifier.     |
|Heap Area           |This is unreserved memory and can be accessed with the system variable __heap.|
|Virtual Program Area|This area is considered to store programs imported with the virtual specifier.|
|                    |↓ Address 0xFFFF                                                     |

## Binary Area

Furthermore, the binary area has the following configuration.

|Area                |Explanation                                                         |
|:-------------------|:-------------------------------------------------------------------|
|Main Program Area   |This area outputs the main program (outside of functions).            |
|Function Area [ ]   |This area outputs the program and data of each function.              |
|Data Area           |This area defines variables in the main program.                      |
|Import Area [ ]     |The above configuration is repeated for each import.                  |