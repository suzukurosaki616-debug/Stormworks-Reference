### 4.13 機構部品(ボタン類・ドア/区画・コネクタ・直動・安定装置)

**【一次資料 2026-09-23】`rom/data/definitions/*.xml` のうち category=2(機構)61 ファイルを読んだ結果。**
ピボット/砲塔リングは §4.3c、Gearbox / Clutch は §4.11.4 に既載なので除く。

#### 4.13.1 ボタン・レバー・キーパッド(9種)

| 部品 | ファイル | ノード |
|---|---|---|
| Push Button / Push Button (2 Sided) | `button_push`, `button_push_2side` | bool `Pressed`(out)/ bool `External Input`(in)/ elec |
| Toggle Button / Toggle Button (2 Sided) | `button_toggle`, `button_toggle_2side` | bool `Toggled`(out)/ `External Input` / elec。**初期状態はプロパティ** |
| Key Button | `button_key` | bool `Activated`(out)/ `External Input` / elec。**押し続ける時間がプロパティ**。もう一度触れると即解除 |
| Lockable Button | `button_lock` | bool `Toggled`(out)/ **bool `Unlock`(in)** / elec |
| Throttle Lever | `button_throttle_lever` | **num `Throttle`(out、−1〜1)** / bool `Up` / bool `Down`(in)/ elec |
| Small Keypad | `button_keypad_small` | num `Output`(out)/ `Backlight` / elec |
| Large Keypad | `button_keypad_large` | num `Output A` / `Output B`(out)/ **bool `Pulse`(out)** / `Backlight` / elec |

- **ボタン類は全部 `External Input` を持つ。** 原文: *"allowing you to chain multiple buttons together to unify their outputs."*
  → **同じ操作のボタンを艦橋と機関室の両方に置いて、出力を1本にまとめられる。** Lua 側は1本読むだけでよい
- **Lockable Button の `Unlock`**: on のときだけ人が触れる。**Lua から人の操作を禁止できる唯一のボタン。**
  危険操作(消火起動・アンカー投下・区画隔離)のインターロックに使う
- **Throttle Lever**【ユーザー知見 2026-09-23】: **手を離しても戻らない。** 速度設定はプロパティで 1〜100% だが、
  **100% が 1 tick あたりいくつかは不明**(§7)。**座席の Axis と違って「戻る/留まる」の設定差が無い**ので、
  操作者の設定に依存しない指令源が要るならレバーのほうが安全
- **Large Keypad**: 原文 *"A short pulse will be emitted from the keypad's pulse node when the confirm button is clicked"* —
  **確定時に1発パルス、値は継続出力。** 「入力されたか」をエッジで拾えるので Lua 側でエッジ検出を書かなくてよい
- **【ユーザー知見】Large Keypad の座標入力は「先にマップに WP を置く → キーパッドを開いてボタンを押す」**で
  その座標が入る。**入る値は GPS と同じ地図座標系**(ユーザーの言い方では Z-up。§4.3 の GPS は Number1=東 / Number2=北)。
  → **NAVAID の目的地入力はこれで足りる。** Lua 側でマップのタッチ座標から逆算する必要がない

#### 4.13.2 ドアと区画(11種)

| 部品 | ファイル | ノード |
|---|---|---|
| Sliding Door (Electric) / Sliding Hatch (Electric) | `door`, `hatch` | bool `Open/Close` + elec |
| Sliding Door / Hinged Door / Sliding Hatch / Hinged Hatch(手動) | `door_manual*` | **bool `Lock` のみ**(電力不要) |
| Hinged Dock Door / Hatch | `door_dock_large`, `door_dock_small` | `Open/Close` / `Magnet Toggle` / **`Connected`(out)** / On/Off・Number・Composite の送受 / elec |
| **Door Frame Controller** | `door_frame_controller` | **bool `Lock Seal`(in)/ bool `Seal State`(out)**。電力不要 |

**Door Frame Controller が区画監視の要**。原文: *"lets you lock the door when closed, and check whether the door is
closed enough to be forming a seal. Only one controller should be placed per frame."*

- **【ユーザー知見 2026-09-23】外枠と内枠のサイズが合致していて、ある程度近ければ sealed になり、水が入らない。**
  → **`Seal State` は「そのドア枠が塞がっているか」であって「部屋全体が密閉か」ではない。**
  区画の密閉を判定したいなら、**その区画に属するドア枠の `Seal State` を全部 AND する**設計になる
