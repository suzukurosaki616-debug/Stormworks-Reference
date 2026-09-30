### 4.3 GPS / コンパス / 高度計 / 距離センサー

以下、**ノードのラベルとdescriptionは `rom/data/definitions/*.xml` の一次資料**。ノードの並び順＝部品上のNumber番号の順。

| 部品 (定義ファイル) | ノード構成(定義順) |
|---|---|
| GPS Sensor (`gps_sensor`) | 出力 `X Coordinate` / `Y Coordinate`、入力 `Electric`。原文: *"The x coordinate of **this component** on the world map."* → **パーツ自身の座標**(下記)。Xは東西(右が正)、Yは南北(北が正)でワールドマップと一致 |
| Altimeter (`altimeter`) | 出力 `Altitude` のみ。原文: *"The measured altitude of **the component** in metres above sea-level."* → **これもパーツ自身の高度**。電力不要 |
| Compass Sensor (`compass_sensor`) | 出力 `Compass Reading`、入力 `Backlight`(bool) / `Electric`。原文: *"a number value representing **the turn that must be made for it to face north**"* — 「北からの方位」ではなく**「北を向くために必要な回転量」**である点に注意(符号が逆になりうる)。ノード側の原文(2026-09-12 確認): *"The angle measured in turns that the needle is rotated from the **white arrow** on the display."* — **基準は部品面の白矢印**。取り付け向きで読みが変わる |
| Tilt Sensor (`rotation_sensor`) | 出力 `Tilt` のみ。*"The measured tilt relative to the horizon."* 0.25turn = +90°。基準0は設置時の青矢印方向。**ファイル名は `rotation_sensor.xml`**(表示名と違うので検索時に注意) |
| Linear Speed Sensor (`linear_speed_sensor`) | 出力 `Linear Speed` のみ。*"The sensor's linear speed in m/s."* スカラー |
| Angular Speed Sensor (`angular_speed_sensor`) | 出力 `Angular Speed` のみ。原文 *"in **rotations per second**"* / *"about the component's **y axis**"*(2026-09-12 確認)。**単位が一次資料で保証されている角速度源**。Physics Sensor ch10-12 も turn/s(ユーザー 2026-09-30、§4.2) |
| Distance Sensor (`distance_sensor`) | 出力 `Distance`、入力 `Electric`。最大500m、未検出時500m |
| Laser Distance Sensor (`laser_distance_sensor`) | 出力 `Distance`、入力 `Electric` / `Active`(bool) / `Wavelength`(number) / **`Pivot`(type=5)**。最大4000m。**Pivotノードでレーザーの向きをcompositeで指令できる**: *"(Value 1 : Pivot X) (Value 2 : Pivot Y)"*。`Wavelength` で波長を指定でき、対の Laser Point Sensor と組にできる |
| Laser Point Sensor (`laser_point_sensor`) | 入力 `Electric` / `Wavelength`、出力 `Data Output`(type=5)。*"On/Off channel 1: is laser detected. Number channel 1, 2: X, Y position within the sensors **120 degree field of view**."* |
| Impact Sensor (`impact_sensor`) | 出力 `Output`(bool)のみ。急激な速度変化で on |
| Gyro (`gyro`) | 入出力ペア: `Roll`/`Stabilised Roll`、`Pitch`/…、`Yaw`/…、`Up/Down`/…(いずれも -1〜1)。入力 `Auto-hover`(bool) / `Electric`。`Stabilised Up/Down` のみ **0〜1** |

**【一次資料で判明】Laser Distance Sensor は `Pivot` 入力で照準できる。** 距離計を固定設置したまま任意方向を測距できるため、光学照準・手動測距系の設計自由度が大きく上がる(`Obj 1872 AAFCS` のマニュアルモード等)。

**座標変換の注意**: Camera StabilizedはYを高度(Altimeter相当)、Zを南北(GPSのY相当)として扱うため、他センサーと軸の意味が異なる。**GPSは (Number1=東, Number2=北) の2次元**であり、Y-up系(X=東, Y=高度, Z=北)で統一する場合は **GPSのNumber2 → Z、高度計 → Y** という入れ替えが要る。この変換は入力段1か所に閉じ込めること。

**【実機検証済み】GPSはパーツ自身の座標を返す**(`Obj 1872 AAFCS`の設計時にユーザーより確認)。ビークルの原点や重心ではないため、離れた位置に置いた2つのGPSは異なる値を出す。

