# XSM Language Support

Z80構造化アセンブリ言語XSMのシンタックスハイライト、アセンブラを提供します。  

## サンプル
```
import "library/msx/msxdos.xsm";

char[] hello = "Hello, World";

HL = &hello;
for(A = 0; A < sizeof(hello); A++) {
    using(AF,HL) {
        _CONOUT(*HL);
    }
    HL++;
}

return;
```

### 始め方
1. コマンドパレットを開き、`xsm init` を実行します。  
  xsmconfig.json ファイルが作成されます。

2. .xsmファイルを作成しプログラムを記述します。

3. コマンドパレットを開き、`xsm build` を実行します。  
  .xsmファイルがアセンブルされ、.comファイルが生成されます。

4. MSX-DOSやCP/Mなどに生成された.comファイルを転送し実行します。
