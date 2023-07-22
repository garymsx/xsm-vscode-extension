# XSM クイックリファレンス

## 特徴
- 機械語をC言語やJavaScriptのように表記することが出来る  
- 存在しない命令を複数の命令を組み合わせることで実現(SUB HL,BC、SLA HL等の機能)  
- ROM化出来ることを想定しているので自己書き換えによる高速化は行っていない
- 変数は静的に配置される(再帰呼び出しは自分で頑張る)  

## リテラル

- 数値  
  ```
  0b11001000  // 2進数
  0x1210      // 4進数
  0o144       // 8進数
  100         // 10進数
  0x64        // 16進数
  ```

- 文字  
  ```
  'A'
  '\n'   // \nはエスケープシーケンスで改行コード(LF)を表す。
  ```  

- 文字列  
  ```
  "ABC"  // 文字の最後はNULL文字(\0)が設定されます。
  ```

- 逐語的文字列  
  @を付けることでエスケープされない文字列を書くことが出来ます。
  ```
  @"Hello World\n"  // \n が改行に置き換わりません。
  ```
- 非NULL終端文字列  
  \`～\`で囲むことにより、NULL文字(\0)を付与しない文字列を書くことが出来ます。
  ```
  `ABC`
  @`ABC`
  ```

- エスケープシーケンス  
  ```
  \"  - "を入力します
  \a  - ベル
  \b  - バックスペース
  \t  - タブ
  \n  - 改行
  \v  - 垂直タブ
  \f  - 改ページ
  \r  - 復帰
  \0  - NULL文字(0)
  \x  - \xに続く16進数の00～ffで文字コードを直接指定した入力ができます。 
  \e  - \eに続く文字で画面の制御コードを送ることが出来ます。
  \\  - \ 記号
  ```
  ※ エスケープシーケンスはMSX-DOSのシステムコールなどの利用時に限ります。

### コメント

- 行コメント  
  ```
  A = 10; // コメント
  ```

- コメントブロック  
  ```
  /*
      コメント
  */
  ```

### データ
- レジスタ  
  表記は大文字、小文字はどちらでも可。
  通常は符号はなしとして扱っている。

  |レジスタの種類| 表記方法 |
  |---|---|
  |8bitレジスタ| ```A B C D E H L IXH IXL IYH IYL IT※1 RF※1``` |
  |16bitレジスタ| ```AF BC DE HL IX IY SP``` |
  |フラグの表現|```$C $NC $Z $NZ $PE $PO $P $M```|
  |メモリの参照の表記方法| ``` *HL IX[n]```|
  |符号あり表現| ```(singed)A (singed)HL```|

  ※1 IT - Iレジスタ、RF - Rレジスタ。


- 変数  

  |型名|内容|
  |---|---|
  |byte|符号なし1バイト数値型|
  |sbyte|符号あり1バイト数値型|
  |word|符号なし2バイト数値型|
  |sword|符号あり2バイト数値型|
  |char|文字型(1バイト)|
  |string|文字列参照型(2バイト)|
  
    - 配列  
    ```
    byte[2] foo;
    byte[2][4] foo;
    ```

    - 初期値  
  
    プログラムロード時の初期値を設定できます。他の言語のように毎回初期化されないことに注意してください。
    ```
    char foo;
    byte foo = 1;
    word[2] foo = {1,2};
    byte[2][4] foo = {{1,2,3,4},{5,6,7,8}};
    byte[] foo = {1,2};
    char[] foo = "ABCDEFG";            // 7文字+NULL文字で8バイト 
    char[][] foo = {"ABCDEFG", "ABC"}; // 2 x 8 = 16バイト 一番長い文字列の長さに合わせます
    string foo = "ABCDEFG";
    ```

- 定数  
    型はありません。式の中で使われた場合に置換されます。
    ```
    const foo = 0x0001;
    const foo = "ABC";
    ```

- 定数/可変定数  
    型はありません。式の中で使われた場合に置換されます。  
    ```
    const foo = 0x0001;
    const foo = "ABC";
    var   bar = 0;
    bar = bar + 1;   // 値を変更、計算することが出来ます  
    ```

- 仮想型  
    ビルドしたファイル内にデータを持たないための仕組みです。  
    例えばプログラムが200バイトでワーク領域に変数を1000バイト定義してしまったりすると
    ビルドしたファイルは1200バイトになってしまいます。  
    仮想型(virtual)を使うとプログラムの外に変数をマッピングします。  
    また、マッピング先のアドレスを直接指定することも可能です。  

    ```
    virtual word foo;              // virtualは初期値を設定することが出来ません  
    virtual char[256] foo;
    virtual(0xd000) char[256] foo; // 直接メモリにマッピングすることも可能です
    ```

    (TODO)以下の記述でvirtualデータを一括で初期化できます。アドレス指定された仮想型は対象外です。
    ```
    clear _virtual, 0, _virtualSize;
    ```

### 構造体

- 定義  
    ```
    struct FOO {
        byte bar1;
        word  bar2;
    } 
    ```
- 宣言と初期化  
    ```
    FOO foo = {1,2};
    FOO[] foo = {
          {1,2}
        , {3,4}
    };
    ```

### 式

- 代入式(=)  

  ニーモニックのLD命令に置き換えられますので、複雑な式は書けません。  
  XSMの代入式では、Z80がサポートする組み合わせ以外の多くの代入式のパターンを疑似的にサポートしています。  
  ```
  A = 1;            // レジスタへの代入
  A = 5 + 2 * 8;    // アセンブル時に値が確定するのであれば計算式を書くことが可能です
  *HL = 1;          // HLの差すメモリへの代入
  foo = A;          // メモリへ直接代入
  A = foo.bar1;     // 構造体の参照
  A = foo[2];       // アセンブル時にアドレスが確定している場合は配列が書けます
  HL = &foo;        // fooのアドレスを取得します
  A = 00;           // A = 0でなくA = 00とすると、XOR Aを実行する
  A = IX[0];        // インデックスレジスタは配列のように扱えます
  HL = BC;          // LD H,B; LD L,C;を展開します
  HL = IX[0];       // LD HL,(IX+1); LD L, (IX+0);を展開します。
  A = (FOO)IX.bar1; // 256バイトまでの構造体であれば、IXを構造体にキャストすることで動的にアクセスできます。  
  ```

- 演算代入式(+= -= *= /= %=)  
  計算を行う場合はこれらの組み合わせで行います。
  ```
  A += 2;
  A:$C += 2;   // キャリー付き(ADC) 
  A -= 3;
  A:$C -= 3;   // キャリー付き(SBC)
  HL -= BC;    // OR A;SBC HL,BC;が実行されます
  HL *= A;     // 乗算、除算、余剰算はサブルーチンによりサポートされます
  HL /= BC;
  HL %= D;
  ```

- インクリメント、デクリメント(++ --)  
  ```
  A++;
  HL--;
  ```  

- シフト、ローテーション(>> << >>> <<<)  
  ```
  A<<4;          // 4回繰り返し      SLA
  (signed)A>>;   // 符号付きシフト   SRA
  A>>>;          // ローテーション   RRC
  HL:$C>>>;      // キャリー付き     RR
  ```

- 論理演算式(&= ^= |=)  
  ```
  A &= 0b00001111;  // AND
  A ^= 0b00001111;  // XOR
  A |= 0b00001111;  // OR
  ```  

- 関数呼び出し  
  ```
  // 定義
  function foo(B, C, D = 3) ...
  function bar(HL) ...

  A = foo(B,2);             // B = Bは省略され、C = 2; D = 3が設定される。
  A = bar("Hello World\n"); // 文字列へのアドレスが渡される
  ```

### 関数定義

- 書式  
  ```
  // :レジスタでreturnは応答に使用するレジスタ、disposeは呼び出しによって壊れるレジスタを定義します。
  // disposeを定義しておくと、関数呼び出し後に記述したレジスタを初期化せずに使用すると警告が出ます。
  function foo(BC, string bar1): A dispose DE {
    ...
  }
  
  // デフォルト値
  function foo(C = 0x05); // パラメータ省略時の値を指定

  // BIOSや常駐処理を呼ぶ際は以下のように定義できます。
  function foo(C) = 0xe000; // アドレスを指定
  ```

### ラベルとジャンプ

- ラベル  
  ```
  foo:
  ```
- ジャンプ  
  ```
  goto foo;
  ```

- サブルーチンコール  
  ```
  call foo;
  ```

- 関数、サブルーチンからの復帰  

  ```
  // 戻り値なし、または省略時
  return;
  // 関数に戻り値が設定されている場合は引数を持ちます。
  return 1;
  // function foo(): A と定義されていれば return 1 は以下のように展開されます。
  LD A,1
  RET
  // return A; のようになった場合、LD A,Aは省略されます
  ```

- returni / returnn  
  ```
  // 割り込み復帰用のreturnです。こちらは引数を持ちません。
  returni;
  returnn;
  ```

# 疑似命令

- unsafe   

  A / HLレジスタの暗黙的使用を許可します。  
  unsafeブロック内では代入式、演算代入式、条件式などが大幅に拡張されます。  
  A / HL レジスタを経由することで可能になる式はその代入を省略できます。
  当然ですが A / HLレジスタは壊れます。
  ```
  unsafe {
    *DE = B;
  }
  // これは以下のように展開されます、Aレジスタの元の値は壊れます。
  LD A,B
  LD (DE),A
  
  unsafe {
    BC += DE;
  }
  // これは以下のように展開されます、HLレジレスタの元の値は壊れます。
  LD H,B
  LD L,C
  ADD HL,DE
  LD B,H
  LD C,L

  // さらにunsafeにパラメータBC(またはDE)を与えるとBCレジスタまで暗黙的に使用するようになります。 
  unsaf(BC) {
    HL += 123;
  }
  LD BC,123
  ADD HL,BC
  ```

### 制御文

- if  

  条件分岐処理
  ```
  if(A == 0) {...}
  if(A != 0) {...} else {...}
  if(C >= 10 && C <= 20) ...                   // unsafe内なら可能
  if((signed)A >= -10 && (signed)A <= 10) ...  // 符号付きで判定するときは(signed)を付ける
  if(HL >= 10 && HL <= 20) ...                 // PUSH/SBCなどを駆使してCP HL,xxを実現していますのでオーバーヘッドがあります。
  if($C) ...    // キャリーフラグ
  if($NZ) ...   // ゼロフラグの否定
  ```  

  ```
  // また、const値と併用して条件付きコンパイルのようなことが可能です。
  const foo = 1;
  if(foo == 1) {        // ビルド時に結果が決まっているのでif文は生成されない
    // このブロックはビルドされる
  } else {
    // このブロックのコードはビルドされない
  }
  ```  

- for  

  ループ処理
  ```
  for(A = 0; A < 100; A++) {
    …
  }
  ```

- while  

  ループ処理
  ```
  while(A < 10) {
    A++;
  }
  
  // 式を省略すると無限ループになります。
  while() {
  }
  ```

- do  

  ループ処理(判定をループ最後に行う)
  ```
  do {
    A++;
  } while(A < 10)
  ```

- loop  

  DJNZによるループ処理。制限はありますがループ処理では一番コンパクトになります。
  ```
  loop(10) { // Bレジスタが暗黙的に使用されます。
    ...      // ループ内の処理は126バイトに納めないとアセンブル時にエラーになります。
  }
  // Bレジスタにすでに値が入っている場合でもこのように書いてください。LD B,Bは省略されます。
  loop(B)
  ```

- switch  

  Aレジスタの値別に分岐を行います。
  ```
  // caseの判定にはAレジスタが使われます。例えばswitch(B)とすると、LD A,Bが事前に実行されます。
  switch(A) { 
    case(0) {
      ...        // C言語のswitchのようなbreakは不要です。次のcaseは判定しません。
    }
    case(1,2) {  // 複数条件を書けます
      ...
    }
    default {
      ...
    }
  }
  ```

- continue / break  

  ループ継続または、ループ処理から脱出します。
  ```
  while(A < 10) {
    if( xxxx ) continue; // ループ最初から
    if( xxxx ) break;    // ループ脱出
  }
  ```

- on ～ goto / call / return  

  フラグの内容が成立している場合、ジャンプ命令を実行します。
  ```
  on $Z  goto foolabel;  // ジャンプ命令
  on $C  call foolabel;  // サブルーチンコール
  on $NZ return;          // 復帰
  ```

- using  

  レジスタの安全なPUSHとPOPを提供します。
  usingを抜ける際に自動的にPOPを実行します。
  ```
  using(HL) {         // PUSH HL
    ...
    using(BC,DE) {    // PUSH BC; PUSH DE;
      ...
      if(...) return; // 条件成立時 POP DE; POP BC; POP HL;
    }                 // POP DE;POP BC;
    ...
  }                   // POP HL
  ```

- try  

  例外処理です。ただし関数を超えて例外を飛ばすことは出来ません。
  ```
  try {
    if(...) throw 1;  // A = 1が実行される
    return;
  } catch(A) {
    ...               // throw時に実行される処理
  } finally {
    ...               // tryを抜けるときに必ず実行される処理
  }
  ```

- @ ブロック  

  無名ブロック。ブロック内で continue と break が使えるようになります。
  ```
  @ {
      if(xxx) break;    // 終了条件
      continue;         // ループ
  }
  ```

### let(TODO)

  let文を使うと計算式を実行し結果を得ることが出来ます。  
  この命令はB/C/IX以外のレジスタを使用して計算式を実行し、使用するレジスタの退避は行いませんので注意してください。  
  ```
  // 普通の式
  B = 1;
  C = 5;
  byte x = 10;
  let A = B + C * x;   // A = 51;

  // アドレスの取得
  B = 1;
  byte[2][2] data = {{0,1},{2,3}};
  let HL = &data[B][0]; // 配列のサイズが256バイト以内におさまる場合、添え字に16bit値は使えません。
  A = *HL;              // A = 2;

  // 間接参照
  byte data2 = 123;
  word ptr = &data2;
  let A = *ptr;         // A = 123;

  // 16bitの場合の間接参照
  word data3 = 1234;
  word ptr = &data3;
  let HL = *ptr;        // HL = 1234;

  // 構造体のキャスト
  // キャストは((構造体)レジスタ)の書式のみです。
  let A = ((FOO)BC).bar;

  ```

  let文は代入先が8bitの場合は8bit式、16bitの場合は16bit式として判断し使用されるレジスタが変わります。  
  最適化により使用されない場合がありますが、破壊されるレジスタの目安にしてください。  
  | 式  | 計算結果 | 途中結果 | 変数参照 | 添え字      |
  |-----|---------|---------|---------|-------------|
  |8bit | A       | D       | HL      |E            |
  |16bit| HL      | DE      | IY      |DE           |

### 内部関数

- データ定義関数(bin/qtr/hex)  
  ```
  byte[] foo  = bin("00000000"         // 2進数表記
                   + "11,11,11,11");    // 数字以外は無視されます
  byte[] foo = qtr("0123,0123");       // 4進数
  byte[] foo = hex("FF,FF");           // 16進数
  ```

- ファイル取り込み関数(from)  
  ファイルのイメージを取り込み変数の初期値とします。
  また、取り込む際にコンバーターを実行しプログラムから扱いやすい形に変換することもできます。
  詳しくは[from関数について](from)を参照してください。
  ```
  byte[] foo = from("data.bin");           // ファイルをそのままfooに格納
  byte[] foo = from("tile.bmp", "sc1");    // BMP(Indexed Color)形式の画像をSCREEN1のVRAM形式に変換
  ```

- 分離関数(high/low)  
  ```
  word foo = 0x1234;
  A = high(foo);     // Aには0x12が入る
  A = low(foo);      // Aには0x23が入る
  ```

- サイズ取得関数(sizeof)  
  ```
  word[10] foo;
  A = sizeof(foo);  // Aには20が入る
  ```

- 要素数取得関数(length)  
  ```
  word[10] foo;
  A = length(foo);  // Aには10が入る
  ```

- オフセット関数(offset)  
  構造体メンバーの先頭からのオフセット値を求めます。構造体に配列を使う場合に必須になってくる機能です。

  ```
  // 例 foo[1].foo2のアドレスを求める
  struct FOO {
    byte  bar1;
    byte  bar2;
  };
  FOO[2] foo;
  HL = &foo;                          // foo[0]のアドレスを取得
  unsafe(BC) HL += sizeof(foo);       // foo[1]のアドレスを求める
  unsafe(BC) HL += offset(foo.bar2);  // foo[1].bar2の先頭アドレス
  ```

- 半角変換関数(half)  
  ```
  string foo = half("あいうえお"); // MSXのひらがなASCIIコードに変換されます。
  ```

- ビット判定関数(set/res)  
  ```
  // if文内で使える関数です。
  if(set(A,0)) { ... } // Aレジスタのビット0がONの場合に条件が成立します。
  if(res(B,7)) { ... } // Bレジスタのビット7がOFFの場合に条件が成立します。
  ```

- 型名取得  
  ```
  // 型名を文字列で取得します。inline関数の判定などで使えます。
  inline xxx(param) {
      if(typename(param) == "A")      info("レジスタです");  // レジスタ
      if(typename(param) == "string") info("文字です"); 
      if(typename(param) == "number") info("数値です");     // リテラル数値
      if(typename(param) == "byte")   info("byte型です");   // 変数
      if(typename(param) == "word")   info("word型です");   // 変数
  } 
  ```

- キャラクタコード変更  
  ```
  // キャラクタコードのマッピングを変更します。
  charmap('A',0xa1);
  charmap('B',0xa2);
  …
  charmap('Y',0xa6);
  charmap('Z',0xba);

  char[] foo = "AB";  // -> 0xa1,0xa2と出力される。
  ```

- メッセージ関数(error/warn/info)  

  ```
  ログを出力する関数です。inline関数のエラーに使ったりします。
  error("message"); 
  warn("message");
  info("message"); 
  ```

#### その他の命令

一般的なニーモニックと同じように使えますが、一部書式が変更されているものや追加されたものがあります。  
追加や変更されているもののみ詳細を記載しています。

- dvr
  除算と余算の結果を同時に得ます。以下の組み合わせのみサポートしています。
  ```
  dvr A,D;        // 商はBレジスタ、余りはAレジスタに入ります。
  dvr HL,D;       // 商はBCレジスタ、余りはHLレジスタに入ります。
  dvr HL,DE;      // 商はBCレジスタ、余りはHLレジスタに入ります。
  ```

- push  
  ```
  push HL;
  push BC,DE,HL;  // まとめてpush出来ます
  ```

- pop  
  記述した順番の逆からpopします。pushと並びを揃える為です。
  ```
  pop HL;
  pop BC,DE,HL;  // pop HL; pop DE; pop BC;
  ```

- ldi / ldir / ldd / lddr  
- cpi / cpir / cpd / cpdr  
- nop  
- halt  
- di  
- ei  
- im  
- ex / exx  
  ```
  ex AF,AF;
  ex DE,HL;
  exx;
  ```
  
- daa  
- cpl  
  ```
  cpl A;
  ```
  
- neg  
  ```
  neg A;
  ```
  
- ccf  
- scf  
- rst  
- in  
- ini / inir / ind / indr  
- out  
- outi / otir / outd / otdr  
- bit  
  ```
  bit A,7;
  bit 7,A; // どちらでも同じ意味です。
  ```
  
- set  
  ```
  set A,7;
  set 7,A; // どちらでも同じ意味です。
  ```
  
- res  
  ```
  res A,7;
  res 7,A; // どちらでも同じ意味です。
  ```

- move  
  ```
  byte[5] buf = {0,1,2,3,4};
  byte[5] buf2;
  move buf2, buf;    // buf2へbufをコピー、以下が使用される
                     // DE - 転送先
                     // HL - 転送元
                     // BC - サイズ
  move buf2, buf, 5; // サイズ指定も可能
  ```
  内部的にLDIRを実行する命令です。BC/DE/HLレジスタが壊れます。
  
- clear  
  ```
  byte[5] buf = {0,1,2,3,4};
  clear buf, 0;
  clear buf, 0, 5; // サイズ指定も可能
  ```
  LDIRを応用した初期化命令です。BC/DE/HLレジスタが壊れます。

### マクロ関連

- inline関数定義  

  ```
  // パラメータは呼び出し時に使用箇所に埋め込まれます。
  // returnは使えません。
  inline foo(bar): A {
    if(bar == 1) A = 10;
    if(bar == 2) A = 20;
    ...
  }
  
  // このマクロは以下のように展開されます
  foo(1); // -> A = 10;
  foo(2); // -> A = 20;
  ```
  
- repeat  
  繰り返し処理、ループ系と違い指定された回数同じコードを出力します。
  ```
  // HLにBCを8回足す
  repeat(8) {
    HL += BC;
  }
  ```

### システム定数、システムラベル

- __virtual / __virtual_size
  virtualデータ領域の開始アドレス、サイズが取得できます。
  ```
  clear __virtual, 0, __virtual_size; // virtual型の一括初期化
  ```

- __heap  
  ```
  HL = __heap;   // プログラムの終わり+1のアドレスを差します。このアドレス以降メモリを自由に使えます。 
  ```

### プリプロセッサ
厳密にはプリプロセッサというわけではないのですが、プログラムではない命令群を便宜上プリプロセッサとします。

- module  
  デフォルトでは出力ファイルは.comファイルが出来上がりますが、moduleで別の名前にすることが出来ます。
  ```
  module foo.bin;
  ```

- org  
  プログラムを配置するアドレスを指定します。デフォルトは0x0100です。  
  プログラムにつき１回しか宣言できません。  
  ```
  org 0x100;            
  org 0x4000, 0x8000;   // 2つ目のパラーメタはvirtual型変数の配置されるアドレスです。
  // 例えばROMファイルを作る場合、同じページに変数を置けませんので別ページとなるアドレスを指定します。
  ```

- import  
  外部ファイルを参照し、呼び出し元のモジュールに追加します。
  ```
  import "foo.xsm";
  import "foo.xsm", 0x8000, virtual;  // 指定されたアドレスにプログラムがロードされているものとして扱います。ビルド時にはバイナリは生成されません。
  import foo from "foo.xsm", 0x8000;  // アドレス指定付きで読み込みます。プログラムをそのアドレスに転送する必要があります。
  // 転送はmove命令で行います。
  move 0x8000, foo;
  ```

- include  
  外部ファイルを取り込みその場に挿入されます。  
  importで足りる場合はimportを使ってください。
  ```
  include "foo.xsm";
  ```

# アノテーション
  最適化のヒントを与えたり、コンパイラの制御を行ったります。

- @jp/@jr  
  if文やfor文等で内部的に生成されるジャンプ命令にどちらを使用するか強制します。
  ```
  @jp
  if(A == 0) {
    ...
  }

  @jr
  for(A = 0; A < 10; A++) {
    ...
  }
  ```

- @ignorewarn  
  warnメッセージがでないようにします。

  ```
  unsafe *BC = D;
  @ignorewarn
  if(A == 0) { // *BC = DによりAが壊れている為警告が出ますが、それを表示しないようにします。
    ...
  }

  ```

