# Memory Configuration

This information may be useful when switching pages or making ROMs.

## Overall

The program is built with the following memory configuration:

|Area              |Explanation                                                         |
|:------------------|:--------------------------------------------------------------------|
|                  |↑ Address 0x0000                                                     |
|Binary Area       |This area contains the program, data, and string literals.            |
|                  |↓ This is not included in the binary file.                           |
|Virtual Type Data Area|This area is referred to by data types with the virtual modifier.    |
|Heap Area         |This is unallocated memory and can be accessed with the system variable __heap.|
|Virtual Program Area| This area is considered to contain imported programs with the virtual keyword.|
|                  |↓ Address 0xFFFF                                                     |

## Binary Area

Furthermore, the binary area has the following configuration:

|Area                |Explanation                                                         |
|:--------------------|:-------------------------------------------------------------------|
|Main Program Area   |This area outputs the main (non-function) program.                    |
|Function Area [ ]   |This area outputs the program and data of the functions. It repeats for each function.|
|Data Area           |This area contains the variables defined in the main program.        |
|Import Area [ ]     |The above configuration is repeated for each import.                 |