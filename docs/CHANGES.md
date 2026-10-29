# 本家との違い（詳細）

[sekigon-gonnoc/zmk-keyboard-torabo-tsuki-lp](https://github.com/sekigon-gonnoc/zmk-keyboard-torabo-tsuki-lp) からの差分です。

## 物理レイアウト

S（44キー）のみを残し、M / L と position map を削除しています。`size_s_transform` は `rows` が `4` でありながら map が `RC(4,x)` を参照していたため `5` に修正しました。

キーマップは S トランスフォームの幾何に沿って 66 → 44 キーに詰め直しています（最上段の数字行、左右の最外列、上2段の内側キーを削除）。レイヤー2・3 から失われたのは `&trans` のみで、数字・記号・F1〜F12・カーソル・BT/OUT はすべて残っています。

* `boards/shields/torabo_tsuki_lp/torabo_tsuki_lp.dtsi`
* `boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_layouts.dtsi`

ZMK Studio 上でのレイアウト切り替えの選択肢は出なくなります。

## キーマップ

**全レイヤーの配列は [KEYMAP.md](KEYMAP.md) を参照してください。** combo とキー位置番号もそちらにまとめてあります。

5レイヤー構成で、役割は次のとおりです。

| レイヤー | 役割 | 呼び出し |
|---|---|---|
| `layer_0` | ベース（QWERTY） | 常時 |
| `layer_1` | 数字・記号 | 位置26 `&lt 1 RET` |
| `layer_2` | ファンクション / 出力切り替え | 位置35 `&lt 2 BSPC` |
| `layer_3` | カーソル・マウス | 位置36 `&lt 3 SPACE` |
| `layer_4` | 空き | 呼び出すキーなし |

`layer_4` は全キー `&none` の空きです（ZMK Studio から設定する想定）。`&trans` ではないので、有効にすると未割り当てのキーは下のレイヤーに委譲されず無反応になります。

レイヤー3 をマウスレイヤーとして使います。トラックボールのスクロールと同じレイヤーに揃えるため、`&mkp LCLK` / `&mkp RCLK` を最下段の位置 33 / 34 / 40 / 41 に配置しました。`'` `"` `` ` `` `~` `|` はこのレイヤーの3段目に残してあります（記号レイヤーに空きが無いため、押す位置は従来どおり）。

マウスクリックはこのレイヤーのキーだけで、combo には割り当てていません。

## IME 切り替え

親指の `&mt LEFT_SHIFT LANGUAGE_1` / `&mt RSHIFT LANGUAGE_2` で切り替えます。ホームポジションの combo（`D` + `F` / `J` + `K`）は廃止したので、残る combo は `bt_clear`（`F` + `G`）だけです。

## mod-tap

ZMK 標準の `&mt` は `hold-preferred` で、他のキーが押された瞬間にホールドが確定します。速いタイピングで意図しない修飾が混ざるため `balanced` に変更しました。割り込んだキーが**離される**まで待つので、修飾キーとして押した場合とキーが転がった場合を区別できます。

```dts
&mt {
    flavor = "balanced";
};
```

`tapping-term-ms` は既定の 200ms、`quick-tap-ms` と `require-prior-idle-ms` は無効のままです。`&lt` は標準の `tap-preferred` を維持しています。

## トラックボールでのスクロール

レイヤー3 の間、ボールを転がすとスクロールします。上流にはトラックボール用のスクロール設定がなく（`scroller-mode` は iqs7211e トラックパッド向け）、`zmk,input-listener` のレイヤー上書きとして追加しています。

```dts
scroll_layer {
    layers = <3>;
    input-processors = <&zip_xy_to_scroll_mapper>,
                       <&zip_scroll_scaler 1 8>;
};
```

* `zip_scroll_scaler` は `値 × mul ÷ div` なので `1 8` で 1/8 に減速します。これが無いとカーソル移動と同じ分解能になり行き過ぎます
* レイヤー上書きは `process-next` が無い場合に親のプロセッサを**実行しません**（`input_listener.c` が一致時点で `return 0` する）。そのため `zip_xy_transform` を書かないことで軸反転が効かなくなり、スクロール方向が反転します。カーソルと同じ向きにしたい場合は上書き側にも `zip_xy_transform` を追加してください

左右両方の overlay に同じ記述があります。

## キーボード名

`torabo-tsuki` → `torabo-tsuki-es`（`Kconfig.defconfig` の `ZMK_KEYBOARD_NAME`）。

`ZMK_KEYBOARD_NAME` は `CONFIG_BT_DEVICE_NAME`・`CONFIG_USB_DEVICE_PRODUCT`・`CONFIG_BT_DIS_MODEL` に流れます。ZMK は `BT_DEVICE_NAME_MAX` を 16 にしているので、名前は15文字までです。16文字以上にするとビルドが `BUILD_ASSERT`（`DEVICE_NAME_LEN < CONFIG_BT_DEVICE_NAME_MAX`）で失敗します（アドバタイズの31バイト制限のため）。

### 2台を見分ける

同じファームウェアを焼いた2台は、ホストの Bluetooth 一覧・USB のデバイス名・ZMK Studio の選択ダイアログのすべてで同名になります。`CONFIG_BT_DEVICE_NAME_GATT_WRITABLE=n` なのでホスト側から改名して区別することもできません。そこで `build.yaml` に2台目用のエントリを足し、`cmake-args` で名前だけ差し替えています。

| artifact | 名前 |
|---|---|
| `torabo_tsuki_lp_right_central` / `torabo_tsuki_lp_left_peripheral` | `torabo-tsuki-es` |
| `torabo_tsuki_lp_right_central_2nd` / `torabo_tsuki_lp_left_peripheral_2nd` | `torabo-tsuki-e2` |

```yaml
cmake-args: -DCONFIG_ZMK_KEYBOARD_NAME='"torabo-tsuki-e2"'
```

Zephyr は cmake の `-DCONFIG_*` を最後に適用する Kconfig フラグメントにまとめるので、`Kconfig.defconfig` の `default` を上書きします。左右が逆の組と double ball 用のエントリには付けていません。

一覧に名前入りで現れるのは central だけです。peripheral のアドバタイズ（ZMK の `app/src/split/bluetooth/peripheral.c`）は flags・Battery Service・split サービスの UUID だけで、名前を載せていません。

**名前より先に、初回ボンディングの取り違えに注意してください。** central は peripheral を名前ではなく split サービスの UUID だけで選び、最初に見つけた1台の MAC アドレスを `ble/peripheral_addresses/0` に保存します。以降はその MAC 以外を弾き、peripheral 側もボンド後は central 宛の directed advertising に切り替わるので、**一度ペアが成立すれば混線しません**。危ないのは両方のスロットが空のとき（新品、または `settings_reset` 直後）で、2台4個の半身を同時に電源投入すると1台目の central が2台目の peripheral を掴む可能性があります。**1台ずつ、もう1台は電源を切った状態でボンドさせてください。** 取り違えたら該当の半身に `settings_reset` を焼き直します。

なお名前を変えたぶん、既にペアリング済みのホストは古い名前を表示し続けることがあります。ホスト側で一度削除してから登録し直してください。

## ホストによる改名の禁止

```
CONFIG_BT_DEVICE_NAME_GATT_WRITABLE=n
```

ZMK は `ZMK_BLE` 配下で `BT_DEVICE_NAME_DYNAMIC` を imply するため、Zephyr 側で `BT_DEVICE_NAME_GATT_WRITABLE` が既定で有効になります。この状態では接続したホストが GAP の Device Name characteristic に書き込め、`bt_set_name()` がその値を settings に永続化し、以降の起動で `CONFIG_BT_DEVICE_NAME` を上書きし続けます。

既に settings に保存されてしまった名前はこの設定では消えません。central 側に `settings_reset` を書き込んでから通常のファームウェアを焼き直してください。

## BLE 経由の ZMK Studio

`config/west.yml` の zmk を [dev8am/zmk](https://github.com/dev8am/zmk) の `studio-ble-fix` ブランチに切り替えています（上流 `v0.3` からの差分は `app/src/ble.c`、`app/src/studio/gatt_rpc_transport.c`、`app/src/battery.c`）。

上流 v0.3 でも `CONFIG_ZMK_STUDIO_TRANSPORT_BLE` は既定で有効ですが、Studio のサービス UUID をアドバタイズしないため Web Bluetooth から発見できず、実質 USB 専用でした。

使い方:

1. Chrome または Edge で https://zmk.studio を開く（Safari は Web Bluetooth 非対応）
2. Connect → Bluetooth → キーボードを選択

この手順で接続はできますが、ZMK Studio での動作確認はしていません。

`CONFIG_ZMK_STUDIO_LOCKING=n` のためロック解除操作は不要です。Studio が接続できるのは central 側だけです（`select ZMK_STUDIO_RPC if !ZMK_SPLIT || ZMK_SPLIT_ROLE_CENTRAL`）。

`revision` がブランチ指定なので、ZMK コアは fork の更新に追随します。固定したい場合はコミット SHA を指定してください。

## 起動時の電池残量表示

電源投入時にステータス LED が残量を1〜3回の点滅で示します。これが正しく出るように2箇所直しています。

**1回しか点滅しない** — 電池の定期計測は低優先度ワークキュー（優先度10）に載っていて、main スレッドの `settings_load()` が終わるまで走れません。一方 LED 側は起動から固定で 500ms 待って残量を読むため、`settings_load()` がそれより長引くと初期値の 0 を読んで「残量ゼロ = 1回点滅」になっていました。ZMK Studio でキーマップを保存すると変更したキーごとに NVS レコードが増える（`keymap/l/<layer>/<pos>`）ので、保存した瞬間からこの症状が出ます。`zmk_battery_init()` で初回だけ同期的に1回計測するようにしました。

**点滅の長さが揃わない** — `config/west.yml` の zmk-feature-status-led を [dev8am/zmk-feature-status-led](https://github.com/dev8am/zmk-feature-status-led) の `fix/serialize-led-patterns` ブランチに切り替えています。上流では Bluetooth の接続コールバックが LED を直接叩いていました。Zephyr の既定は `BT_RECV_WORKQ_BT` なのでこれは BT RX スレッド（`K_PRIO_COOP(8)`）で、残量点滅が載っているシステムワークキュー（優先度 -1）より優先度が高く、点滅の最中に別スレッドから LED を1秒点灯できてしまいます。左右間とホストの接続はどちらも起動直後に成立するので、タイミング次第で点滅がくっついたり途切れたりしていました。接続・切断のコールバックをシステムワークキューに逃がして、LED のパターンが同時に走らないようにしています。

あわせて、パターンの再生自体を `k_sleep` からワークアイテムの再スケジュールに書き換えています。上流は点滅の間隔を `k_sleep` で数えていたため、残量表示の 750ms と接続表示の1秒のあいだワークキューが丸ごと止まっていました。現在は1ステップ点灯／消灯するたびにそのステップの長さぶん自分を再スケジュールするので、ステップの合間はキューが空きます。パターンには優先度（残量 > 接続 > アドバタイズ）を持たせ、低いほうの要求は割り込まずに再生終了まで待ちます。

## 設定ファイルの場所

| 調整したいもの | ファイル |
|---|---|
| キー配列・レイヤー・combo・mod-tap | `config/keymap.keymap` |
| キーボード名 | `boards/shields/torabo_tsuki_lp/Kconfig.defconfig` |
| BLE / 電源 / Studio などの Kconfig | `boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_{left,right}.conf` |
| トラックボールの軸・スクロール | `boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_{left,right}.overlay` |
| 物理レイアウト・マトリクス | `boards/shields/torabo_tsuki_lp/torabo_tsuki_lp{,_layouts}.dtsi` |
| ZMK 本体・モジュールの取得元 | `config/west.yml` |
| ビルドするターゲット | `build.yaml` |

## 既知の制限

* **慣性スクロールは実装できません。** ZMK に該当する機能がなく、入力プロセッサを自作する必要があります。`zip_scroll_acceleration` は加速であって慣性ではありません
* **`layer_1` の中身はレイヤー3 と重複しています。** 上流ではマウスクリックが置かれているのに `&lt 1` / `&mo 1` がどこにも無く、到達できない状態でした。位置26 の `&lt 1 RET` で呼べるようにしてありますが、クリックはレイヤー3 でも押せるため、レイヤー1 は別の用途に振り直す余地があります
* **電池が空に近いと、USB を挿しても起動直後に電源が落ちます。** 非LiPoドライバは計測のたびに `CONFIG_ZMK_NON_LIPO_LOW_MV`（900mV）を下回っていないか見て、USB 給電が無ければ `sys_poweroff()` します。この判定に使う `zmk_usb_is_powered()` は USB ホストとのやり取りが始まるまで false のままで、初期化の優先度も電池が 90（`APPLICATION_INIT_PRIORITY`）、USB が 94（`ZMK_USB_INIT_PRIORITY`）です。起動時の初回計測（`zmk_battery_init()`、上の「起動時の電池残量表示」を参照）はどちらより先なので、USB を挿していても必ず「給電なし」と判定されます。**電池を抜いて USB だけで動かす場合も同じ**です。電池を替えれば済むので直していません。ブートローダはアプリより先に走るため、リセット2回押しでの書き込みは効きます
* **本家で追加されたオプションのスニペットのうち、4方向スイッチ（`kscan-4-direction-switch` / `kscan-4-direction-switch-central`）と高分解能ダイヤル（`input-hires-dial` / `split-input-hires-dial`）には対応していません。** 本家はこれらを M / L レイアウトとキーマップの `#if` 分岐で有効にしていますが、このリポジトリの S レイアウトとキーマップにはその分岐を入れていません。`led-plus` はレイアウトに依存しませんが、動作は確認していません