- 帰結: **センサーと武装が機体上で離れている場合、それぞれの近くにGPSを置けば視差(パララックス)補正の計算そのものが不要になる**。逆に単一GPS構成なら、各部位への取付オフセットを機体ローカルで持ち、機体姿勢で回してworldに載せる補正が要る
- 実用上は「見た目のために機首/砲側にGPSを増設したくない」という理由で単一GPS+オフセット方式を選ぶことが多い。**どちらでも成立するので設計選択の問題**
- **【実機検証済み】高度計(Altimeter)も同様にパーツ自身の位置を返す**(同上)

### 4.3b 砲弾の挙動(【実機検証済み】/ コミュニティ実測値)

`Obj 1872 AAFCS` の設計時に確定した事項。詳細と導出はそのプロジェクトの設計記録にある(非公開)。

- **重力は全砲共通で 30 m/s² = 0.5 m/s per tick**(tick = 60Hz と整合)
  - **【重要・一次資料で確認】この30は「砲弾の重力」であって、ビークルの重力ではない。ビークルは 10 m/s²。**
    出典は `sdk/data/game_constants.xml` の `gravity="10.0"`(`Obj 9901 SWSIM/swsim/params.lua` も
    同じ値を`rom`信頼度で持っている)。**ミサイル・航空機・車両はすべてビークルなので10を使うこと。**
    ここを取り違えると重力補償が3倍になり、常に過大な機首上げ(=余分な迎角と抗力)が乗る。
    `Obj 1121 BVRAAM` が実際に踏みかけた
- **抗力は毎tickの速度比例減衰 `v ← v·(1−k)`。線形なので厳密な閉形式解が存在し、数値積分は不要**
- 結果として**仰角によらない水平到達距離の絶対上限** `d_max = v0(1−k)/(60k)` が存在する。**Stormworksの対空砲戦は1〜2kmの近距離戦になる**

**【一次資料で確認済み】初速・抗力・発射間隔の全表**(`stormworks64.exe` の `.data` にある静的テーブル2本を直読。オフセットと生データは `Obj 9901 SWSIM` の手元資料。非公開)。
**コミュニティ実測値だった k と d_max は、この直読値と完全に一致した**(下表の d_max はすべて上式で再計算したもの)。

| weapon_class | 砲 | **初速 v0 [m/s]** | **抗力 k** | **d_max [m]** | 発射間隔 [tick] | スピンアップ [tick] |
|---|---|---|---|---|---|---|
| 0 | Machine Gun | 800 | **0.025** | **520** | 2 | 0 |
| 1 | Light Autocannon | 1000 | 0.02 | 816.7 | 4 | 0 |
| 2 | Rotary Autocannon | 1000 | 0.01 | 1650.0 | 1 | **30** |
| 3 | Heavy Autocannon | 900 | 0.005 | 2985.0 | 16 | 0 |
| 4 | Battle Cannon | 800 | 0.002 | 6653.3 | 75 | 0 |
| 5 | Artillery Cannon | 700 | 0.001 | 11655.0 | 250 | 0 |
| 6 | Bertha Cannon | 600 | 0.0005 | 19990.0 | 600 | 0 |

- **Machine Gun の k = 0.025 は今回新たに判明**(従来「不明」だった項目)。d_max はわずか520mで、全砲中で最短
- Rotary の行にだけ立っている **30 tick = 0.5秒** が、下記の「トリガー投入から発砲まで約0.5秒」の実機計測と一致する。**スピンアップはこの1行がすべて**
- **【ユーザー談 2026-09-29】風は「対気速度に抗力が掛かる」だけ。** 横風があれば、そのぶん空気との相対速度が増えて抗力を受ける。
  上の抗力の式を対気速度に掛ければよい: `v ← w + (v − w)·(1−k)`(w = 風)。線形のままなので閉形式は残る。
  `Obj 1872 AAFCS` の `BallisticSolver` はこのモデルで実装済み(対気の相対量だけで解くので、Wind Sensor の見かけ風をそのまま使える。
  詳細は AAFCS の設計記録。非公開)。
  横流れの目安(手計算・未検算、Battle Cannon・横風 10 m/s): 1000m で約1m、2000m で約5m。鉛直方向の風があるかは不明
