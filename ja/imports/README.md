# 標準importファイル

## ファイル一覧

- [bios.xsm](./xsm/msx/bios.xsm)

  MAIN ROMのBIOSが定義されています。  
  インタースロットコール経由で呼び出すインライン関数の定義がされている為、DOSからもそのまま呼び出せます。

- [extbios.xsm](./xsm/msx/extbios.xsm)

  SUB ROMのBIOSが定義されています。  
  こちらもDOSからもそのまま呼び出せます。

- [msxdos.xsm](./xsm/msx/msxdos.xsm)

  MSX-DOSのファンクションコールが定義されています。  

- [workarea.xsm](./xsm/msx/workarea.xsm)

  BIOSやBASICから参照されるワークエリアが定義されています。
