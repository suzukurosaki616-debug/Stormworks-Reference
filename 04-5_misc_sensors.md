### 4.4 Sonar

水中限定。プロペラ・エンジン等の音を出す部品をパッシブ検知できるほか、大きな音(ping)を出してアクティブ探知も可能。**pingを送信している間はパッシブ信号が抑制**され、新しいpingを送るたびに古い信号は上書きされ以後データが返らなくなる。

**【一次資料で確認済み】`rom/data/definitions/sonar_advanced*.xml` を直読。レンジの60/100/200kmは裏取り完了。**

| 定義ファイル | name | radar_range | radar_speed | mass |
|---|---|---|---|---|
| `sonar_advanced.xml` | Sonar (Small) | 60000 | 0.04 | 10 |
| `sonar_advanced_5.xml` | Sonar (Medium) | 100000 | 0.04 | 50 |
| `sonar_advanced_7.xml` | Sonar (Large) | 200000 | 0.04 | 200 |

**【重要】ソナーは距離を出さない。レーダーの `Radar Data` とはレイアウトが別物なので、模型や解析コードを流用すると必ず間違う。**

| | レーダー (`Radar Data`) | **ソナー (`Sonar Data`)** |
|---|---|---|
| ターゲット数 | 8 | **16** |
| bool ch | 1〜8 (Target n Found) | **1〜16** |
| number ch | 4/target: 距離 / 方位 / 仰角 / 検出後経過 | **2/target: Pivot(方位) / Pitch(仰角) のみ** |
| 距離 | 出る | **出ない(bearing-only)** |

定義XMLの原文: *"Outputs data for up to 16 targets. On/Off 1-16 : Target 1 Found, ... Values 1-32: Target 1 Pivot, Target 1 Pitch, Target 2 Pivot, ..."*

- ノード構成(3機種共通): `Electric` / `Activate`(bool入力) / **`Ping`(bool入力)** / `Sonar Data`(composite出力)。**Sonar (Small) のみ** `Torpedo Output`(composite出力、最も近い目標の x/y を rocket fins へ直結する用)を追加で持つ
- passive/active は別部品ではなく、**同一部品の `Ping` 入力で切り替える**
- **ソナーにFOV入力ノードは存在しない**(旧型と違い、画角を指定できない)
- 【未検証】音速の値(pingの反射が戻るまでの遅延)は**ROMからは読めなかった**。実機計測が必要

**旧型(アナログ、`radar_sonar*.xml`)は距離を出す。** ノード構成は旧型レーダーと同形(`FOV`/`Facing Yaw` 入力、`Target Distance`/`Signal Strength`/`Elevation Angle`/`Target Found` 出力)。Sonar (Large) レンジ3000 / Sonar (Small) レンジ1000、いずれも Max FOV 0.125。

`sonar_jammer.xml` = **Sonar Noisemaker**(兵装カテゴリ、`radar_range=5000`)。ノードは `Enable` と `Electric` のみ。パッシブ聴取中のソナーに対するデコイ。

### 4.5 その他センサー(概要)

| 部品 | 出力 |
|---|---|
| Player Sensor | Number1=検出人数、Bool1=1人以上検出。Sphere/Hemisphere形状、Detect All/Players/NPCsモード、半径0.25〜10m |
| Wind Sensor | **【一次資料で訂正】現行版の定義順は Number1=`Wind Speed`、Number2=`Wind Direction`**。従来この資料には逆(方向が先)と書かれていたが `wind_sensor.xml` のノード定義順は速度が先。原文は *"Wind Speed: The relative wind velocity."* / *"Wind Direction: The direction of the wind **relative to the component**."* — **どちらも部品基準の「見かけの風」**であり、絶対風を得るには自機速度で補正する必要がある。**【一次資料 2026-09-12】tooltip 原文: *"only measures wind in the plane of the sensor"* — センサー面内の成分しか測らない。傾けて付けると風が減る。方向は -0.5〜0.5 turn** |
| Rain Sensor | Number1=降雨強度(0=快晴〜1=豪雨) |
| Humidity Sensor | Number1=湿度(0=無霧〜1=最大霧) |
| Temperature Sensor | Number1=周囲温度[°C] |
| Torque Meter | Number1=RPS(回転/秒)、Number2=トルク |
| Fluid Pressure Sensor | Number1=圧力 |
| Fluid Meter | Number1=部屋の容量、Number2=部屋内の流体量[L] |
| Clock | Number1=現在時刻(0=午前0時、0.5=正午) |