- **【実機検証済み】砲弾は発射母体の速度を100%継承する**。走行間・航行間射撃では初速ベクトルへの加算が必須
- **【実機検証済み】当たり判定はtickごとの線分(swept)。すり抜けは発生しない**
- **【実機計測】Rotary Autocannon (`gun_v`) はトリガー投入から発砲まで約0.5秒(30tick)、停止も約0.5秒。** **スピンアップ専用の入力ノードは存在せず `Trigger` しかない**ため、初弾の遅れを詰めたい場合は交戦直前にトリガーを断続投入して回転を維持するしかない(数発が的外れに出て発砲音も不自然になる)。TOF 1.5秒の対空交戦では0.5秒は3割の遅れに相当し無視できない
- **【一次資料 2026-09-30】砲のロジックノードは2系統に分かれる**(ROM `gun_*.xml`):
  - **給弾式**(Machine Gun `gun_xs` / Light Autocannon `gun_s` / Heavy Autocannon `gun_m` / Rotary Autocannon `gun_v`):
    `Trigger` = *"feeds, loads, and fires"* — **トリガーが給弾・装填・発射を全部兼ねる**。トリガーを止めると装填も止まる
  - **尾栓式**(Battle Cannon `gun_l` / Artillery Cannon `gun_xl` / Bertha Cannon `gun_xxl`): `Trigger` = *"fires a loaded shell"* のみ。
    **`Open Breech` 入力で尾栓を開け、フィーダーで砲弾を送り込み、尾栓を閉じると撃てる。装填にトリガーは関係ない**(ユーザー 2026-09-30)
  - どちらも `Loaded` 出力(撃てる弾が入っていると true)を持つ
- **【ユーザー確認 2026-09-30】`Fuse Timer` が 0 なら着発。**
- **【実機検証済み】時限信管(`Fuse Timer`)は発射時にセットされる。** `Fuse Timer` ノードを持つのは Heavy Autocannon 以上のみ(Machine Gun / Light Autocannon / Rotary Autocannon には無い = 直撃必須)
- 砲弾はDespawn Timer(tick)とDespawn Speed(50 m/s。終端落下速度 `0.5/k` が50以下になるMG/LAC/Rotaryでのみ実効)で消滅する
- **【ユーザー提供 2026-09-29】Machine Gun: 初速 800 m/s、重力 30 m/s²(0.5 m/s/tick)、Despawn Timer 300 tick、Despawn Speed 50 m/s。** 抗力は上の表の k = 0.025(当時は不明としていた)
  Battle Cannon も初速は約 800 m/s(上の到達距離の上限 6653m と k=0.002 から逆算)で、違うのは抗力だけ
- **【ユーザー確認 2026-09-29】弾種を変えても弾道プロファイル(初速・抗力・重力)は変わらない。** 弾道パラメータは砲ごとに1組でよい
- **【ユーザー談 2026-09-29】爆発の範囲ダメージは、間にあるものをすべて無視する球。** 壁や装甲の陰に隠しても防げない。
  **防ぐ手段は装甲を分厚くすることだけ**。榴弾のダメージ半径: **Battle Cannon 10 ブロック(2.5 m)/ Heavy Autocannon 6 ブロック / Light Autocannon 2 ブロック**。
  普通の大きさの戦車では耐えるのは現実的でない(大型化した戦車なら、という程度)。弾頭部品の加害半径は `04-8a_missile_warhead.md`
  - 帰結: 同じ車体の2部品が1発で同時にやられないようにするには、**半径の2倍(20 ブロック = 5 m)以上**離す必要がある
    (2部品の中間で炸裂すると両方が半径に入る)。普通の戦車の砲塔幅ではまず足りず、離して置いても「同時にやられる確率が下がる」だけ

### 4.3c ピボット/砲塔リング(【一次資料で確認済み】)

`rom/data/definitions/multibody_*.xml` を直接読解。**位置指令できる部品は可動範囲が±90°しかない**という制約が設計に強く効く。

| 部品 (定義ファイル) | 可動範囲 | 指令 | max_motor_speed | max_motor_force | Current Rotation出力 |
|---|---|---|---|---|---|
| Robotic Pivot (Power) (`multibody_robotic_pivot_01_a`) | **±0.25turn(±90°)** | **位置**(`Rotation Target`) | 5 | 200 | あり(-0.25〜0.25) |
| Robotic Hinge (`multibody_robotic_hinge_01_a`) | ±0.25turn | **位置** | 5 | 400 | あり |
| Velocity Pivot (`multibody_velocity_pivot_a`) | **無制限** | 速度のみ | **属性が空=無制限** | 1000 | あり(0〜1turn) |
| Compact Velocity Pivot (`multibody_compact_pivot_velocity_a`) | 無制限 | 速度のみ | 空=無制限 | 100 | **なし** |
| Turret Ring (Medium) (`multibody_turret_medium_a`) | 無制限 | 速度のみ | 1 | 1000 | あり |
| Turret Ring (Large) (`multibody_turret_large_a`) | 無制限 | 速度のみ | 0.5 | 2000 | あり |

