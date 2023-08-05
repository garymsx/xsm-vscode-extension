# xsmconfig
Defines the settings for the compiler.  
Can be created using the xsm init command.

## Example
```
{
	"locale": "ja",
	"rootDir": "./src",
	"outputDir": "./build",
	"importDir": [
		"./include",
	],
	"optimize": {
		"cacheRegister" : true,
		"jrJump": true
	},
  "debug": {
		"source": true,
		"comment": true
  }
}
```

### locale
Specifies the locale.  
Error messages will vary depending on the locale.  
  - ja - Japanese  
  - en - English

### rootDir
Specifies the root directory of the source files.

### outputDir
Specifies the output directory of the build files.

### importDir
Specifies the root directory of the files referenced by import or include.  
Multiple directories can be specified.

### optimize
Specifies the optimization options.  
  - cacheRegister - If implicit register usage is detected, the used register is cached to avoid repeated referencing.  
  - jrJump - Replaces JP instructions generated internally with JR instructions in if statements, etc. If JR is not possible, JP instructions remain the same.

### debug
Specifies the debug options.  
  - source - Allows you to view the assembler source during the build.  
  - comment - Adds detailed information to the source.