### 4.5b モニタ (Monitor) — タッチ出力

**【一次資料で部分確認 + JP Wiki】** `rom/data/definitions/monitor_*.xml` のノード構成は全機種共通:

| ノード | mode | type | 内容 |
|---|---|---|---|
| `Video Signal` | 入力 | 6 | 映像入力(MCの `onDraw` 出力を接続) |
| `Touch Output` | **出力** | 5 | タッチ情報の composite(下記) |
| `Electric` | 入力 | 4 | 電力 |
| `Power Switch` | 入力 | 0 (bool) | *"Controls whether or not the screen is switched on."* |

**`Touch Output` の description は定義XMLでは空**であり、内訳はdefからは分からない。チャンネル割り当ては以下(JP Wiki準拠、**実機での最終確認を推奨**):

| number ch | 内容 |
|---|---|
| 1 | モニタ解像度 X [px] |
| 2 | モニタ解像度 Y [px] |
| 3 | 1点目タッチ X [px] |
| 4 | 1点目タッチ Y [px] |
| 5 | 2点目タッチ X [px] |
| 6 | 2点目タッチ Y [px] |

| bool ch | 内容 |
|---|---|
| 1 | 1点目が押されているか |
| 2 | 2点目が押されているか |

**設計上の要点**:

- **解像度が ch1/ch2 で降ってくるのが重要。** 罠8番により `onTick` 内で `screen.getWidth()`/`getHeight()` を呼ぶとエラーになるため、`onTick` 側でタッチ座標をワールド座標へ変換するには**この composite から解像度を得るしかない**。これがモニタ側から解像度を送ってくる理由
- 2点同時タッチが取れるので、ピンチ操作(ズーム変更)を組める
- **【ユーザー実機確認 2026-09-12】押下boolはレベル(押されている間ずっとtrue)。** 「タップ」として扱うには**立ち上がりエッジを自前で取る**(前tickの値を保持して比較)
- 座標系はモニタ座標系(左上原点、y下向き)。`map.screenToMap` にそのまま渡せる
- **モニタの画素数は偶数(32・64・96…)なので、中心の画素が無い。** 64px なら中心は 31.5(画素 31 と 32 の間)。
  「中心 32」で左右対称に置くつもりの図形は、右・下に半画素ずれる。**対称に置くなら端からの距離で考える**
  (例: 縁に接する 2×2 の点の左上は 0〜62。`32 ± 31` だと左と上だけ1px 空く。`Obj 1882` で実機に出た、2026-09-28)
- **半透明(アルファ < 255)で描くなら、同じ画素を2回塗らない。** 重なった画素だけ濃さが変わる。L字を横棒と縦棒の2つの `drawRectF` で描くと角の1画素が別の色に見える。
  **角を片方から外して描く**(`Obj 1882` の HUD、2026-09-29。机上の `sim/hud_render.lua` が重ね塗りの画素を数える)。
  【実機観察・原因未確定】半透明の `drawText`(黒地の上の `NO POWER`)が2色のムラに見えた(ユーザー 2026-09-29)。
  **ムラの境目は文字の画素の区切りと合わず、目で見た形とスクリーンショットの形も違う** → 重ね塗りではなく、ゲームが半透明色を描くときのディザや時間方向の処理らしい。
  **大きく塗る面は不透明で描く**のが安全(黒地の文字は不透明にして回避)

#### Viewing Scope と HMD の描画範囲(【ユーザー実機確認 2026-09-26】)

- **Viewing Scope(`viewing_scope.xml`)の画角は、クライアントの FOV 設定に左右されず一定。** 映像の上に重ねた描画の位置合わせが、プレイヤーを問わず成り立つ
  (以前「クライアントの FOV が反映されるらしい」と `Obj 1882` に書いていたのは誤り)
- **ただし Viewing Scope を覗いている間は、タッチパネルを同時に押せない**
- **【ユーザー確認 2026-09-29】Viewing Scope の解像度は横 288 × 縦 160 px。** 正方形でないので、§4.5e の画角の式の f が横・縦どちらの全角かは未検証(§4.5e の【未検証】行)
- **【ユーザー確認 2026-09-29】映像出力は複数の表示器へ分配できる。** ただし `onDraw` は表示器の数だけ呼ばれる(§1)。
  異なる画面サイズを1つの `onDraw` で描き分けられるようにするための仕様らしい。分配先が増えるほど描画の負荷も増える