- **180°を超える旋回が必要な軸ではRobotic系が使えない。** 速度指令型を使い、位置制御を自前で組むことになる
- **【ユーザー確認 2026-10-01】Robotic Pivot の `Rotation Target` は ±1 の正規化値**(±1 = 可動域の端 ±0.25 turn)
- **【ユーザー談 2026-10-01】速度指令型ピボットの実際の角速度は、上に重い物を載せると落ちる(と見込まれる)。** 入力と角速度の比を固定値として信用しない。
  フィードフォワードの倍率はずれる前提で、残りはフィードバックに拾わせる
- **【ユーザー確認 2026-10-01】`Current Rotation` は、+ に回すと + になる**(指令の符号と実測の符号が一致する)
- **【ユーザー確認 2026-10-01】ピボットの既定の回転の向きは、上から見て反時計回り。** 反転設定で逆にできる(規約は §9.2b)
- **【ユーザー実測 2026-10-01】Turret Ring (Large) は入力 1 で 0.5 turn/s、入力 10 で 5 turn/s。** 入力に比例し、1 で頭打ちにならない。
  **定義の `max_motor_speed="0.5"` と一致する** → この属性は「入力 1 あたりの turn/s」と読める(Medium の 1 は 1 turn/s と推定。未測定)。
  Velocity Pivot の「1 を超えると速度上限を超えて加速し続ける」挙動(上記)とは違う。重い物を載せたときに落ちるかは未測定
- **`Compact Velocity Pivot` は `Current Rotation` を持たないので閉ループが組めない**
- **【実機情報】Robotic Pivot (Power) は電気だけで動く。`RPS` ノードは駆動用ではなく、接続先へ機械動力を伝達するためのもの**(ユーザー 2026-09-28。以前ここに「RPS 配線が必要」と書いていたのは誤り)。トルクは速度指令型より小さい(200 対 1000)
- **【実機情報】Velocity Pivotは角速度計で約10(rotations/s)まで出る**(ユーザー実測)。定義上も無制限で、速度そのものが制約になることは稀

**【実機計測】`Rotational Speed` の単位とスケール**(ユーザー実測):
- **ギア比1:1で入力1.0を与えると Angular Speed Sensor が 5 を返す**(= 5 rotations/s = 5 turn/s = 1800°/s)。したがって **入力値 = 指令角速度[turn/s] / 5**
- **ギア比はトルクにほぼ関係せず、立ち上がり速度(応答の速さ)に効く。** 1:1固定で組むのが素直
- **【罠】入力に1を超える値を入れると、部品の速度上限を超えて加速し続けるバグじみた挙動をする。** 使わないこと。**出力は必ず ±1 にクランプする**
- 実用上の指令値は非常に小さい。対空砲の最悪ケース(300m横断の目標250m/s)でも要求0.133 turn/s = **入力0.027**。**入力レンジの3%以下で制御することになる**ため、**静止摩擦やデッドバンドがあれば低速側で効く**。実機では「動き出す最小入力値」を測っておくとよい

**砲身の実姿勢フィードバック(マズルリファレンス)**:
- ピボットの `Current Rotation` は**関節角**であり、たわみ・バックラッシュを含んだ**砲身の実際の向きではない**。砲弾は後者に沿って飛ぶ
- **砲の機関部にPhysics Sensorを置けば砲身の実姿勢(オイラー角)が直接取れる。** M1A2等のマズルリファレンスシステムと同じ発想
- たわみ0.5°は1000m地点で8.7mの照準誤差に相当し、他の誤差要因と同オーダー。**両方配線して差分を診断出力し、実測してから使うかを決めるのが安全**

**速度指令型ピボットの位置制御の要点**(`Obj 1872 AAFCS` で机上検証):
- **指令角を実測と同じtick数だけ遅らせてから誤差を取る**こと。これをやらないと誤差がループ遅延に比例して増える(遅延4tickで23m相当 → 補償すると4.9m相当)
- **指令角の変化率をフィードフォワードで送る**。フィードバックは残差の補正だけを担当させる
- `Current Rotation` は0〜1で連続回転するので、誤差計算は必ず `(a+0.5)%1-0.5` で最短経路に折り返す
- **旋回速度の上限より、砲塔の慣性による一次遅れの方が効く**(時定数0.2秒で1000m地点12.4m相当の誤差)。実機ではステップ応答の時定数を測ること

