# xsmconfig
コンパイラの設定を定義します。  
xsm initコマンドで作成することができます。

## 例
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
ロケールを指定します。  
エラーメッセージがロケールに応じて変わります。  
  - ja - 日本語  
  - en - 英語

### rooDir
ソースファイルのルートディレクトリを指定します。  

### outputDir
ビルドファイルの出力先を指定します。

### importDir
importやincludeが参照するファイルのルートディレクトリを指定します。  
複数指定することができます。

### build
ビルド対象のファイルを指定します。  
指定されていない場合は、開いているファイルをビルドします。  

### optimize
最適化オプションを指定します。  
  - cacheRegister - 暗黙的レジスタ使用が行われた場合、使用したレジスタをキャッシュし何度も参照しないようにします。  
  - jrJump - if文などで内部で生成されるJP命令をJR命令に置き換えます。JRに出来ない場合はJP命令のままです。

 ### debug
デバッグオプションを指定します。  
  - source - ビルド時のアセンブラソースが確認できます。  
  - comment - ソースに詳細な情報を追加します。

