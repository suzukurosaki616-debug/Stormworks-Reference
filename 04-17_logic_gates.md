### 4.17 ロジックゲート(37種)

**【一次資料 2026-09-23】`rom/data/definitions/gate_*.xml` 37 ファイルを読んだ結果。**
**名前どおりの挙動で、未知の要素は無い。** MC 内で Lua を書く本リポジトリでは大半が不要なので、
**「Lua の外に置く価値があるもの」と「XML 由来の注意点」だけ**を本文にし、残りは表。

#### 4.17.1 Lua の外に置く価値があるもの

| 部品 | ファイル | 理由 |
|---|---|---|
| **PID Controller** | `gate_pid_controller` | bool `Active` / num `Process Variable` / num `Setpoint` → num `Control Output`。**P/I/D はプロパティ。** §4.10.3 のとおり Gyro より素直で、**機関の RPS 追従や喫水保持は MC を1個増やすより速い**。`Active` を off にすると内部値がリセットされる(積分のワインドアップ対策に使える) |
| **Memory Register** | `gate_float_register` | num + bool `Set` / bool `Clear` → num。**MC の外に値を残せる。** 電源を切っても保持されるなら設定値の保存に使えるが**未検証**(§7) |
| **Blinker** | `gate_bool_blink` | on/off の周期がプロパティ。**警報の鳴動・点滅を Lua の tick 計算から追い出せる**(§4.12.4 の一発音ブザーと組める) |
| **Capacitor / Delay** | `gate_bool_capacitor` / `gate_bool_delay` | 充放電時間・遅延時間がプロパティ。**「N 秒続いたら」「N 秒後に切る」を Lua のカウンタ無しで作れる** |
| **Up/Down** | `gate_up_down` | bool 2本 → num −1〜1。**ボタンを連続値に変える。** §4.13.1 の Throttle Lever と同じ役割を任意のボタンで |
| **Push to Toggle** | `gate_push_to_toggle` | Push Button をトグル化。§4.13.1 の Toggle Button を使えば済むが、既設の配線を変えずに済む |
| Function (1 / 3 inputs) | `gate_function_small` / `_large` | 独自の式を書ける。**Lua があるなら不要** |

> **判断基準**: 「MC を増やすほどでもない単機能」と「Lua の tick 管理を減らせるもの」だけ外に出す。
> 分岐・演算は Lua で書いたほうが読みやすく、§2 の composite 1本で済む。

#### 4.17.2 XML 由来の注意点

- **Trigonometry(`gate_float_sin`)は turn 単位。** 原文: *"Sin, cos and tan accept inputs measured in turns.
  Asin, acos and atan output values measured in turns."*
  → **Lua の `math.sin` は radian(§3)。ゲートと Lua を混ぜると単位を取り違える。**
  §4.2 の Physics Sensor(radian)、§4.3 のコンパス・Tilt(turn)と合わせて、**単位は部品ごとに確認する**
- **Divide は 0 除算で bool `Error` を出し、出力は 0 になる。** 落ちない。Lua 側の `0/0 = nan` とは挙動が違う(§6)
- **Clamp / Threshold / Exponent / Constant / Counter の上下限・定数はプロパティ。** 配線からは見えないので、
  **実機で開かないと値が分からない。** 設計文書に残すこと
- **Counter は `Speed`(num)を入れ続けると増え続ける。** 上限が無いので、**Dial に直結すると一周する**(§4.12.2)
- **SR Latch は Set/Reset 同時で Reset 優先、JK Flip-Flop は同時でトグル。** 原文どおりで違いはここだけ
- **`gate_train_junction`(Train Junction Controller)の tooltip は Push to Toggle のコピペ**(定義XMLの誤り)。
  ノード構成も Push to Toggle と同一。**非掲載部品**(§4.10.6)
- **`gate_torque_add`(Power Add)/ `gate_torque_multimeter`(Power Meter)も非掲載**(§4.10.6)。
  現行で動力を測るのは Torque Meter(§4.11.4)

#### 4.17.3 一覧(残り)

**bool 系**: And / Or / Not / Xor / Constant On Signal / SR Latch / JK Flip-Flop / Blinker / Capacitor / Delay / Push to Toggle
**num 系**: Add / Subtract / Multiply / Divide / Modulo / Abs / Exponent / Numerical Inverter(×−1)/ Clamp /
Constant Number / Counter / Counter (Ping Pong) / Trigonometry / Function (1/3 inputs) / Memory Register / Up/Down
**比較・分岐**: Greater-than / Less-Than / Threshold Gate / Numerical Junction(1入力→2出力、非選択側は 0)/
Numerical Switchbox(2入力→1出力)
**制御**: PID Controller
**動力**: Power Add / Power Meter(いずれも非掲載)
**その他**: Train Junction Controller(非掲載)

---
