### 4.15 甲板・艤装部品(ウィンチ/アンカー・照明・音・カメラ・消火)

**【一次資料 2026-09-23】`rom/data/definitions/*.xml` のうち category=4(装備・特殊)111 ファイルを読んだ結果。**
うち 60 個は Equipment / Outfit Inventory(elec 1本のみ、4.15.6)。船で使うのは約30。

#### 4.15.1 ウィンチ・アンカー(rope ノード系、14種)

**繋ぎ方が他と違う。** 2つの Anchor / Rope Node の間に **rope 型(type=8)のロジックリンクを引く**と、
そこにロープ/ケーブル/ホースが張られる。**ボクセル隣接でも `logic_node_links` の通常配線でもない第3の方式。**

**【ユーザー知見 2026-09-23】rope ノードは繋ぐ先によって張られるものが変わり、異種同士は繋がらない。**
ロープアンカー↔ロープアンカーのように**同種同士が基本**で、**ウィンチだけがロープ/ホース/ケーブルの3種を受け入れる。**
それ以外の部品は1種類のみ。→ **ウィンチを中継点にすれば種類の違う索を1台で扱えるが、アンカー同士では混ぜられない。**

| 部品 | ファイル | mass | 主なノード |
|---|---|---|---|
| Rope Anchor | `rope_hook` | 1 | rope `Rope Node` / bool `Detach Rope` |
| Fluid Hose Anchor | `rope_hook_fluid` | 1 | + fluid。**リンク1本のみ**、流体を渡せる |
| Electrical Cable Anchor | `rope_hook_composite` | 1 | + elec / comp 送受 / bool 送受 / video 送受 / audio 送受。**リンク1本のみ** |
| Sail Anchor | `rope_hook_sail` | 3 | rope 2本 |
| Small Winch | `rope_hook_winch_small` | 10 | bool `Up`/`Down` / rope / elec / fluid。**5ノードのみ** |
| Medium / Large / Huge Winch | `rope_hook_winch*` | 20 / 100 / 400 | bool `Up`/`Down` / **num `Speed`(in)** / **num `Length`(out、m)** / rope / elec / fluid / comp・bool・video・audio の送受。Large/Huge は `Detach Rope` も |
| Rope Pulley / Corner | `winch_pulley*` | 25 | **num `Rope 1 Length` / `Rope 2 Length`(out、m)** + rope 2本 |
| Fluid Hose Pulley / Corner | `winch_pulley_hose*` | 25 | 同上 + fluid 2口 |
| Electric Cable Pulley / Corner | `winch_pulley_cable*` | 25 | 同上 |

- **【ユーザー知見 2026-09-23】`Length` は巻き上げ切った状態が 0。ケーブル長がそのまま出る。**
  → **アンカー(錨)の繰り出し長は `Length` をそのまま読めばよい。** 水深と突き合わせれば着底判定が作れる
- **【ユーザー知見】bool `Up`/`Down` と num `Speed` は併用できたはず。** `Speed` は**巻取り/繰り出しの速度を変える**もので、
  **範囲が −1〜1 かは未確認**(原文は *"clamped to a relative range of -1 to 1"*。§7)
- **【ユーザー知見】Small Winch にノードが少ないのは下位互換品だからではなく、サイズが小さくてノードを置く余裕が無いから**(4.15.5)
- **ウィンチ/ケーブルアンカーは繋いだ相手と comp / bool / video / audio / elec / fluid をやりとりできる。**
  → **陸電・給水・データリンクを1本のケーブルで兼ねられる。** DECKSYS の陸電表示はこれで組む
- Pulley が出すのは**長さ**であって張力ではない。**張力を直接読む手段は無い**(Mag All の `Force` は別部品、§4.13.3)
- **旧ウィンチ(`winch_a` / `winch_electric` / `winch_large_a` / `winch_huge_a` と `water_hose`)は非掲載部品**(§4.10.6)。
  ノード構成が違うので混同しないこと

#### 4.15.2 照明(6種)

| 部品 | ファイル | mass | ノード | 備考 |
|---|---|---|---|---|
| Light | `small_light` | 1 | bool `Light Switch` + elec | 色は塗装 |
| Light (RGB) | `small_light_rgb` | 1 | **comp `Color Data`** + elec | RGB/HSV をプロパティで切替、読むチャンネルも指定可 |
| Rotating Light | `rotating_light` | 1 | bool + elec | 回転灯 |
| Search Light | `searchlight` | 20 | bool + **num `Rotation`(−1〜1 = ±0.25 turn)** + elec | 射程 100m 級(原文) |
| Small Spotlight (Block / Mounted) | `searchlight_small*` | 1 | bool + **comp `Pivot`(num1=X, num2=Y)、±0.125 turn** + elec | 射程 60m |

- **【ユーザー知見 2026-09-23】Search Light は設置向きを変えられるので、符号は取り付け側で合わせる。**
  → **Lua 側で符号を決め打ちしない。** §9.3 の「符号は規約ではなく較正」がそのまま当てはまる
