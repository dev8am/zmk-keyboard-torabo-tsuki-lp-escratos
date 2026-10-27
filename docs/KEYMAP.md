# キーマップ
`config/keymap.keymap` から生成しています。キーを変更したらこのファイルも更新してください。
物理レイアウトは S（44キー）です。上2段には内側キーが無いため、その位置は空欄になっています。

## layer_0 — ベース（QWERTY）
呼び出し: 常時
```
 Q     W    E    R      T                              Y    U   I     O     P
 A     S    D    F      G                              H    J   K     L     ;
 Z     X    C    V      B       Spc         L1/Ent     N    M   ,     Dot   RShf/Slsh
 Ctrl  Gui  Alt  L2/BS  L3/Spc  Shft/Eisu   RShf/Kana  Ent  BS  RAlt  RGui  RCtl
```

## layer_1 — 数字・記号
呼び出し: 位置26 `&lt 1 RET`
```
 !  @  #  $  %         ^    &    *  (  )
 1  2  3  4  5         {    -    =  }  :
 6  7  8  9  0  .   .  _    +    [  ]  Bsls
 .  .  .  .  .  .   .  Tab  Del  .  .  .
```

## layer_2 — ファンクション / 出力切り替え
呼び出し: 位置35 `&lt 2 BSPC`
```
 F1   F2   F3   F4   F5          .  .  .  .  .
 F6   F7   F8   F9   F10         .  .  .  .  .
 F11  F12  USB  BLE  NXT  .   .  .  .  .  .  .
 .    .    .    .    .    .   .  .  .  .  .  .
```

## layer_3 — カーソル・マウス
呼び出し: 位置36 `&lt 3 SPACE`
```
 .  .     .     .    .           Home  PgUp  PgDn  End  Esc
 .  .     .     .    .           Lft   Dn    Up    Rgt  Tab
 .  .     .     .    .    .   .  '     "     `     ~    |
 .  RClk  LClk  Del  Tab  .   .  .     LClk  RClk  .    .
```

## layer_4 — 空き（ZMK Studio で設定）
呼び出し: 呼び出すキーなし
```
 none  none  none  none  none               none  none  none  none  none
 none  none  none  none  none               none  none  none  none  none
 none  none  none  none  none  none   none  none  none  none  none  none
 none  none  none  none  none  none   none  none  none  none  none  none
```

## 表記

| 表記 | 意味 |
|---|---|
| `A/B` | 長押しで A、単押しで B（`&mt`） |
| `Eisu` `Kana` | 英数 / かな（`LANGUAGE_1` / `LANGUAGE_2`） |
| `Lft` `Dn` `Up` `Rgt` | カーソルキー |
| `L3/Spc` | 長押しでレイヤー3、単押しで Space（`&lt`） |
| `.` | 下のレイヤーに委譲（`&trans`） |
| `none` | 無効（`&none`） |
| `LClk` `RClk` `MClk` | マウスの左 / 右 / 中クリック |
| `USB` `BLE` | 出力先の切り替え（`&out`） |
| `NXT` `CLR` | Bluetooth プロファイル操作（`&bt`） |

## キー位置

combo や `hold-trigger-key-positions` で使う位置番号です。

```
  0   1   2   3   4        5   6   7   8   9
 10  11  12  13  14       15  16  17  18  19
 20  21  22  23  24  25   26  27  28  29  30  31
 32  33  34  35  36  37   38  39  40  41  42  43
```

## combo

| キー | 位置 | 動作 | レイヤー |
|---|---|---|---|
| `F` + `G` | 13 14 | `&bt BT_CLR` | レイヤー2のみ（その位置は `F9` + `F10`） |