- **【ユーザー実機確認 2026-09-28】覗いている間も座席の軸とホットキーは使える。視線(`Look X`/`Look Y`)は使えない。**
  スコープ越しの操作は軸とホットキーで組む(マウスで照準を動かす設計は成り立たない)。座席側の詳細は下
- **HMD はクライアントの FOV 設定によって描画範囲が変わる。** プレイヤーごとに見える範囲が違うので、位置合わせの難易度が高い

#### 座席(`seat.xml` ほか)の軸とホットキー(【ユーザー実機確認 2026-09-28】)

- **軸は4本、各 −1〜1。名前は無い**(Axis 1〜4)。既定キーは 軸1 = A/D、軸2 = W/S、軸3 = ←/→、軸4 = ↑/↓。**プレイヤーごとにキー設定で変わる**
  (ユーザーの環境では軸3 = Q/E、軸4 = LShift/LCtrl)。Workshop に出すものは特定のキー配置を前提にしない
- `Seat data` composite(ROM): Value 1〜4 = 軸1〜4、Value 9/10 = 視線 X/Y、On/Off 1〜6 = ホットキー、On/Off 31 = トリガー、On/Off 32 = 着席中
- **軸の動き**: 押している間 ±1 へ寄るが、**一定の速さではない**。±1 に近づくほど1tick に寄る量が減る(関数は不明)。
  **感度 100% なら1tick で ±1 に張り付く**。離したときに戻る速さも不明の関数
- **離したときの挙動は軸ごとに選べる: Reset(0 へ戻る)/ Sticky(その値で止まる)**。Sticky の軸を一発で 0 に戻す手段は無い
- **よくある使い方: Reset・感度 100% にして、しきい値でボタンとして読む**(ユーザーの常用)。キー操作では軸は実質 −1 / 0 / +1 の3値になる
- 左右のキーの同時押しは、何も押していないのと同じ扱いになる(推測。Reset なら 0 へ寄る)
- ゲームパッドなら中間の値を自在に入れられる(推測)。**キーボードでは中間の値を狙って保てない**
- ホットキーは6個、既定キーは数字の 1〜6。**ホットキーごとに Push(押している間 on)/ Toggle(押すたびに on/off が入れ替わる)を選べる**(座席部品の設定、ユーザー 2026-09-29)。
  トグルの状態を Lua に持たずに済む。**座席の Toggle 状態はマルチプレイで同期される**(普通のロジックノードは同期され、されないのは Lua だけ。§1、ユーザー 2026-09-29)。
  マルチプレイで後から参加した人の状態が食い違う罠 #9 を避けられる
- **Reset/Sticky と感度は座席部品の設定**(ビークルに保存され、作った人が決める。プレイヤーごとではない)。
  **Lua 側はこの設定を前提にしてよい** — 前提にした値は機体の組み立て文書に書く
- **トリガーの既定キーは Space。Viewing Scope を覗いている間も使える**

### 4.5c レーザー系部品(【一次資料で確認済み】)

`rom/data/definitions/` の `laser_beacon.xml` / `laser_point_sensor.xml` /
`laser_distance_sensor.xml` / `radar_advanced_missile_laser.xml` / `camera_gimbal_laser.xml`。

#### 波長は「プロパティ」ではなく「数値入力ノード」

**照射側・検出側の双方に `Wavelength` という number 入力ノードがある**(`mode="1" type="1"`)。
**部品プロパティではないので、Lua から毎tick変えられる。**

| 部品 | ノード |
|---|---|
| Laser Beacon | `Wavelength`(emit) — *"The specific light wavelength to **emit** (whole number)."* |
| Camera Stabilized | `Laser Wavelength`(emit) |
| Laser Point Sensor | `Wavelength`(detect) — *"...to **detect** (whole number)."* |
| Laser Sensor (Missile) | `Wavelength`(detect) |

**整数であること。** 一致しなければ検出されない(=存在しないものとして扱われる)。
**したがって周波数アジリティが可能で、複数の照射器で別々の目標を同時に狙える。**
建造時固定にする必要はない。

#### Laser Sensor (Missile) — **距離が出る**

`radar_advanced_missile_laser.xml`。`radar_range="5000"` `radar_speed="0.03"` `radar_type="2"`。

```
Sensor Data (composite out):
  Bool 1-8   : Target 1-8 Found
  Number 1-32: 各目標につき Distance / Azimuth Angle / Elevation Angle / Time Since Detection
Missile Output (composite out): Value1=Yaw, Value2=Pitch（Rocket Fins へ直結する用）
Activate (bool in) / Wavelength (num in)
```