- 指向の符号は §9.2 の**但し書き(センサーの指向は上向き正が自然)**の側で扱う。舵の規約と揃えようとしないこと
- **RGB 照明と RGB 指示灯(§4.12.3)だけが comp 入力**。Lua から色を出せる

#### 4.15.3 音(4種)

| 部品 | ファイル | mass | 可聴範囲(原文) |
|---|---|---|---|
| Foghorn | `foghorn` | 5 | **1500m** |
| Siren | `siren` | 5 | **3000m** |
| Megaphone Speaker (Small / Large) | `speaker_medium` / `_large` | 3 / 10 | audio 入力 + bool。§4.12.4 の Speaker より広い |
| Buzzer | `buzzer`(category 6) | 1 | 30m。§4.12.4 |

**警報音の使い分け**: 船内向けは Buzzer(30m)、船外・他船向けは Foghorn / Siren。
`Obj 1951 ALARM` で「誰に聞かせる警報か」を決めるときの選択肢。

#### 4.15.4 カメラ・消火・その他

| 部品 | ファイル | mass | ノード |
|---|---|---|---|
| Camera Small | `camera_small` | 5 | video 出力 + elec + **comp `Pivot`(±0.125 turn)** |
| Camera Medium | `camera_med` | 10 | 同上 + bool `Infrared Mode` + num `Field of View` |
| Camera Gimbal | `camera_gimbal` | 50 | video + elec + `Infrared Mode` + `Field of View` + **num `Pivot Rotation` / `Pitch Rotation`(個別 num)** |
| Camera Stabilized | `camera_gimbal_laser` | 60 | §4.5c(レーザー照射器つき) |
| **Fluid Nozzle** | `water_nozzle` | 1 | fluid `Fluid Supply` + **num `Spray Angle`(0=集束 〜 1=拡散)** + elec |
| **Fluid Cannon** | `watercannon` | 11 | **num `Turret Swivel`(ヨー)/ num `Nozzle Pitch`(ピッチ)** + fluid + elec |
| Heater | `heater` | 3 | bool + elec。*"warm either the compartment it is within or any players within a 10m radius"* |
| Transponder | `transponder` | 1 | bool `Active` + elec。Transponder Locator(§4.5)で遠距離から探せる |
| Vehicle Parachute | `parachute` | 5 | bool |
| Mounted Welder / End-Effector | `vehicle_tool_*` | 10 | bool + elec |

- **Fluid Cannon の原文は *"Filters to water only"* だが、【ユーザー知見 2026-09-23】海水も真水も使える。**
  **船底から吸い上げてもタンクに貯めても自由。** → **消火系の吸入側は海水で組んでよい**(無限の水源)。
  タンクを積む必要があるのは、海水を使いたくない用途(甲板洗浄など)だけ
- Camera は **Small/Medium が comp `Pivot`、Gimbal が num 2本**と指令の型が違う。§4.5c の Camera Stabilized も別系統
- **Camera Gimbal は mass 50、Stabilized は 60** と重い。小型船で視界が欲しいだけなら Camera Small(5)で足りる

#### 4.15.5 ノードは「置ける面」を食い合う(【ユーザー知見 + 観察】)

**【ユーザー知見 2026-09-23】部品には物理的なサイズ制限とは別に、ロジックノードが重ならないための設置制限がある。**
Instrument Panel(§4.12.1)が占有ボクセル 1x2 なのは、**入出力で1面ずつ要るから。**

定義XMLのボクセル数とノード数を突き合わせると整合する:

| 部品 | voxel | ノード数 |
|---|---|---|
| Indicator Light | 1 | 2 |
| Dial / Digital Display / Instrument Panel | 2 | 3 / 3 / 4 |
| Gauge Display | 4 | 4 |
| **Small Winch** | **2** | **5** |
| **Medium Winch** | **6** | **15** |

→ **Small Winch に `Length` / `Speed` / comp 送受が無いのは、機能を削った下位互換品だからではなく、
2 voxel にノードを置き切れないから。** 同じ理由で、**小さい部品ほど composite 1本に機能をまとめる設計になっている**
(Camera Small の `Pivot`、Spotlight の `Pivot`、RGB 照明の `Color Data`)。

> **設計上の含意**: 「小さい部品を選ぶ = 制御できる項目が減る」。
> 質量と設置スペースだけで部品を選ぶと、後から必要なノードが無いことに気づく。

#### 4.15.6 Equipment / Outfit Inventory(60種、表のみ)

`inventory_equipment_*`(44)/ `inventory_outfit_*`(14)/ `inventory_small` / `inventory_medium`。
**全部 elec 1本のみ**(一部の Outfit は fluid `Oxygen` を持つ: diving / scuba / space / firefighter SCBA)。
**Lua から中身を知る手段は無い。** 置くだけの部品。

---
