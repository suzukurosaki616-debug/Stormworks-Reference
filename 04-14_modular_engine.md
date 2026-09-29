### 4.14 モジュラーエンジン(24部品)— 構成と、Lua から見える窓口

**【一次資料 2026-09-23】`rom/data/definitions/modular_engine_*.xml` 24 ファイルを読んだ結果。**
一体型エンジンは §4.9.6(ノード)と §4.11.3(運転知見)。Gearbox / Clutch の比率は §4.11.4。

#### 4.14.1 組み立ての骨格

原文をまとめると:

- **Crankshaft が核。** *"Attach cylinders to the outer surfaces to generate power."*
  複数の Crankshaft ブロックを隣接させて大きなエンジンにできる
- **Cylinder は Crankshaft の外面に付ける。** *"The cylinder requires a manifold to move air, fuel, and exhaust through the cylinder."*
  **隣接して並べた Cylinder は Manifold を1つ共有できる**(Fuel / Air / Coolant / Exhaust のいずれも)
- **Manifold は Cylinder に付ける**: Fuel(`modular_engine_intake_manifold`、表示名は *Fuel Manifold*)/ Air / Coolant / Exhaust(Straight・Corner)
- **Drive Belt に付けるもの**: Starter / Alternator / Fluid Pump
- **Crankshaft に付けるもの**: Clutch、Temperature Sensor、Flywheel
- **Crankshaft Converter**(3→1 / 5→3)でサイズの違う Crankshaft 間のトルクを変換

サイズ: Crankshaft 1x1 / 3x1 / 3x3 / 5x5(mass 1 / 9 / 27 / 100)、Cylinder 1x1 / 3x3 / 5x5(1 / 27 / 100)、
Clutch 1x1 / 3x3 / 5x5(1 / 9 / 25)、Flywheel 1x1 / 3x3 / 5x5(**mass 100 / 200 / 300**)。

#### 4.14.2 Lua から見える窓口(これだけ)

**出力**

| ノード | 部品 | 内容 |
|---|---|---|
| num `RPS` | Crankshaft / Flywheel | 回転数 |
| num `Temperature` | Temperature Sensor(Crankshaft に付ける) | 温度 |
| comp `Composite Data` | **Cylinder(1本ごと)** | **num1 = Air Volume / num2 = Fuel Volume / num3 = Temperature** |

**入力**

| ノード | 部品 | 内容 |
|---|---|---|
| num `Throttle` | **Air Manifold / Fuel Manifold(別々)** | 4.14.3 |
| num `Clutch Pressure` | Clutch / Alternator / Fluid Pump | 0〜0.3 は伝達せず、0.3〜1.0 で段階的(§4.11.4) |
| bool `Starter` + elec | Starter | *"apply an inefficient force on the engine to get it started"* |

- **【ユーザー知見 2026-09-23】Cylinder の出力は気筒ごとにわずかにばらつくが許容誤差。
  代表1本を読めば足りる。** 全気筒ぶん comp を集める必要はない
- Alternator / Fluid Pump が `Clutch Pressure` を持つのは、**ベルトからの切り離し**のため。
  補機を切って主機の負荷を軽くする、という制御ができる

#### 4.14.3 空燃比 — Air がスロットル、Fuel がフィードバック

**【ユーザー知見 2026-09-23】Air Manifold の `Throttle` を出力指令にし、Fuel 側はフィードバック制御で追従させる。**
Cylinder の comp(num1 Air / num2 Fuel)を読んで比を目標値に寄せる。

- 【JP Wiki クラフトガイド/エンジン系】運転可能な空燃比は **12〜16 程度、理想 14.6**。
  簡易制御なら **Air Manifold への入力の半分を Fuel Manifold に入れる**
- **【ユーザー知見】ポンプ等で空気を圧縮して送り込めば、その分だけ燃料を増やせる(過給)。**
  → 出力を上げたいなら、スロットルを上げるのではなく**吸気側を加圧する**のが筋
- 【JP Wiki】**RPS 2 以下でエンスト**(一体型と同じ)

> **これが FUELSYS の機関制御の形を決める。** 「スロットル1本」ではなく、
> **Air = 指令、Fuel = 空燃比を保つ従属ループ**という2段構えになる。目標 RPS への追従はさらにその外側。

#### 4.14.4 設計上の判断材料

- **Flywheel は積まなくてよい。【ユーザー知見 2026-09-23】** *"acts as a momentum store ... but also makes the engine
  harder to start"*(原文)だが、**シリンダ1個でも単気筒とは思えない滑らかな回転が出る**ので、
  mass 100〜300 を払う価値は薄い
- **Alternator で主機の電力は賄えない。【ユーザー知見】** 車の電装品を動かす程度の位置づけで、負荷は軽いが出力もほぼ無い
  (§4.9.7 の `electric_magnitude` は Generator 0.6〜0.85 に対し **Alternator 0.05**)。
  **モーターを回すような電力が要るなら、Crankshaft から別途 Generator を回すこと**
- **一体型との使い分け【ユーザー知見】: 勝てるのは「サイズを自由に決められる」点。**
  船体に合わせた形・出力のエンジンを組める。**燃費と出力は一体型と大差ない**(検証データを見た記憶、とのこと。§7)
  → **省スペース・省手間を取るなら一体型で十分。** 形の都合が出たときにモジュラーを選ぶ

#### 4.14.5 未解決

- 出力(トルク)が**気筒数・Crankshaft サイズとどう対応するか**は定義XMLに無い
- **燃料消費モデル**は一体型と同じく未検証(§4.9.9)
- Coolant Manifold は `Coolant A`(in)/ `Coolant B`(out)の2口。**冷却ループの要求流量**は不明
- 温度の上限(一体型は 115〜120 ℃ で爆発、§4.11.3)がモジュラーでも同じかは未確認

---
