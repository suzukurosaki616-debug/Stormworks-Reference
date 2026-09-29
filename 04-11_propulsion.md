### 4.11 推進系部品(プロペラ・電動モーター・一体型エンジン・動力計測)— 定義XMLの読解結果 + ユーザー知見

**【一次資料 2026-09-18】`rom/data/definitions/*.xml` のうち category=3(推進)64 ファイルを機械抽出して読んだ結果。**
列車 21・ローター 12・ロケット 9 は船に無関係なので 4.11.6 の表だけ。一体型エンジンのノード表は §4.9.6 に既載。

> 定義XMLに **`force_emitter_max_force` / `engine_max_force`** という数値がある。JP Wiki(PROPULSION)の「推力」「出力」欄と完全一致するが、
> **JP Wiki が XML を転記しただけの可能性が高く、独立の裏付けにはならない**(§0b 規則5)。推力との対応は未検証。
> **サイズ間の比率の目安**としてだけ使い、絶対値を N と読まないこと。

#### 4.11.1 プロペラ(水中用、7種)

| 部品 | ファイル | mass | `force_emitter_max_force` | ノード |
|---|---|---|---|---|
| Small Propeller | `propeller` | 3 | 20,000 | power `RPS` |
| Large Propeller | `large_propeller` | 20 | 100,000 | power `RPS` |
| Giant Propeller | `giga_prop_small` | 100 | 200,000 | power `RPS` |
| Small Pitchable Propeller | `propeller_pitch_small` | 10 | 20,000 | power `RPS` + num `Pitch`(−1〜1) |
| Pitchable Propeller | `propeller_pitch` | 40 | 80,000 | 同上 |
| Large Pitchable Propeller | `propeller_pitch_large` | 100 | 160,000 | 同上 |
| Azimuth Thruster | `azimuth_thruster` | 4 | 40,000 | power `RPS` |

- 【JP Wiki】直径: Small 3 / Large 5 / Giant 9 ブロック、Azimuth 3。**Large Propeller は見た目と設置判定がずれている。** Azimuth Thruster は動力接続部と推進方向が 90°
- 原文(全種): *"This propeller will only operate underwater."* **【ユーザー知見 2026-09-18】水上では一切効かない。** 0/1 であって徐々に落ちるのではない
- 原文: *"Power received from an engine affects the forward and backward force generated."* — **回転方向で前後進する**
- **【ユーザー知見】サイズの使い分け: Small = 小型船、Large = 中〜大型船。Giant は大きすぎて用途が謎(潜水艦なら?)**
- **【ユーザー知見】後進の作り方は3通りあり、どれも実用**:
  1. エンジン + 固定ピッチ → **Gearbox の逆転**(現行の Gearbox は `modular_engine_gearbox_*`、§4.9.6。旧 `torque_gearbox` は非掲載 §4.10.6)
  2. 可変ピッチ → `Pitch` を負にする(0 で推力ゼロ、−1 で後進、と推定。未検証)
  3. 電動モーター → `Throttle` を負にする(4.11.2)
- **Azimuth Thruster**: 原文 *"can be attached to a robotic pivot to provide steering as well as propulsion."*
  **【ユーザー知見】動力伝達のある可動部(Robotic Pivot (Power) 等、§4.3c)を経由して動力が届く。** サイドスラスター/ポッド推進はこれで作る

#### 4.11.2 電動モーター(3種)

| 部品 | ファイル | mass | `electric_magnitude` | ノード |
|---|---|---|---|---|
| Small Electric Motor | `motor_small` | 5 | 1.2 | power `RPS`(出力)/ elec `Electric` / num `Throttle` |
| Medium Electric Motor | `motor_medium` | 100 | 1.1 | 同上 |
| Large Electric Motor | `motor_large` | 400 | 1.05 | 同上 |

- **【ユーザー知見 2026-09-18】`Throttle` は −1〜1。負で逆転する。**(原文は範囲を書いていない)
- **【ユーザー知見】効率が極端に悪い。** 昔「発電機とモーターを直結すると電力が黒字になる」バグがあり、その修正で
  モーターの消費が大幅に引き上げられた。**Large はバッテリーでは到底足りず、発電機直結が前提。** 電動推進を組むなら
  「バッテリー航行」を設計に入れないこと(消費率の実測は §7)
- `force_emitter_max_force` は 2 / 60 / 250 とプロペラと桁が違う。**【JP Wiki PROPULSION】トルク 3.0 / 61.0 / 251.0、無負荷時 RPS ±19.9**(XML 値 +1 に一致。XML 転記の疑いあり)

#### 4.11.3 一体型エンジン(Small / Medium / Large、ノード表は §4.9.6)

