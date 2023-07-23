# XSM Language Support

Z80構造化アセンブリ言語XSMのシンタックスハイライト、アセンブラを提供します。  

## オールマシン語
なんと聞こえのいい言葉か。

## 世界一面倒なHello, World
冗長かつ最適化されていない"Hello, World"ですがXSMの構文がわかります。
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
                unsafe B = *DE;              // B = A = *DE;
                // null char then exit
                if(B == 0) {                 // inc B; dec B; // set Z flag
                    break;                   // pop DE; pop AF; jp loop_end;
                }            
                // put char to console
                MSXDOS._CONOUT(B);           // E = B; C = 0x02; call BDOS;
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