**レーダーと同一のレイアウト**(距離・方位・仰角・検出経過時間 × 8目標)。
**方向だけでなく距離が直接出る**ので、**距離を別のセンサーから調達する必要がない。**

**【実機確認済み 2026-09-09】** 以下はすべて実測。

- **視野は前方120度、つまり ±60度の「正方形」。** 円錐ではなく**方位・仰角それぞれ独立に
  ±60度**なので、斜め方向では実質85度まで見える。定義XMLには数値が無く、公式説明は
  "acts along the **Z axis**" のみ
- **方位・仰角の単位は turn。右向きが正、上向きが正**(レーダーと同じ規約)
- **【重要】毎tick更新・ノイズ無し。** `radar_type="2"` なのでレーダーと同じ量子化を
  疑っていたが、**そうではない。レーダーの `Di = max(round(Er/2000),1)` も 1% の距離ノイズも
  乗らない。** つまり**レーザーが載る局面では、レーダーより桁違いに素性の良い測定が得られる**

#### Laser Point Sensor — **出力は角度ではなく方向余弦**

```
Bool 1   : is laser detected
Number 1 : X position within the 120 degree FOV   (-1 .. 1)
Number 2 : Y position within the 120 degree FOV   (-1 .. 1)
```

**原文: *"The -1 to 1 values map to an fov of -cos(0.5) to cos(0.5)"*。**
**`値 × 60度` と線形に読んではいけない。** 中心付近では近似が効くが端で外れる。
**正確な変換は要実測。** また *"The **closest point to the center** of the sensor's
field of view will be reported"* とあり、**報告されるのは1点だけ**(最も視野中心に近いもの)。

#### Laser Distance Sensor(LiDAR)

`laser_distance_sensor.xml`。最大 4000 m。*"Can be pivoted up to **0.125 turns** using
composite input."* `Pivot` は composite 入力で `Value1=Pivot X` / `Value2=Pivot Y`。

**【実機計測 2026-09-25、`Obj 9105 LASERLAG`】指令から読みまでの遅延:**
- **Pivot を変えてから距離の読みが変わるまで 4 tick**(6回とも 4)。経路は「Lua の composite 出力 → Pivot 直結」「Distance → Composite Write 2段 → Lua」
- **Active を入れてから有効な距離が返るまで 5 tick**(2回とも 5)。同じ経路
- **【実機 2026-09-28、`Obj 1882 Trinocular-M`】Active を1tick だけ入れても距離は返る**(測距が成立した。ユーザーの試験、回数・返る tick 数は未記録)
- **Pivot の段差は1回で全部反映される**(読みが中間値を通らない)。首振りに速度制限は見られない
- 読みの経路(Composite Write 2段)も遅延に含まれる。経路が違えば値も変わるので、使う側と同じ経路で測ること

**【ユーザー確認 2026-09-24】測距にノイズは無い。** 同じ点を撃ち続ければ同じ値が返る(レーダーの距離1%ノイズとは対照的)。

**【ユーザー確認 2026-09-12】水面を透過して海底の地形を返す。** 水面では反射せず、地面では返る
(`Obj 1171 MRM` の地形追従でも同じ趣旨が確認されている)。**したがって測深に使える** —
真下へ撃てば海底までの距離がそのまま出る。`Obj 1841 NAVAID` の測深・前方測深(FLS)は
この性質の上に立っている。

> **前方測深の制約は射程ではなく幾何。** 俯角 φ で水深 D の海底に当たるとき、
> 見通せる前方距離は `D / tan(φ)`。水深10m・俯角15°で37m、水深20m・俯角15°で75mしかない。
> **浅いところほど前が見えない**ので、4000mの射程はまるごと余る。

**【重要・実機確認 2026-09-09】入力の範囲は −1〜1 であって turn ではない。**
可動域 ±0.125 turn を **−1〜1 に正規化した値**を入れる。

```
入れる値 = 振りたい角度[turn] / 0.125  =  振りたい角度[度] / 45
                                            -- turn をそのまま入れると 1/8 しか振れない
可動域: -1 -> -45度 / 0 -> 正面 / +1 -> +45度
```

- **`Value1`(ch1)は、部品に描かれた2つの矢印のうち小さいほう**の軸
- **矢印の向きが正**

