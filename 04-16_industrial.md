### 4.16 産業部品(蒸気機関・炉・固形資源搬送・原子炉・採掘)

**【一次資料 2026-09-23】`rom/data/definitions/*.xml` のうち category=14(産業)33 ファイルを読んだ結果。**
**比較的新しい部品群で、ノード構成は素直**(入力と出力が対になっていて、状態が num で出る)。
船で使う可能性があるのは **蒸気プラント**と**炉**だけなので、そこを本文にし、残りは表。

#### 4.16.1 蒸気プラントの一周(船の主機/発電に使える唯一の非ディーゼル系)

**熱源 → Boiler(水→蒸気)→ 仕事(Turbine か Piston)→ Condenser(蒸気→水)→ Boiler へ戻す**、という閉ループ。
熱は**すべて「冷却液(Coolant)を温める」形で渡される**。熱源と Boiler は直結ではなく、**Coolant ループで繋ぐ**。

| 部品 | ファイル | mass | ノード |
|---|---|---|---|
| **Firebox / Firebox Large** | `steam_coal_firebox` / `_l` | 100 / 400 | bool `Ignition` / **num `Coal Level`・`Temperature`(out)** / fluid `Coolant In`・`Coolant Out`・`Air`・`Exhaust`。*"Consumes 1 coal to ignite the fire."* |
| **Industrial Diesel Furnace** | `furnace_industrial` | 350 | bool `Ignition` / **num `Diesel Level`・`Temperature`(out)** / fluid `Coolant In`・`Out`・`Air In`・`Exhaust Out`・`Diesel In` |
| **Electric Furnace** | `furnace_electric` | 220 | bool `Enable` / **num `Temperature`(out)** / fluid `Coolant In`・`Out` / elec。石炭も燃料も要らない |
| **Steam Boiler** | `steam_boiler` | 500 | fluid `Coolant A`・`B`(熱をもらう)/ `Water In` / **`Steam Out`** / **num `Fluid Volume`・`Temperature`(out)** |
| **Steam Turbine** | `steam_turbine` | 500 | fluid `Steam In`・`Steam Out` / **power `RPS` ×2** |
| Steam Piston (Small / Medium / Large) | `steam_piston*` | 12 / — / — | fluid `Steam In`×2・`Steam Out`×2 / power `RPS` ×2 / **num `RPS`・`Piston Position`(out)** |
| **Steam Condenser** | `steam_condenser` | 250 | fluid `Coolant A`・`B`(冷やす)/ `Steam In` / **`Water Out`** / num `Fluid Volume`・`Temperature`(out) |

- 原文(Boiler): *"Control the rate of evaporation to maintain consistent output steam pressure."*
  → **蒸気圧を一定に保つのは制御側の仕事。** 熱源の出力(石炭の焚き方 / ディーゼル / 電力)で蒸発量を加減する
- 原文(Condenser): *"should be continuously cooled to function efficiently."* → **復水器にも冷却ループが要る。**
  熱源側と復水側で**2系統の Coolant ループ**を持つことになる
- 【JP Wiki】**100 ℃ 超で真水が蒸気に変わる**(§4.9 の流体種別 ch8 = Steam)
- **Steam Piston** は原文 *"Alternating pressure to the couplings based on the piston position will allow the crankpin to
  [rotate]"* — **`Piston Position` を読んで2つの `Steam In` を交互に加圧する制御を自分で書く**。
  **Turbine は蒸気を通すだけで回る。** 制御の手間は段違い。
  ただし **【ユーザー知見 2026-09-23】ピストンはトルクが割と太い。サイズ・効率・利便性はどれも悪いが、
  パワーは出せるという変な位置にいる。** 手間を許容できるなら選ぶ理由はある
- 熱源は3択で、**Electric Furnace は電力 → 熱 → 蒸気 → 動力**という往復になるので、発電用途では成立しない
  (§4.11.2 のモーター直結と同じ理屈)。**使うとしたら Firebox(石炭)か Industrial Diesel Furnace**

> **船の発電系としての位置づけ**【ユーザー知見 2026-09-23】: **蒸気は捨ててもいいし、Condenser で真水に戻して再利用してもいい。**
> 捨てれば真水が減っていくだけで、閉ループにすれば減らない。**どちらにせよ趣味の範囲。**
> ディーゼル + Generator(§4.9.7)に比べ部品が大きく(Boiler 500・Turbine 500)、Coolant ループも2系統要る。
> **趣味で選ぶもの。**
> ただし **Industrial Diesel Furnace は §4.9.6 に既載のとおり「燃料を消費する熱源」**なので、
> 暖房・造水など「熱が欲しいだけ」の用途には単体で使える。

#### 4.16.2 固形資源(石炭・鉱石)の搬送

