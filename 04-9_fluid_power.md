### 4.9 流体系・機関・電力部品 — 定義XMLの読解結果(挙動は未検証)

**【一次資料 2026-09-17】`rom/data/definitions/*.xml` 759ファイルのうち、fluid ノード(type=3)を持つか
`water_component_type≠0` の 133 ファイルを機械抽出して読んだ結果。** 加えて機関・電力系
(`modular_engine_*` / `battery_*` / `generator_*` / `electric_*`)も読んだ。

> **ここに書いてあるのは「何のノードがあり、tooltip が何と言っているか」だけ。**
> §0b のとおり、**定義XMLは挙動を書いていない。** 容量・流量・消費率・更新レートは
> 一つも書かれておらず、**全部実機で測るしかない**(4.9.9 に測るべき項目を列挙)。
> ストワの流体力学は独自実装で、実世界の直感で推論しないこと。

#### 4.9.0 読み方の注意

- **【訂正 2026-09-23】流体はパイプブロックで繋ぐ。** 該当部品は `trans_*.xml`(§4.18.2、名前が `pipe_*` ではないので
  検索で見つからなかった)。**動力(power)も同じパイプを使う。**
  **「流体はロジック接続の線で繋ぐ」は過去の仕様で、本資料の旧記述は誤りだった**(ユーザー指摘)。
  ただし **現行システムにノード制の名残が残っている可能性がある**(部品側に fluid / power ノードは存在し続けている)。
  §5.7 で「Torque(2)と Fluid(3)は `logic_node_links` に一切登場しない」と観察されているのは、
  **パイプがボクセル隣接で繋がるため**と読むと整合する
- 定義XMLの `mode` 属性は `0`=出力 / `1`=入力だが、**新しめの部品では mode 未指定のノードが多い**
  (下表で「?」)。ラベルと tooltip から向きを読むこと。**fluid ノードは in/out の区別が実質無く、
  双方向に流れる**部品が多い(バルブ tooltip: *"Fluid can flow in both directions."*)