**どちらが方位でどちらが仰角かは取り付けで決まる。** 小さい矢印を振りたい向きへ向けて置く。
なお `distance_sensor.xml`(Distance Sensor)は 500 m で**ピボット不可**。

**【ユーザー実機確認 2026-09-12】無反射時の出力は 4000 ちょうど**(最大レンジ)。0 でも前値保持でもない。

**【一次資料 2026-09-12】Laser Distance Sensor には bool 入力 `Active` がある。** Distance Sensor には無い。
**on を配線しないと永久に無反射**になる。配線漏れが「センサーが壊れている」ように見える罠。

#### Camera Stabilized(照射器側)

`camera_gimbal_laser.xml`。`Enable Laser`(bool in)、**`Laser Distance`(num out、実測距離)**、
`Laser Wavelength`(num in)、そして

```
Composite Output: 1-3 = laser target の x/y/z position, 4 = pitch(turns), 5 = yaw(turns)
```

**照射器の側で目標のワールド座標が直接出る。** 上位から弾体へ座標を送るなら、
**照射器の composite をそのまま使えばよい**(別途の測距や座標計算が要らない)。

### 4.5d Hardpoint Connector(【ユーザー確認 2026-09-09】)

ミサイルやカートリッジの搭載に使う。**接続の検知と、コンポジットの受け渡しを兼ねる。**

- **コンポジットは双方向。** 親→子だけでなく**子→親も流れる**。搭載物が自分の種別や
  残弾を親へ返せるので、**親が搭載内容を知るのに別系統の配線が要らない**
- **【重要】コンポジットは何も切り離さない。** 分離するのは
  **親(`Hardpoint Connector Body`)の `Launch` / `Release` 入力だけ。**
  **これを駆動し忘れると、点火した弾がボルト留めのまま燃える**(`Obj 1171 MRM` の初弾が
  まさにこれ。2026-09-10)
- **`Launch` と `Release` の違いは、子側の `Launched` 出力が on になるかどうかだけ**
  (【ユーザー実機確認 2026-09-10】)。原文の "activate the ordinance" は
  **その出力を立てること以上の意味を持たない。** **同じ解放をして信号が1本増えるので
  `Launch` を使うほうが得。**
- **子側の `Launched` は「本当に分離した」ことを示す唯一の信号。** 発射指令は
  「発射しろと言われた」までしか示さない。**解放に失敗しても弾は発射したつもりで動く**
- **分離するとコンポジットが死ぬ。** **発射コンポジットのラッチが済んでから解放すること**
- 接続の有無が取れるので、**「無接続 = 発射済み」として残弾を数え上げられる**
- **分離すると子側のコンポジットは死ぬ。** 発射前に注入した値は、**子側でラッチしないと消える**
- **【一次資料 2026-09-29】composite も映像も「送り」「受け」が1本ずつ**(`connector_hardpoint_a/b.xml` の `Composite Data Send/Receive`・`Video Data Send/Receive`)。
  上り下りとも1回線ずつなので、搭載物に複数の表示器・操作器を持たせるなら片側へ寄せる(`Obj 1882 Trinocular-M` はポッド側と機体側の2マイコンに分けた)

### 4.5e カメラ

**部品の比較**(【一次資料】`rom/data/definitions/camera*.xml` 直読 2026-09-29):

| 部品(定義ファイル) | 質量 | 入力 | 画角 |
|---|---|---|---|
| Camera Small(`camera_small`) | 5 | `Pivot` のみ(暗視・画角の入力なし) | 固定 |
| Camera Medium(`camera_med`) | 10 | `Pivot` / `Field of View` / `Infrared Mode` | `camera_fov_min` 2.2 〜 `camera_fov_max` 0.025 rad |
| Camera Gimbal(`camera_gimbal`) | 50 | `Pivot Rotation` / `Pitch Rotation` / `Field of View` / `Infrared Mode` | 同上 |
| Camera Stabilized(`camera_gimbal_laser`) | 60 | 上に加え `Enable Laser` / `Stabilizer Mode` / `Tracker Mode` / `Laser Wavelength`。出力 `Laser Distance` / `Composite Output` | 同上 |

