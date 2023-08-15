# XSM Language Support

Z80構造化アセンブリ言語XSMのシンタックスハイライト、アセンブラを提供します。  

## オールマシン語
なんと聞こえのいい言葉か。

### リファレンスマニュアルなど
- [xsm-vscode-extension](https://github.com/garymsx/xsm-vscode-extension)

### アップデート
- 0.0.5
  - xsmconfig.jsonにbuildを追加  
    ビルド対象を固定することが出来ます。
    ```
    	"build": [
        "main.xsm", "sub.xsm", ...
    	],
    ```
  - バグ修正
    - __heapが正しく参照できていない問題の対応

## 特徴
- 機械語をC言語やJavaScriptのような文法で表記することが出来る。
- 存在しない命令を複数の命令を組み合わせることで実現(SUB HL,BC、SLA HL等の命令)。
- ROM化可能なコードを出力する。
- 変数は静的に配置される(再帰呼び出しは自分で頑張る)。

## 世界一面倒なHello, World
冗長かつ最適化されていない"Hello, World"ですがXSMの文法を学ぶことが出来ます。
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

### 始め方
1. コマンドパレットを開き、`xsm init` を実行します。  
  xsmconfig.json ファイルが作成されます。

2. .xsmファイルを作成しプログラムを記述します。

3. コマンドパレットを開き、`xsm build` を実行します。  
  .xsmファイルがアセンブルされ、.comファイルが生成されます。

4. MSX-DOSやCP/Mなどに生成された.comファイルを転送し実行します。
