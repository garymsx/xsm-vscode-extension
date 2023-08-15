# xsmconfig
Defines the settings for the compiler.  
Can be created using the xsm init command.

## Example
```
{
	"locale": "ja",
	"rootDir": "./src",
	"outputDir": "./out",
	"build": [
		"program1.xsm",
		"program2.xsm"
	]
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
Specifies the output directory for the build files.

### importDir
Specifies the root directory of the files referenced by import or include.  
Multiple directories can be specified.

### build
Specifies the files to be built.  
If not specified, the currently open file will be built.

### optimize
Specifies the optimization options.  
  - cacheRegister - If implicit register usage is performed, caches the used registers to avoid repeated references.  
  - jrJump - Replaces JP instructions generated internally in if statements with JR instructions. If JR is not possible, it remains as JP instructions.

### debug
Specifies the debug options.  
  - source - Allows viewing of the assembler source during the build.  
  - comment - Adds detailed information to the source.