- **【ユーザー知見 2026-09-17】流速は基本的に「入口と出口の圧力差」で決まる。** それ以上の法則は分かっていない。
  二次資料([note 爆速流体輸送法](https://note.com/mumenry/n/n50fc3c069e62))も「流体移動は圧力差によってのみ発生」と
  一致。同資料の実測: 大型電動ポンプ最大 60 atm / 小型 10 atm / インペラー 60 atm(逆止なし)、
  輸送元・輸送先とも 30〜40 atm が効率的、**流路を直結で合流させると流量が落ちる**(カスタムタンク経由なら維持)
- ほぼ全部品に `pump_pressure="0.01"` が付いている(ポンプは 10 / 100)。**意味は未検証**
  (配管抵抗か、ポンプの押し出し圧か)。上の実測(10 atm / 60 atm)とは Large が一致しないので、素直な最大圧ではない

#### 4.9.1 タンク

| 部品 | voxel | mass | ノード |
|---|---|---|---|
| Fluid Tank Small | 2 | 1 | `Stored Fluid`(fluid、**2面**)/ `Tank Content`(num 出力、**L**)/ `Tank Pressure`(num、**atm**) |
| Fluid Tank Medium | 12 | 6 | 同上 |
| Fluid Tank Large | 45 | 22 | 同上 |
| Gas Tank Small/Medium/Large/Huge | 3/7/63/225 | 2/5/12/22 | `Stored Gas` / `Tank Content` / `Tank Pressure`。**専用型(`water_component_type=20`)** |

- tooltip: *"The tank will spawn full of the selected fluid type."* — **初期充填する流体種はプロパティで選ぶ**
- **容量は定義XMLに書かれていない。** [JP Wiki FLUID](https://wikiwiki.jp/sbarjp/FLUID) の値:
  **Small 31.25 L / Medium 187.5 L / Large 703.125 L**(= voxel 数 × 15.625 L、2:12:45 に厳密に比例)。
  Gas Tank は Small 46.41 / Medium 108.28 / Large 974.53 / Huge 3480.47 L。**いずれも spawn 時 約60 atm**
  (液体タンクも 60 atm 台で spawn する、と JP Wiki は記載。要実機確認)
- `Tank Content` は L 単位の絶対量。**割合ではない。** 割合表示には容量を実測して割る必要がある
- **【ユーザー実機確認 2026-09-23】1 voxel = 0.25³ m³ = 15.625 L ちょうど。** 上の3サイズも全部 voxel 数 × 15.625 に一致する。
  **容量は部品固有値ではなく「囲った物理体積」**(→ カスタムタンクも同じ。4.9.4b)
- **自作タンク(カスタムタンク)** は 4.9.4b

#### 4.9.2 計測部品

| 部品 | ノード | 備考 |
|---|---|---|
| **Liquid Meter** (`water_measure.xml`) | num1 `Liquid Level`(L)/ num2 `Fluid Capacity`(L、**非密閉なら 0**)/ comp `Composite Data` | 密閉区画内の液体量。**【訂正】4.5 の旧表は順序が逆**(定義順は Level が先) |
| **Gas Meter** | num1 `Gas Level` / num2 `Fluid Capacity` / comp | 同上の気体版 |
| Fluid Pressure Sensor | fluid `Fluid` / num `Pressure` | 接続した流体ネットワークの圧力 |
| Barometer | num `Pressure` | *"pressure in atmospheres of the compartment or current altitude"* |
| Torque Meter | power `RPS` / num `RPS` / num `Torque` | 動力系。4.5 と同じ |
| バルブ・ポンプ・フィルタ各種 | num `Flow Rate` | **L/s。** 4.9.3 参照 |

**Liquid Meter の `Composite Data` — 流体種別ごとの量が channel に載る**(tooltip 原文より):

| ch | 液体(Liquid Meter) | ch | 気体(Gas Meter) |
|---|---|---|---|
| 1 | Water | 4 | Air(= O2 + CO2 + N2 の和) |
| 2 | **Diesel** | 5 | CO2 |
| 3 | **Jet Fuel** | 8 | Steam |
| 6 | Oil | 11 | Oxygen |
| 7 | Seawater | 12 | Nitrogen |
| 9 | Slurry | 13 | Hydrogen |
| 10 | Saturated Slurry | | |

**【ユーザー実機確認 2026-09-23】Liquid Meter は毎 tick 更新、分解能は 0.1 L 以下。**
→ **残量計に平滑化はほぼ要らない。** 量子化による階段状の変化を気にせず、**前 tick との差から消費率を直接出せる**
(60 tick 分を足せば L/s)。§4.9.9 の項目2は解決。

*"The data can be directly merged with a Gas meter"* — 液体・気体で channel が重ならないよう割ってある。
**燃料タンクに海水が混入したかを ch2 vs ch7 で判別できる**(はず。未検証)。

> **Liquid Meter を密閉区画の外に置くと「水面からの高さ」を返す**(原文: *"If not inside an enclosed
> volume, it will give the height relative to the water surface."*)。**【ユーザー実機知見 2026-09-17】確認済み。
> 波があればそれに応じて上下する、絶対高度(Altimeter)とは別の値。** 喫水計に使える。
> 符号・基準点(部品位置か)は未記録。NAVAID の喫水算出(Altimeter ベース)の代替候補。

#### 4.9.3 バルブ・ポンプ・フィルタ

| 部品 | 制御入力 | 電力 | `Flow Rate` | pump_pressure | 備考 |
|---|---|---|---|---|---|
| Fluid On/Off Valve | bool `Valve Control` | **要** | あり | — | 双方向 |
| Fluid Variable Valve | num `Valve Control` | **要** | あり | — | 双方向。開度の非線形性は未検証 |
| Fluid On/Off Valve (Manual) | 手動(interact) | 不要 | あり | — | Lua から動かせない |
| Fluid Flow Valve | なし | 不要 | あり | — | **逆止弁**。**【ユーザー実機知見】流量制限が厳しく、燃料線に挟むとエンジンが性能を出せない。使わない** |
| Liquid / Gas Relief Valve | なし | 不要 | あり | — | 液体のみ / 気体のみ通す |
| Fluid Filter (`fluid_filter.xml`) | なし(プロパティで種別選択) | 不要 | あり | 0.01 | 指定した流体種だけ通す |
| Fluid Filter (`fluid_filter_v2.xml`) | なし(プロパティで液体/気体) | 不要 | **なし** | — | 同名の別部品。ワークベンチでどちらが出るか要確認 |
| **Fluid Pump** | bool `On/Off` | **要** | あり | **10** | |
| **Large Fluid Pump** | bool `On/Off` | **要** | あり | **100** | |
| Fluid Pump (Manual) | 手動 | 不要 | あり | 3 | |
| Impeller Pump / (Small) | power `RPS`(トルク駆動) | 不要 | あり | 9 / 1 | 旧名 turbocharger。エンジン軸で回す |
| Modular Engine Fluid Pump | num `Clutch Pressure` | 不要 | なし | 0.01 | Drive Belt に付ける |

**`Flow Rate`(L/s)を持つ部品が多いのが最大の収穫。** 燃料消費率は「タンク残量の時間微分」ではなく
**燃料配管上のバルブの `Flow Rate` を直読する**のが素直。**ただし逆止弁(Flow Valve)は絞りが強くて不可
(ユーザー実機知見)。On/Off Valve か Variable Valve を全開で挟む**(どちらも電力が要る。
**【JP Wiki】無給電だと制御入力に関わらず全閉**)。**符号(向き)・tick ノイズ・更新レートは未検証**(4.9.9)。
**【JP Wiki】Fluid Pump は Off で流体を通さず、逆方向は On/Off・給電に関わらず通さない**(逆止弁を兼ねる)。

> **【ユーザー実機知見 2026-09-17】燃料の流れは連続ではなく「少し流れて止まる」を繰り返すことがある。**
> 起きる条件は2つ: **(a) 消費量が極小のとき**(どこかにバッファがあって、溜まっては流れる挙動)、
> **(b) エンジンがレブリミットに当たっているとき**(燃料カットの断続)。どちらも `Flow Rate` が
> 0 と正の値を往復するので、**消費率として使うなら時間窓で平均する必要がある。** 逆に高負荷で
> 定常運転しているときは丸めなくても読める。窓長はプロパティにして実機で決める。

#### 4.9.4 ポート(区画/海との接続)

| 部品 | voxel | tooltip |
|---|---|---|
| Fluid Port (`water_inlet.xml` / `water_outlet.xml`) | 2 | *"Place the port inside of an enclosed volume to connect to that volume, or outside of the vehicle to connect to the ocean."* — **inlet と outlet は別ファイルだが同文。各ファイルの fluid ノードは1個**(2個あるのはタンクの `Stored Fluid`)。**【JP Wiki】ポート系(Exhaust / Intake / Port / Slot Port)はエフェクトと音以外に性能差なし。Fluid Port は水密面を持つ最小サイズ** |
| Fluid Port End | 1 | 同上(1 voxel 版) |
| Fluid Intake | 6 | 同上(*"exterior"*) |
| Fluid Slot Port | 24 | 同上(大型、`water_component_type=12`) |
| Fluid Exhaust | 2 | 区画または海へ出す |
| Air Filter / Air Ram | 1 | 空気の出入口 |
| Air Scoop Intake | 1 | *"performs better at higher velocity"* |

#### 4.9.4b カスタムタンク(自作タンク = 密閉区画)

**タンク部品を置かずに、船体内に囲った空間そのものを容器として使う。** 大容量・船体形状に合わせた形が取れるので、
**燃料タンクもバラストタンクも実用上はこれで作る。**

- **繋ぎ方**: その空間の内側に Fluid Port 系(4.9.4)を置くと「その区画」に繋がる。**同じ部品を船外に置けば海に繋がる。**
  置く場所が内か外かだけの違い
- **【ユーザー知見 2026-09-23】区画の境界として認められる面は自由度が高い。**
  ブロック・窓・**ウェッジ等の平らな面**、**ドア**も空間判定になる。**カスタムドアは外枠だけで空間判定**になる
- **【ユーザー確認 2026-09-23】Physics Flooder(§4.18.3)とは競合しない。** ただし人が入れなくなるので点検はできない
- **区画の中にブロックや部品を置いたとき、それが容量をどれだけ食うかは不明。**【ユーザー知見】完全に謎、とのこと。
  配管・ポンプを区画内に通すと容量が減るのかどうかが分からない。**調べる価値は薄い**(誤差の範囲と見てよい)が、
  **容量を実測で決める以上、「後から中に物を足したら容量が変わるかもしれない」ことは頭に置く**
- **【ユーザー知見・重要】「空間として判定されるか」と「水が漏れる/入るか」は別。**
  枠さえあればシールされていなくても**タンクとしては成立**し、**シールされていれば漏れない**。
  → **§4.13.2 の Door Frame Controller の `Seal State` は「漏れないか」の側**。タンク判定の有無とは対応しない
- **容量 = 内部ボクセル数 × 15.625 L。【ユーザー実機確認 2026-09-23】**
  15.625 L は 0.25³ m³ そのもの(1 voxel の物理体積)で、標準タンク3サイズの容量とも一致する。
  **容器の種類によらず「囲った体積ぶん入る」**と考えてよい
- **量の読み方**: `Tank Content` に相当するノードが無いので **Liquid Meter / Gas Meter**(4.9.2)で読む。
  **【ユーザー知見】返るのはリットル(絶対量)。** 割合表示にするには容量を自分で持つ必要がある
  → **航続距離計・燃料残量計は「区画の容量」を定数として持つ設計になる。** 実測して `pn()` で持たせる
- **圧力**: **【ユーザー知見】加圧できる。** ただし**燃料タンクとして使うときは加圧しないと負圧になり、燃料供給が滞る例がある。**
  → **カスタム燃料タンクにはポンプか圧縮空気での加圧を設計に入れる。** 「置けば吸い出せる」ではない。
  **【ユーザー知見 2026-09-23】必要な圧力の条件は未解明。経験則として 5 atm ほど掛けておけばとりあえず問題ない、という程度。**
- **【ユーザー知見 2026-09-23】Enclosed パイプ(`trans_block_*`、§4.18.2)はカスタムタンクの壁にできる。とても便利。**
  → **タンクを配管が貫通しても密閉を保てる。** 区画の中を通す配管の取り回しで困らない
- **【JP Wiki クラフトガイド/潜水艦】圧縮空気の実装以降、カスタムタンク内に気体が混入する挙動になった。**
  内部が真空でないと成立しない構成(古いバラスト排水手法)は壊れている
- **損傷時**: **【ユーザー知見】ブロックが消滅してタンクが壊れる、のではない。ブロックが破損すると液体が通るようになる、
  と考えるほうが近い。** 結果として**中身が漏れることも、外から浸水することも両方起きる**(水圧との関係と推測)

**バラストタンク = 密閉区画 + Fluid Port(区画内)+ ポンプ + Fluid Port(船外=海)。**
**海水は ch7 `Seawater`** なので、Fluid Filter で種別を絞れば燃料系との混合を防げる(はず)。
【JP Wiki 潜水艦】注水が遅いのは上下の圧力差が小さいため。バルブを増やすか、区画内の空気を抜くと速くなる。
トリムタンク間の海水移動は深度の影響を受けない。

#### 4.9.5 冷却・熱

| 部品 | 内容 |
|---|---|
| Fluid Heat Sink / Fluid Heat Radiator | fluid A/B。*"Fluid inside the cooler will lose temperature."* 受動 |
| Fluid Heat Radiator 3x3 / 5x5 (Electric) | + bool `Fan On/Off`、elec、num `Temperature` 出力 |
| Liquid-Liquid Heat Exchanger 2x2 / 5x5 | A in/out・B in/out、num `Temp A`/`Temp B` |
| Air-Liquid / Air-Air Heat Exchanger 各サイズ | 同構成。*"rate proportional to the component's size"* |
| Cryo Cooler | 電力で A を冷やし B へ移す。bool `On/Off` |

エンジン(4.9.6)は `In Coolant`(冷)/ `Out Coolant`(熱)を持ち、これらを通すループを組む。

#### 4.9.6 機関(燃料の消費側)

**一体型ディーゼル**(Small / Medium / Large Engine、`engine.xml` / `aircraft_engine.xml` / `engine_diesel.xml`):

| ノード | 型 | 内容 |
|---|---|---|
| `Throttle` | num 入力 | 0〜1 |
| `Starter` | bool 入力 | 電力でRPSを上げる |
| `RPS` | power 出力 | 動力接続 |
| `Fuel` / `Air` | fluid 入力 | 燃料・空気 |
| `In Coolant` / `Out Coolant` | fluid | 冷却ループ |
| `Exhaust` | fluid 出力 | Large は2口 |
| `RPS` / `Temperature` | num 出力 | 計器用 |
| `Electric` | elec | |

mass: Small 30 / Medium 80 / Large 400。**燃料消費モデル(スロットル比例か RPS 比例か負荷比例か)は書かれていない。**
**RPS 上限・温度 120 ℃ で爆発・アイドル無し、といった運転上の知見は §4.11.3。**

**モジュラーエンジン(24部品)は §4.14 に独立させた。** 構成・空燃比制御・Alternator の位置づけはそちら。

**ジェット系**は §4.6。Combustion Chamber は `Fuel`(fluid、Jet Fuel)+ num `Throttle`。

**その他の燃料消費者**: Industrial Diesel Furnace(`Diesel In` fluid、num `Diesel Level` 出力。**蒸気プラントは §4.16.1**)、
Hydrogen Fuel Cell(H2/O2 → elec + Water Out)。

#### 4.9.7 電力(参考。燃料と並んで「残量管理」の対象)

| 部品 | 容量(`electric_charge_capacity`) | mass | ノード |
|---|---|---|---|
| Battery Small | **1600** | 10 | elec `Electric Store` / num `Charge`(**0〜1**) |
| Battery Medium | **12800** | 60 | 同上 |
| Battery Large | **256000** | 800 | 同上 |
| Generator Small / Medium / Large | (1 / 10 / 100) | 5 / 100 / 400 | power `RPS` 入力 / elec / num `Output` |
| Electric Relay | | | elec A/B + **bool `Relay State`**(Lua から切れる) |
| Electric Circuit Breaker | | | elec A/B(bool 入力なし。手動?) |
| Electric Charger (`electric_diode.xml`) | | | 一方向、*"when there is a significant discrepancy in charge"* |

**【JP Wiki ELECTRIC】容量: Small 400 / Medium 3200 / Large 64000 SWatt**(XML の `electric_charge_capacity` のちょうど 1/4。XML から換算した値の可能性があり、実測ではない)。
**【JP Wiki】発電機出力 = k × RPS²、k = Small 0.00642 / Medium 0.241 / Large 1.138。** 回転数の2乗なので低回転では極端に出ない。
Circuit Breaker は雷で自動遮断、Relay は雷でも切れない(JP Wiki)。
**Battery の `Charge` は 0〜1 の割合。タンクの `Tank Content` は L の絶対量。** 同じ「残量」でも単位が違う。
`electric_magnitude`: Generator 0.6 / 0.75 / 0.85、Alternator 0.05。意味は未検証(発電効率か)。

#### 4.9.8 推進・その他

- **Azimuth Thruster**: power `RPS` 入力のみ。*"attached to a robotic pivot to provide steering"*、水中のみ動作。
  **サイドスラスターは Azimuth Thruster か通常プロペラを横向きに置く**ことになる(専用部品は無い)
- **Fluid Jet**(ウォータージェット): fluid `Fluid Flow In` + power `RPS` + num `Vertical Trim` / `Deflector A` / `Deflector B` + elec
- RCS Thruster: comp 入力(18 bool ch)+ fluid `Fluid Supply`(圧縮気体)。宇宙用
- Robotic Pivot (Fluid) / Velocity Pivot (Fluid) / Robotic Hinge / Linear Track: **fluid を通せる可動部**。
  回転部や伸縮部をまたいで配管したいときはこれ
- Fluid Connector / Large Connector / Hardpoint Connector / Winch 各種 / Fluid Hose Anchor: **艇間で流体を渡せる**
  (給油・給水)。Fluid Connector は bool `Release Connector` + bool `Connected` 出力
- Desalinator(海水→真水、受動)、Electrolyser(水→H2/O2、電力)、Separator(遠心分離、power 駆動)、
  Distillation Port、Slurry Filter、Steam 系(Boiler は num `Fluid Volume`/`Temperature` 出力)

#### 4.9.9 実機で測るべきこと(本節の未検証事項)

流体を使う最初のプロジェクトで、以下を先に潰す:

1. ~~タンク容量~~ → **決着。1 voxel = 15.625 L(ユーザー実機確認 2026-09-23)。カスタムタンクも同じ**(4.9.1 / 4.9.4b)
2. ~~分解能と更新~~ → **決着。Liquid Meter は毎 tick 更新・分解能 0.1 L 以下(ユーザー実機確認 2026-09-23)。** 平滑化不要、差分から消費率を直接出せる(4.9.2)
3. **`Flow Rate` の符号・更新レート**: 全開の On/Off Valve を燃料線に挟んで読めるか。**全開バルブ自体が
   流量を絞らないか**も見る。断続する条件は判明済み(上記)なので、**断続の周期**を測って平滑化窓の既定値を決める
4. **エンジンの燃料消費モデル**: ~~どれに比例するか~~ → **【ユーザー知見 2026-09-23】回転数だけが高くても、スロットルだけが高くても消費しない。両方を引数にした関数と思われる**(§4.11.3)。**関数形そのものは未特定**。モジュラーエンジンは別(§4.14)
5. **ポンプの実流量**(L/s)。二次資料の「小 10 atm / 大 60 atm」の裏取り。~~バルブに電力が要るか~~ → JP Wiki: 要る
6. **Liquid Meter の「水面からの高さ」**: ~~使えるか~~ → 使える(ユーザー確認)。**符号と基準点**だけ未記録
7. **Liquid Meter の comp ch2/ch7** で燃料と海水の混入が判別できるか
8. ~~inlet/outlet の差~~ → JP Wiki: ポート系は性能差なし
9. 同名 2 種の Fluid Filter のどちらがワークベンチに出るか

> **Door Frame Controller の `Seal State`(§4.13.2)は「そのドア枠が塞がっているか」であって、流体側の密閉区画判定とは別物。**
>
> **4.9.4 の「密閉区画」の判定規則(どの面が密閉と見なされるか)も定義XMLからは分からない。**
>
> **【ユーザー実機知見 2026-09-18】区画の圧力・容量は破損検知に使えない。**
> - 外板に穴が開いても **`Fluid Capacity` は 0 に落ちない**
> - 圧力の漏れは **ドアの開閉でも起きる**ので、漏れ続け = 穴、とは言えない
> - 浸水量は隣接区画からの流入と区別できず、水線上の穴は見えない
>
> 破損検知は Temperature Sensor の「破壊→0」(§4.5)か、指令と応答の突き合わせで行う。

---