**流体でも電力でもない第4の系統。** 石炭・鉱石はこの系を通って運ばれる。

| 部品 | ファイル | ノード |
|---|---|---|
| Hopper / Medium / Large | `steam_coal_hopper*` | **num `Fill Level`(out)**。*"Pouring resources into the receptacle will add it to the connected system."* |
| Duct / Medium / Large | `steam_coal_duct*` | **num `Fill Level`(out)**。搬送路 |
| **Flexible Duct** | `steam_coal_flex` | **rope `Duct`** — **rope リンクで繋ぐ**(§4.15.1 と同じ第3方式) |
| Funnel Duct | `steam_coal_funnel` | bool `Open`。*"slowly move resources out of the connected system"* |
| Vacuum Duct | `steam_coal_vacuum` | bool `Active` + elec。前方のダクト/ホッパーから吸い上げる |

- **`Fill Level` が num で出るので、石炭残量は Lua から読める。** 燃料残量計(FUELSYS)と同じ作りにできる
- 単位(個数か割合か)は原文に無い(§7)

#### 4.16.3 原子炉(2部品)

| 部品 | ファイル | mass | ノード |
|---|---|---|---|
| Nuclear Fuel Assembly | `steam_nuclear_fuel_assembly` | 80 | bool `Release Fuel Rod`(in)/ **num `Fuel Rod Temperature`(out)** |
| Nuclear Control Rod | `steam_nuclear_control_rod` | 100 | **num `Insertion Target`(in)/ num `Insertion`(out)** |

原文: 燃料棒はワークベンチの**ウラン地金を消費**する。制御棒は*"decrease the rate of reaction of adjacent rods"*。

- **`Insertion Target` を入れて `Insertion` が返る = 位置指令と実測値のペア。** 挿入は即時ではなく追従する
- **反応度制御は「隣接する燃料集合体の温度を見て制御棒の挿入量を決める」PID**になる。
  熱の取り出しは 4.16.1 と同じ Coolant ループ(集合体に Coolant ノードは無いので、**周囲の流体部品で受ける**と思われる。未検証)

#### 4.16.4 採掘・漁労(表のみ)

| 部品 | ファイル | ノード |
|---|---|---|
| Oil Rig Well Head | `oil_rig_well_head` | bool `Anchor` / `Is Anchored` / **num `Drill Depth`・`Well Depth`(m、out)** |
| Oil Rig Rotary Table | `oil_rig_drill_driver` | bool `Clamp` / power `Torque` / num `RPS` |
| Oil Rig Drill Connector / Clamp | `oil_rig_drill_connector` / `_grabber` | bool `Clamp Rod` / `Rod Clamped` / num `Slider Velocity` / bool `Connect/Disconnect` / `Connector Aligned` |
| Oil Rig Drill Swivel | `oil_rig_drill_swivel` | bool `Clamp Rod` / `Rod Clamped` / fluid `Fluid In`・`Out`(掘削スラリーと原油) |
| Oil Rig Rod Storage / Pumpjack | `oil_rig_drill_storage` / `oil_rig_pumpjack` | bool `Rod Stored` / fluid `Fluid Out` |
| **Net Anchor** | `rope_hook_net` | rope `Net Node` / bool `Release Catch`・`Extend`・`Retract` / **comp `Net Data`: num1 = 網の充填率、num2 = 展張率**。**4個を rope リンクでループ状に繋ぐと網になる** |
| Lobster Pot | `lobster_pot` | bool `Release` / **num `Fill Level`** |
| Mineral Converter | `mineral_converter` | num `Mineral Level` / fluid。**非掲載部品**(§4.10.6) |
| Water Extractor | `water_extractor` | elec / bool `Process` / fluid `Water Out`。含水鉱石から水を取る |

- Oil Rig 一式は**掘削リグを自作するための部品**で、ノードは素直(指令 bool + 状態 bool + 位置 num)。
  **`Drill Depth` / `Well Depth` が m で出る**ので、自動化を書くなら素直に書ける
- **Net Anchor は漁業用だが、rope を4本ループで張る**という独特の接続。`Net Data` で充填率が取れる

#### 4.16.5 この章で見えた一般則

- **状態は必ず num か bool で出る。** `Coal Level` / `Diesel Level` / `Fill Level` / `Temperature` / `Drill Depth` など、
  **新しい部品ほど「中身の量」を素直に出す。** 古い部品(タンクの `Tank Content` を割合で出さない等)との差
- **位置指令型は「目標」と「実測」が対になる**(制御棒の `Insertion Target` / `Insertion`、Piston の `Piston Position`)。
  §4.13.4 の Pneumatic Piston と同じ設計。**追従を前提に書ける**
- **rope リンクは固形資源(Flexible Duct)と網(Net Anchor)にも使われている。** §4.15.1 のウィンチ専用ではない

---