### 4.3d ビークル物理・空力/流体の全定数(【一次資料で確認済み】)

**`<Stormworks>/sdk/data/game_constants.xml` にバニラの全定数が、説明と既定値つきの平文で入っている。**
`rom/` ではなく `sdk/`(Component Mod SDK)配下なので見落としやすい。**ゲーム物理を模擬するうえで最も価値の高い1ファイル。**

見つけ方: ワークショップMOD「Tweaked Aerodynamics」(`workshop/content/573090/3519724536/data/game_constants.xml`)が
このファイルを上書きする形になっており、**MODとバニラを突き合わせるとパラメータ名がそのまま分かる**。

#### ワールド

| 定数 | 既定値 | 説明(原文の訳) |
|---|---|---|
| `gravity` | **10.0** | ワールド重力 |
| `vehicle_linear_damping` | 0.1 | ビークルの基本線形減衰。**大気の高度が上がるほど0へ近づく** |
| `vehicle_angular_damping` | 0.1 | ビークルの基本角減衰 |
| `vehicle_friction` | 0.8 | ビークル物理表面の基本摩擦 |
| **`radar_noise`** | **0.001** | 報告されるレーダー位置の微小な誤差の正負レンジ |
| **`radar_distance_noise`** | **0.01** | 報告されるレーダー距離の**最大誤差率** |
| **`sonar_noise`** | **0.001** | 報告されるソナー位置の微小な誤差の正負レンジ |

- **`radar_noise = 0.001` は、これまで「コミュニティ実測値」として扱ってきた値の一次資料。** `Obj 1872 AAFCS/sim/rig.lua` が使っている 0.001 turn はこれで裏付けられた
- **`radar_distance_noise = 0.01`(距離の1%)は §4.1「測定ノイズ」表(ユーザー確認)の一次資料。** 遠距離ほど絶対誤差が大きくなる
- **【要注意】`gravity = 10.0` はビークルに効く重力であり、§4.3b の「砲弾の重力 30 m/s²」とは別物**(§4.3b に詳細)

#### 空力

| 定数 | 既定値 | 説明(原文の訳) |
|---|---|---|
| `air_density` | 1.225 | 空気密度 |
| `air_drag_factor` | 0.2 | ビークル本体に適用される空気抗力の基本係数 |
| `air_drag_factor_linear_pressure` | 0.05 | 圧力抗力に対する速度の線形項 |
| `air_drag_factor_quadratic_pressure` | 0.1 | 圧力抗力に対する速度の2乗項 |
| `air_falloff_power_pressure` | 0.6 | 運動方向に対する面の向きによる圧力抗力の減衰指数 |
| `air_drag_factor_linear_suction` | 0.025 | 吸引抗力に対する速度の線形項 |
| `air_drag_factor_quadratic_suction` | 0.05 | 吸引抗力に対する速度の2乗項 |
| `air_falloff_power_suction` | 0.4 | 同、吸引側の減衰指数 |
| `air_speed_threshold` | **15.0** | 面が空力を受け始める速度しきい値 [m/s] |
| `air_high_speed_threshold` | **50.0** | 高速スケーリングが掛かり始める速度しきい値 [m/s] |
| `air_high_speed_factor_scale` | 8.0 | 高速時の加算スケーリング(0.1 が 1.1 倍に相当) |
| `viscous_air_resistance` | 0.25 | 粘性抵抗力の係数 |
| `lift_force` | 0.25 | 面に適用される揚力のスケーリング |

- **抗力は「圧力(pressure)」と「吸引(suction)」の2系統**を、それぞれ線形項+2乗項で計算し、面の向きに対する減衰指数を掛ける構造
- **`air_speed_threshold = 15 m/s` 未満では空力が一切働かない。** 低速域の挙動を模擬するときはこの不連続に注意
- 50 m/s を超えると別のスケーリングが入る(2段階モデル)

#### 流体(水) — 【exe逆アセンブルで確定】

**抗力の式そのものを `stormworks64.exe` から特定した。** 追跡経路(定数パーサ VA 0x140294F70 →
シングルトン VA 0x140295A10 → 水力本体 VA 0x14048E6A0)と生データは
`Obj 9901 SWSIM` の手元資料(非公開)。