- `Throttle` 0〜1、`Starter` bool、power `RPS` 出力、num `RPS` / `Temperature` 出力、燃料/空気/冷却/排気の fluid
- `engine_max_force`: Small 20,000 / Medium 60,000 / Large 200,000(比率の目安)
- **【ユーザー知見 2026-09-18】アイドリングは存在しない。** 制御側で最低スロットルを作る
- **【ユーザー知見】RPS の上限は既定 20、最大 100**(プロパティ。【JP Wiki】設定範囲 5〜100)。**ジェット(§4.6)と違い回転数では壊れない**
- 【JP Wiki クラフトガイド/エンジン系】**RPS 2 以下でエンスト**。停止時 RPS 0。始動は `Starter` on → 一定 RPS を超えたら off にして電力を節約。
  **電力を切ってもエンジンは止まらない**(電力はスターターだけが使う)。止めるにはスロットル 0。**吸気口が水中・密閉空間だと性能低下/停止。**
  燃料に水・ジェット燃料・原油が混ざると性能低下。冷却水ポンプは内蔵(配管を繋ぐだけで循環)、海水でも可
- 【JP Wiki】トルク: Small 11 / Medium 31 / Large 101
- **【ユーザー知見】`Temperature` が 120 ℃ あたりを超えると爆発炎上する(【JP Wiki】115 ℃)。温度は回転数と外気温の両方で決まる。外部の熱の影響を受けるので、
  機関室で1基燃えると連鎖して全部爆発しがち。** → 温度監視は警報バス(`Obj 1951 ALARM`)の必須項目。冷却ループは省略できない
- **燃料消費モデル【ユーザー知見 2026-09-23】: 回転数だけが高くても、スロットルだけが高くても燃料は減らない。**
  **両方を引数にした何らかの関数**と思われる。**関数形は未特定**(§7)。モジュラーエンジンはこの限りではない(§4.14)
  → **航続距離計は「スロットルから燃費を推定する」方式では作れない。** タンク残量の実測差分(§4.9.2、毎 tick・0.1 L 以下)から
  消費率を出すほうが確実
- `turbine.xml`(Turbine Engine、`Throttle` −1〜1)は非掲載部品(§4.10.6)

#### 4.11.4 動力計測・その他

- **Torque Meter**(`torque_meter`、mass 4): power `RPS` 通過 + num `RPS` / num `Torque` 出力。
  **【ユーザー知見 2026-09-18】`Torque` は回転と無関係に「接続先の回転抵抗のようなもの」を固定値で出すだけ。
  回転数を取る以外の用途はない。** RPS 計として使う
- **Torque Crank**(`torque_crank`): 人力クランク。power `RPS` 1本。非常用の手回し
- **Gearbox / Clutch**(§4.9.6 のノード表)【JP Wiki MECHANICS】: ギア比 1:1 / 6:5 / 3:2 / 9:5 / 2:1 / 5:2 / 3:1、**−1:1 で逆転**。`Gear Switch` off = Ratio 1 / on = Ratio 2。
  **1台につき 5% の伝達損失。Ratio 1 だけでも電力が要る。** 1x1 / 3x3 / 5x5 は性能同じ。
  Clutch の `Clutch Pressure` は **0〜0.3 は伝達せず、0.3〜1.0 で段階的に伝達**
- **Robotic Pivot (Power)**【JP Wiki】: 動力を両側に通す。可動 ±90°(0.25 turn)、指令 −1〜1(§4.3c と一致)
- **Aircraft Propeller**(`aircraft_propeller`、70,000): power + num `Collective`。航空機用
- **Ducted Fan** Small / Large(2,000 / 10,000): power のみ。空気中用

#### 4.11.5 動力(power、type=2)ノードの注意

- **動力はパイプ(`trans_*`、§4.18.2)で引き回す。流体と同じ部品。**
- power ノードは `logic_node_links` に**出てこない**(§5.7)。パイプがボクセル隣接で繋がるため。
  **Lua から直接読み書きできない**ので、回転数が欲しければ Torque Meter かエンジンの num `RPS` を経由する
- 新しめの部品(Large Engine 等)は power ノードの `mode=` が無い。双方向と考えてよい(§5.6b)

#### 4.11.6 船に無関係な部品(表のみ)

| 部品群 | ファイル | ノード |
|---|---|---|
| Rotor (Light / Small / Large)、Rotor Propeller (Small / Large)、各 End | `rotor_coaxial_*` | num `Roll` / `Pitch` / `Collective` + power `RPS`(End 以外は Power Connection A/B の2本) |
| Rotor (Tail) | `tail_rotor` | power `RPS` + num `Yaw` |
| Large / Huge / Heavy / Heavy Large Rotor | `large_rotor` 等 | 旧型、非掲載(§4.10.6) |
| Train Wheel Assembly 各種(17) | `train_wheels*` | elec / num `Brakes` / power `RPS` / bool `Wheel Slip`(out) / bool `Wheel Release` |
| Train Wheel Drive Piston S/M/L | `train_wheels_piston*` | fluid `Steam In`×2 / `Steam Out`×2 / bool `Reverser` |
| Liquid Fuel Rocket S/M/L | `liquid_rocket*` | fluid + num(6本) |
| Solid Rocket Booster / Fuel 各種 | `solid_rocket_*` | bool / num / comp |

---