- 手動ドアは `Lock` しか持たない。**開閉状態は取れない**ので、状態を知りたい区画には電動ドアか Door Frame Controller を使う
- §4.9.4 の「密閉区画(自作タンク)」の判定規則とは別物。あちらは流体側の話

#### 4.13.3 コネクタ(8種)— 艇間で何を渡せるか

| 部品 | ファイル | 渡せるもの |
|---|---|---|
| Small Connector | `connector_small` | comp / video(+ `Magnet Toggle` / `Connected`) |
| Large Connector | `connector_large` | **bool / num / comp / video / power / fluid / elec** — 全部 |
| Electric Connector | `connector_electric` | elec + comp / video。**近づけば自動で繋がる**(トグル不要、`Release Connector` で切る) |
| Fluid Connector | `connector_water` | fluid + comp / video。同じく自動 |
| Torque Connector | `connector_torque` | power + comp / video。同じく自動 |
| Hinge Connector | `connector_hinge` | comp / video。**長辺で繋がりその軸でヒンジする** |
| Sliding Connector Gripper | `connector_slider_gripper` | bool `Release Connector` / bool `Brake` のみ |
| Mag All | `magall` | bool `Magnet Toggle` / `Connected` / **num `Force`(out)** / elec。**応力 200 で外れる**(原文) |

- **`Magnet Toggle` 型(Small / Large / Hinge / Mag All)は両方 on で繋がる。
  `Release Connector` 型(Electric / Fluid / Torque)は近づけば勝手に繋がり、on で切る。** 論理が逆なので注意
- **陸電・給油の接続状態は `Connected` bool で取れる。** DECKSYS の陸電表示はこれ
- Mag All の `Force` は**現在の応力が num で出る**。切断前に警報を出せる

#### 4.13.4 直動(5種)

| 部品 | ファイル | ノード |
|---|---|---|
| Linear Track Base | `linear_base` | bool `Down` / `Up`(in)/ **num `Slider Position`(out)** / elec / **power / fluid を通せる** |
| Compact Linear Track Base | `linear_compact_base` | num `Slider Speed`(in)/ elec |
| Linear Track Head | `linear_head` | power / fluid(通すだけ) |
| Pneumatic Piston | `linear_matic_a` | num `Target Position`(in)/ **num `Rod Position`(out、−0.5〜0.5 m)** / elec / **fluid 駆動** |
| Pneumatic Piston(従) | `linear_matic_b` | fluid のみ |

- Linear Track Base は **bool 2本(Up/Down)で動かす位置指令ではない方式**。Compact 版は速度指令(num)。**指令の型が違う**
- Pneumatic Piston は原文 *"can expand by a length of 1 metre"*、位置は −0.5〜0.5 m で読める。**空気圧が要る**(fluid)
- Linear Track / Robotic Hinge は **power と fluid を通せる**(§4.9.8 と同じ話)

#### 4.13.5 安定装置 — Reaction Wheel と Keel

**Reaction Wheel**(`gyroscopic_stabilizer` / `_small` / `_large`、mass 15 / 2 / 80): num `Rotation` + elec のみ。
原文: *"can be wired directly to an aligned angular rotation sensor to stabilize its rotation."*

- **【ユーザー知見 2026-09-23】船にも積めるが出力的に現実的ではない。** 効かせようとすると大型を大量に積む規模になり、
  **パッシブではないので電力も食う。効率が悪い最終手段。**

**Keel**(`keel_small` / `_medium` / `_large`、category 1、mass 500 / 1000 / 2000): **ロジックノードを持たない。**
原文: *"The keel resists rolling forces while submerged, helping to keep a boat upright and stable. The keel also
resists lateral sliding forces, causing the majority of a boat's movement to be in the direction of the keel axis."*

- **【ユーザー知見 2026-09-23】ロール安定はキールを積むのが無難。** パッシブで電力を食わない
- **横滑りも抑える**(原文)ので、**旋回の挙動そのものが変わる。** 操舵制御を組む前にキールの有無を決めること。
  Lua からは何も見えない/触れないが、**制御の前提として効く**

> **HULLSYS のヒール制御を「能動制御」で組む前に、キールで足りないかを先に見ること。**
> Reaction Wheel は電力収支(§4.9.7)を圧迫する。

---
