# torabo-tsuki LP（S サイズ）用 ZMK ファームウェア — Escratos 対応版

> [!WARNING]
> これは個人が趣味で作っている非公式のファームウェアです。書き込みや使用は自己責任でお願いします。
> このファームウェアを使ったことでキーボードやパソコンが故障した場合も、その他どのような損害が生じた場合も、作者は一切の責任を負いません。

[torabo-tsuki LP](https://github.com/sekigon-gonnoc/torabo-tsuki-lp) の S サイズ（44 キー）向けの ZMK ファームウェアです。
本家 [sekigon-gonnoc/zmk-keyboard-torabo-tsuki-lp](https://github.com/sekigon-gonnoc/zmk-keyboard-torabo-tsuki-lp) をもとに、キーマップ編集用の Web アプリ [Escratos](https://dev8am.github.io/escratos/) から **Bluetooth 経由で** キーマップを編集できるようにしています（USB でも編集できます）。

## 本家との違い

| | 本家 | このファームウェア |
|---|---|---|
| 物理レイアウト | S / M / L | **S（44 キー）のみ** |
| ZMK Studio の接続 | USB のみ（Bluetooth では見つからない） | **USB と Bluetooth** |
| キーマップ | 66 キー前提 | 44 キーに詰め直し、レイヤーを再構成（[配列](docs/KEYMAP.md)） |
| mod-tap | `hold-preferred` | `balanced`（速く打っても修飾キーが混ざりにくい） |
| トラックボール | カーソル移動 | レイヤー 3 の間はスクロール |
| キーボード名 | `torabo-tsuki` | `torabo-tsuki-es`（2 台目用に `torabo-tsuki-e2` のビルドあり） |
| ホストからの改名 | できる | できない（勝手に名前が変わらない） |
| 起動時の電池残量表示 | 残量が正しく出ないことがある | 正しく出るよう修正 |

それぞれの理由と仕組みは [docs/CHANGES.md](docs/CHANGES.md) にまとめています。

このために、ZMK 本体とステータス LED モジュールは次の fork を使っています（`config/west.yml`）。

* [dev8am/zmk](https://github.com/dev8am/zmk) の `studio-ble-fix` ブランチ — ZMK Studio を Bluetooth で見つけられるようにする修正と、起動時の電池計測の修正
* [dev8am/zmk-feature-status-led](https://github.com/dev8am/zmk-feature-status-led) の `fix/serialize-led-patterns` ブランチ — LED の点滅パターンが混ざらないようにする修正

## 書き込み方

1. [Releases](https://github.com/dev8am/zmk-keyboard-torabo-tsuki-lp-escratos/releases/latest) から、使う uf2 ファイルをダウンロードします
2. 名前に `_central` が付いている uf2 をトラックボールがある側に、`_peripheral` が付いている uf2 を反対側に書き込みます

| トラックボールの位置 | トラックボール側 | 反対側 |
|---|---|---|
| 右 | `torabo_tsuki_lp_right_central` | `torabo_tsuki_lp_left_peripheral` |
| 左 | `torabo_tsuki_lp_left_central` | `torabo_tsuki_lp_right_peripheral` |
| 右（2 台目） | `torabo_tsuki_lp_right_central_2nd` | `torabo_tsuki_lp_left_peripheral_2nd` |

2 台を使い分ける場合は、1 台ずつ、もう 1 台の電源を切った状態で左右をペアリングしてください。理由は [docs/CHANGES.md](docs/CHANGES.md#2台を見分ける) にあります。

うまく動かなくなったときは、同じ Releases にある `settings_reset` を書き込んでから、もう一度ファームウェアを書き込みます。

リリース前の最新の変更を試したい場合は、[Actions](https://github.com/dev8am/zmk-keyboard-torabo-tsuki-lp-escratos/actions) の最新のビルドから `firmware` をダウンロードすることもできます（GitHub へのログインが必要です）。

## キーマップの編集

* **Escratos**: Chrome または Edge で https://dev8am.github.io/escratos/ を開き、Bluetooth または USB で接続します
* **ZMK Studio**: https://zmk.studio からも同じように接続できますが、ZMK Studio での動作確認はしていません

ロック解除の操作は不要です（`CONFIG_ZMK_STUDIO_LOCKING=n`）。接続できるのはトラックボールがある側（central）だけです。

## 対応していないもの

* M / L サイズ
* 本家で追加されたオプションのスニペット（4 方向スイッチ、高分解能ダイヤル）

ほかの制限は [docs/CHANGES.md](docs/CHANGES.md#既知の制限) を参照してください。

## ライセンス

本家と同じく [LICENSE](LICENSE) に従います。
