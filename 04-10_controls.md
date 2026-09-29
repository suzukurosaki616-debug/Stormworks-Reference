### 4.10 操縦系部品(座席・舵・Gyro・車輪)— 定義XMLの読解結果 + ユーザー知見

**【一次資料 2026-09-18】`rom/data/definitions/*.xml` のうち category=1(操縦系)55 ファイルを機械抽出して読んだ結果。**
RX 系 12 ファイルは §4.7 に既にあるので除く。**挙動の記述は「ユーザー知見」札のもの以外は未検証。**

> §4.9 と同じく、**定義XMLは挙動を書いていない。** 舵角・応答速度・発生力はどこにも書かれておらず、
> 書いてあるのは「何のノードがあり、tooltip が何と言っているか」だけ。

#### 4.10.1 座席(`seat_*.xml`、8種)

**Pilot Seat / Compact Pilot Seat / Control Handle / Helm / Driver seat / Saddle Seat は全部ノードが同じ(18本)。**
違いは mass(24 / 7 / 1 / 15 / 10 / 5)と乗降姿勢(`seat_pose`)だけ。
**【ユーザー知見 2026-09-18】見た目以外の挙動差は無い。** 船なら Helm を置けばよく、Lua 側は区別しなくてよい。

| # | mode | 型 | ノード | 内容(原文要約) |
|---|---|---|---|---|
| 1 | out | bool | `Occupied` | 着座中 on |
| 2 | out | bool | `Trigger` | トリガーキー押下中 on(レベル) |
| 3 | out | num | `Look X` | 視線方向(turn) |
| 4 | out | num | `Look Y` | 視線方向(turn) |
| 5 | out | num | `Axis 1` | 左/右。**右で +1** |
| 6 | out | num | `Axis 2` | 上/下。**上で +1** |
| 7 | out | num | `Axis 3` | ペダル左/右。**ペダル右で +1** |
| 8 | out | num | `Axis 4` | スロットル上/下。上で +1 |
| 9-14 | out | bool | `Hotkey 1`〜`6` | 押下中 on(レベル) |
| 15 | out | comp | `Seat data` | 下記 |
| 16 | out | audio | `Headset Audio` | |
| 17 | in | audio | `Headset Audio` | |
| 18 | in | video | `Headset Video` | *"Displays video UI overlay on a helmet mounted display."* |

**`Seat data`(composite)の割り当て(原文 `seat.xml`)**:
bool 1〜 = Hotkey / **bool 31 = Trigger / bool 32 = Occupied** / num 1〜4 = Axis 1〜4 / **num 9 = Look X / num 10 = Look Y**。
**num 5〜8 は空き**(HOTAS では Controller Axes)。Lua なら `Seat data` 1本で座席の全情報が取れる。

- **【ユーザー知見】`Axis 1〜4` は設定次第で「離すと 0 に戻る」とも「その位置に留まる」ともできる。軸ごとに感度と挙動をゲーム設定で変えられる。**
  → Lua 側は「この軸はセンタリングする」と決め打ちしないこと。スロットル軸を留まる設定にしている操作者と、戻る設定の操作者が同じ機体に乗りうる
- **【ユーザー知見】`Look X`/`Look Y` は turn 単位だが、真横まで向けるわけでもなく真後ろは向けない。実用範囲は ±0.3 程度。**
  **【JP Wiki VEHICLE CONTROL】Look X = −0.35〜0.35(±126°)、Look Y = −0.2〜0.2(±72°)。** 基準(座席正面=0 と推定)と符号は未検証
- 軸の符号規約は **§9.2** を正とし、ここでは繰り返さない

**Pilot Seat (HOTAS)**(`seat_hotas.xml`、mass 50、64ノード): 上記に加えて **num `Axis 5〜8`(controller axis)と bool `Hotkey 7〜48`** を持つ。
`Seat data` の num 5〜8 に Axis 5〜8 が入ることは原文にある。**Hotkey 7〜48 が bool 7〜30 に入るのか、31/32(Trigger/Occupied)と衝突して
切り捨てられるのかは原文に無い。** 【ユーザー知見】HOTAS 自体の挙動は不明。使うなら実機で bool 7 以降を読んで確かめる。

**Space Seat**(`seat_space.xml`、mass 3、19ノード): 上記 18 本 + **fluid `Fluid In`**。宇宙用で酸素供給(【JP Wiki】*着席時に酸素供給可能*)。使う予定は無いが存在だけ記載。

**Passenger Seat**(`passenger_seat.xml`、category 4)は bool `Occupied` 1本だけ。乗員検知に使える。

#### 4.10.2 舵・操縦翼面(8種)

**全種ノードが同じ**: num `Rotation`(in、−1〜1)+ elec `Electric`(in)。原文: *"a value between -1 and 1 that represent the two extremes of the rudder's rotation."*

| 部品 | ファイル | mass | `rudder_surface_area` |
|---|---|---|---|
| Rudder | `rudder` | 10 | 13 |
| Fin Rudder | `rudder_surface` | 5 | 3.2 |
| Control Surface (Small / Medium / Large) | `control_surface_*` | 10 / 15 / 25 | 10 / 28 / 70 |
| Control Fin (Small / Medium / Large) | `control_fin_*` | 2 / 5 / 10 | 2 / 8 / 12 |