- Camera Small / Medium の `Pivot` は説明文どおり ±0.125 turn(値の決まりは下の「Pivot」)
- **大きさ**(ROM の `<voxel>` 数、2026-09-29): Camera Small = **1 ボクセル**、Camera Medium = 2 ボクセル。**正面から見た面積は同じ**(ユーザー。Medium は奥に長い)。Camera Small の定義には `camera_fov_*` が無く、固定の画角の値は不明(実測が要る)
- **Camera Stabilized を `Obj 1882` で採らなかった理由**(ユーザー実機知見): 大きい(質量60)、照射点のトラッカーは地点へ向け続けるだけで**移動目標は追えない**、
  挙動が不安定なことがままある。ただし地面をロックする機能をゼロから作るよりは圧倒的に楽。照射器としての出力は上の §4.5c「Camera Stabilized(照射器側)」

**【実機確認 2026-09-25〜26、`Obj 1882 Trinocular-M`】`Field of View` 入力 → 画角(Camera Medium)**:

```
f = 2.2 − 2.175 × 入力            f = 画面の端から端までの全角 [rad]。入力 0 → 2.2 rad(126°)、1 → 0.025 rad(1.4°)
                                  入力は画角に「角度で」比例する(tan ではない)
倍率 m(最も広い視野を 1 倍、tan の比):  f = 2·atan(tan(1.1) / m)     入力 = (2.2 − f) / 2.175
```

- **確かめ方**: この式で映像に重ねた目標マークが、4 / 8 / 16 倍、その後 16 / 32 / 64 倍の全段で実物に重なった(ユーザー実機)
- **性質**: 入力を等速で動かすと、倍率は最後にギュッと上がる(倍率 ∝ 1/tan(f/2))。「16倍が16倍っぽくない」のは1倍の基準を最も広い 126° に取っているから。
  基準は用途で決めてよい(`Obj 1882` はプロパティ `Zoom 1x FOV`)
- 【未検証】正方形でないモニター(1×2 など)で、f が横・縦どちらの画角になるか。上は 2×2(64×64)でだけ確かめた。
  **Viewing Scope(288×160、§4.5b)に重ねるなら最初に確かめること**(`Obj 1871 GFCS`)

**映像に重ねる描画の投影(64×64)**: 目標をカメラ座標(Physics Sensor の姿勢で機体座標へ → `Pivot` のヨー、ピッチの順に回した後。X 右・Y 上・Z 前)に直して

```
sx = 32 + (x / z) · k     sy = 32 − (y / z) · k     k = 32 / tan(f / 2)   [px]
```

- **`Field of View` への入力と k は、同じ式(上の f)から両方出す。** 別々の推測値を持つと、倍率を変えたときにマークがずれる(`Obj 1882` の初期に縦横ともずれた)
- 64px の画面の中心は画素の境目(31.5)。図形を左右対称に置くときの注意は §4.5b「モニタの画素数は偶数」
- **重ねる描画も、指向と同じだけ自機姿勢を先読みする。** 映像は「今」の世界を映すが、Lua に届く Physics Sensor の姿勢はロジック部品の数だけ古い。
  指向で先読みした姿勢と目標、カメラ角は今効いている(1tick 前に出した)指令で投影する。**先読みしないと機体の回転中にマークが回転の向きへずれる**
  (`Obj 1882` の実機で機首下げ中にマークが下へずれた。机上の遅れ模型で 回転中 19px → 2px)

**Pivot**: 値は 0.125 turn(45°)を 1 とする正規化値で、±1 でクランプ(Laser Distance Sensor と同じ。§4.5c)。
**可動域はヨー・ピッチが独立した正方形**(ユーザー実機知見)。指令から効くまでの遅れは、同じ経路の Laser Distance Sensor で Pivot 4 tick(§4.5c)。
- 【未検証】`Pivot` の写像が「ヨーを振ってからピッチを振るジンバル」か「画面上の tan に比例する別の写像」か。上の投影は前者の仮定で全倍率一致した

**【ユーザー確認 2026-09-29】カメラは窓部品越しに映す。赤外モードでも映る**(ゲームの赤外は熱を見る FLIR ではなく、ただの暗視=NV なので)。
**窓はダメージを受けても透明のまま。** ただし爆発の範囲ダメージは窓も装甲も無視する球なので(§4.3b)、窓の内側に置いても榴弾からは守れない。
利点は「車内から修理に手が届く」ことのほう
**【ユーザー談 2026-09-29】1ブロック(25 cm 四方)より小さい穴は開けられない。** 装甲の奥のカメラに覗き穴を通すと、装甲に 25 cm 四方の穴(窓)ができる

**画角と表示器**: Viewing Scope は画角がクライアント設定によらず一定、HMD はクライアントの FOV 設定で描画範囲が変わる(§4.5b)。