```
S = |相対流速|,  cosθ = dot(流向, 面法線),  A = 方位ビンの合計面積
粘性(全濡れ面):  0.5 × 1000 × Cf × A × Feff × S²
                 Cf = 0.00075 × (1.5 + 20/(S·L·1e6 + 10))
圧力(cosθ>0):    (15S + 17S²) × A × cosθ^2.5      ← 密度も water_drag_factor も掛からない
吸引(cosθ<0):    (12S +  5S²) × A × (-cosθ)^0.3
Feff = water_drag_factor + 4.0 × clamp((質量kg − 2000)/23000, 0, 1)   → 1.0〜5.0
```

**【重要】実測感覚と食い違いやすい点**

- **抗力は「水面に触れている断面積」ではなく、没水部の露出面すべてに効く。** ただし圧力項が
  `cosθ^2.5` で急に落ちるため、結果として**没水正面投影面積に近い挙動**になる
- **造波抵抗に相当する項は存在しない。** 水面でだけ発生するのは slamming のみ
- **部分没水と完全没水で抗力の式は切り替わらない。** 違いは (a) slamming は水面をまたぐ
  ノードでのみ発動、(b) 完全没水なら空力計算を呼ばない、(c) 完全水面下では波高サンプリングを省略
- **質量は抗力に効く。ただし粘性項にだけ。** 2000kg以下で倍率1.0、**25000kg以上で5.0**(頭打ち)。
  (2000/23000 は exe 内のリテラルで、XMLの `buoyancy_mass_modifier_lower/upper` とは別物)
- **実用上、粘性項が支配的**。哨戒艇クラス(20m・40t・10kt)で圧力311N / 吸引930N / **粘性9511N**。
  **船首を寝かせても全抗力は8割程度までしか落ちない**(粘性項は船型に無関係のため)
- 力は面ごとではなく、8分木ノード × **26方向(58ビン)に集約された「方向別合計面積+面積重心」**単位で計算される
- 波速度は `wave_velocity_factor`(上昇時0.5 / 下降時さらに×0.3)で相対速度から差し引かれる

**slamming は通常航行時にも効く**(着水時限定ではない)。ノードが水面をまたいでいれば毎tick発動しうる:
`F = surfaceVec · dot(surfaceVec, normal) · clamp(閉じ速度, 0, 5)²/25 · 質量 · (2A/T) · 100.0`
**質量に比例し、閉じ速度は 5 m/s で頭打ち。**

**減衰は切り替えではなく連続ブレンド**(submergence = **没水体積の割合**。深度ではない):
```
角減衰 = 0.05 + 0.92 × (没水体積 / 総体積)      → 完全没水で 0.97
線形減衰 = 0.2 + 0.4 × (同じ比)                 (0.2/0.4 は exe内リテラル)
```

**【未検証】** 58方位ヒストグラムが外板のみか内部面も含むか。`Cf` の長さ量 `L` の正体。8分木の葉サイズ。

#### 流体(水) — 定数一覧

| 定数 | 既定値 | | 定数 | 既定値 |
|---|---|---|---|---|
| `water_density` | 1000.0 | | `drag_factor_linear_pressure` | 15.0 |
| `water_drag_factor` | 1.0 | | `drag_factor_quadratic_pressure` | 17.0 |
| `water_drag_weight_factor_multiplier` | 4.0 | | `falloff_power_pressure` | 2.5 |
| `viscous_resistance_factor` | 0.5 | | `drag_factor_linear_suction` | 12.0 |
| `slamming_force_multiplier` | 100.0 | | `drag_factor_quadratic_suction` | 5.0 |
| `wave_velocity_factor` | 0.5 | | `falloff_power_suction` | 0.3 |
| `wave_velocity_factor_reverse` | 0.3 | | | |

**浮力**: `buoyancy_mass_multiplier` 10.0 / `buoyancy_mass_modifier` 10.0 / `buoyancy_mass_modifier_lower` 2000.0 / `buoyancy_mass_modifier_upper` 23000.0 / `buoyancy_mass_modifier_pow` 1.0 / `buoyancy_damping` 0.05 / `buoyancy_damping_submerged` 0.92

- **水の抗力係数は空気の 200〜300倍**(`drag_factor_linear_pressure` 15.0 対 0.05)。潜水艦・魚雷の挙動はこちらの表を使うこと

---