`rudder_surface_area` は定義XMLに書かれている数少ない「効きに関係しそうな数値」だが、**力との対応は未検証**。
**【ユーザー知見】Control Surface (Large) は見た目の割に効かない**、という体感があるので面積比のまま信じないこと。

- **【ユーザー知見】位置指令。応答は即時**(動く速さは事実上無限)。速度指令ではない
- **【ユーザー知見】最大舵角は 45° 程度。90° は見たことがない。【JP Wiki VEHICLE CONTROL】Rudder / Fin Rudder / Control Surface S-M-L = ±45°、Control Fin S-M-L = ±15°**
- **【ユーザー知見】`Rotation` の +1 が「右」なのではなく、パーツ設置時の矢印に従う。反転設置すれば逆になる。**
  → **符号はプロパティで較正する(§9.3)。** Lua 側で「+1=右舵」と書いてよいのは、§9.2 の規約に合うように取り付けと較正を済ませた後
- **`Electric` を切ったときの挙動(中立へ戻る / その場で止まる / フリー)は不明。** 船の「電源喪失時に舵が効くか」に直結するので §7 に積んだ

#### 4.10.3 Gyro(`gyro.xml`)

ノードは §4.3 の表に既載(入出力ペア `Roll`/`Stabilised Roll` 等 + bool `Auto-hover` + elec)。
**【ユーザー知見 2026-09-18】中身はブラックボックス。信頼はそこそこあるが、PID を専用に組んだほうが安定する。**
入力を繋がずに `Stabilised *` を姿勢センサー代わりに読めるかは未検証(§7)。`Stabilised Up/Down` だけ 0〜1 でヘリ用。

#### 4.10.4 車輪・スキー・履帯(27種、表のみ)

民生船では使わないので名前とノードだけ。

| 部品群 | ファイル | ノード |
|---|---|---|
| Wheel 3x3 / 5x5 / 7x7 / 9x9(各 Suspension 版あり)、Small / Medium Wheel | `wheel_advanced_*`, `wheel_small/medium` | bool `Brake` / power `RPS` / num `Variable Brake`(0〜1)/ num `Steering`(−1〜1) |
| Wheel Coaster / Large Landing Wheel | `wheel_coaster*` | bool `Brake` / num `Variable Brake`(駆動・操舵なし) |
| Tank Wheel(Small / Small-Wide / Medium / Large / Huge) | `wheel_tank_*` | bool `Brake` のみ。**同サイズ・同向きの車輪を一列に並べると履帯を形成**(原文) |
| Tank Drive Wheel(同5種) | `wheel_tank_drive_*` | bool `Brake` / power `RPS`。履帯を回すのはこれ |
| Ski / Ski (Small) | `ski*` | num `Steering`(−1〜1)。*"low friction along its length, grips laterally"* |

#### 4.10.5 Data Logger(`data_logger_bool` / `data_logger_number`)

bool/num 入力1本だけ。tooltip: *"record data for unit tests."*
**【ユーザー知見 2026-09-18】ゲーム内インベントリに出ない部品。XML 経由で置けば存在はするが、機能は不明。**
記録先が分かれば §3 の HTTP ロギングの代替になりうるが、現状は「存在だけ記載」。

#### 4.10.6 インベントリに出ない部品(定義XMLの `flags` bit 29、【推定】)

**【ユーザー知見 2026-09-18】定義XMLはあるがゲーム内インベントリに無い部品が複数あり、コピペや XML で置けば機能する。**
その例(Data Logger・旧レーダー・旧ウィンチ)を突き合わせると、**`<definition flags="...">` の bit 29(`536870912`)が共通して立っている**。
bit 29 が立つ 31 ファイル:

- `data_logger_bool` / `data_logger_number`、`01_block_static`
- 旧レーダー: `radar` / `radar_dish` / `radar_large` / `radar_huge`(現行は `radar_advanced*`、§4.1)
- 旧 Radio RX: `rx_small` / `rx_med` / `rx_large` / `rx_huge`(**現行は `_v2`**。§4.7 の「どちらがメニューに出るか」はこれで決着)
- 旧ウィンチ: `winch_electric` / `winch_a` / `winch_large_a` / `winch_huge_a`(現行は `rope_hook_winch*`)、`water_hose`
- 旧動力: `torque_gearbox` / `torque_gearbox_2` / `turbine`(Turbine Engine)、`gate_torque_add` / `gate_torque_multimeter`
- 旧ローター: `large_rotor` / `huge_rotor` / `heavy_rotor` / `heavy_rotor_large`(現行は `rotor_coaxial_*`)
- 流体: **`water_outlet`(Fluid Port の片方。`water_inlet` は表示側)**、**`fluid_filter` と `fluid_filter_v2` の両方**
- その他: `gate_train_junction` / `mineral_converter` / `passenger_seat`

> **これは4件の一致からの推定**。bit の本来の意味は不明で、`flags` の他の bit(8192 / 16384 / 1 など)も未解読。
> §4.9.9 の「inlet/outlet の差」「Fluid Filter どちらが出るか」は、この推定が正しければ「outlet は非表示」「Filter は両方非表示(現行は別名?)」になる。要実機確認。
> 【ユーザー知見】`Air Scoop Intake 1x2` は XML ごと消えている(`scoop_intake_2.xml` = 1x1 のみ残存)。

---
