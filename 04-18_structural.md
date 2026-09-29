### 4.18 構造・装飾部品(ブロック・窓・手すり・パイプ・可動部の従ブロック)

**【一次資料 2026-09-23】category=0(構造 39)/ 8(装飾 17)/ 15(窓 41)と、category 未指定 10 を読んだ結果。**
**ロジックノードはほぼ無い**(可動部の従ブロックだけが1本持つ)。Lua からは触れないが、
**質量と体積は喫水・復原性・カスタムタンク容量に効く**ので、そこだけ拾う。

#### 4.18.1 質量は「詰まっている体積」に比例する(【観察】)

| 部品 | voxel | mass |
|---|---|---|
| Block | 1 | **1** |
| Wedge | 1 | **0.5** |
| Pyramid | 1 | **0.25** |
| Inverse Pyramid | 1 | **0.75** |
| Wedge 1x2 / 1x4 | 2 / 4 | 1 / 2 |
| Pyramid 1x2 / 1x4 | 2 / 4 | 0.5 / 1 |
| Inverse Pyramid 1x2 / 1x4 | 2 / 4 | 1.5 / 3 |
| Pyramid 2x2 / 2x4 / 4x4 | 3 / 6 / 10 | 1 / 2 / 4 |
| Inverse Pyramid 2x2 / 2x4 / 4x4 | 4 / 8 / 16 | 3 / 6 / 12 |
| **Weight Block** | 1 | **10** |

- **Block 1 に対し Wedge は 0.5、Pyramid は 0.25、Inverse Pyramid は 0.75。**
  それぞれ立方体を半分・1/4・3/4 に削った形なので、**mass = 占有体積そのもの**と読める
- **§4.9.1 の「1 voxel = 15.625 L」と同じ考え方。** ゲームは体積を素直に扱っている
- **Weight Block は同体積で 10 倍**。原文: *"larger impact on the vehicle's centre of mass, making them useful for
  balance and stability."* 重心調整用
- 窓は同体積のブロックより軽い(Window 2x2 は 4 voxel で mass 3)

> **喫水計(HULLSYS)の設計に効く**: 排水量は船体の質量総和で決まり、**質量は形状ブロックの占有体積に比例する。**
> ウェッジを多用した船首は、見た目の体積より軽い。**カスタムタンク容量(内部ボクセル × 15.625 L)と合わせて、
> 「体積」で一貫して考えられる。**

#### 4.18.2 パイプ(`trans_*`、16種)— **流体と動力の配管**

`trans_straight` / `trans_angle` / `trans_corner` / `trans_t` / `trans_t_corner` / `trans_cross` /
`trans_cross_corner` / `trans_omni` と、それぞれの `_block_`(Enclosed)版。**全部 1 voxel・mass 1・ロジックノード無し。**

- **【ユーザー指摘 2026-09-23】現行の流体はこのパイプで接続する。動力も同じパイプ。**
  ファイル名が `pipe_*` ではなく `trans_*`(transmission)なので、名前で探すと見つからない
- **本資料は「流体はロジック接続の線で繋ぐ」と書いていたが、それは過去の仕様。**(§4.9.0 で訂正済み)
  **ただし現行システムにノード制の名残が残っている可能性がある** — 部品側の fluid / power ノードは存在し続けており、
  **どこまでがパイプ経由でどこからがノード直結なのかは切り分けられていない**(§7)
- 定義XMLでは `type="6"`、`trans_conn_type` 属性を持つ。値は Straight が 0、その他が 1 か 2 だが、
  **形状との対応が取れない**(`trans_cross`=2 に対し `trans_block_cross`=1 など)。**意味は未解読**
- **【ユーザー知見 2026-09-23】Enclosed 版(`trans_block_*`)はカスタムタンクの壁になる**(§4.9.4b)。**配管でタンクを貫通しても密閉が保てる**ので、区画内の取り回しに効く

#### 4.18.3 Physics Flooder(`physics_flooder`、2 voxel、mass 1)

原文: *"Floods an enclosed volume with massless physics, allowing for physics shape optimisation without
compromising vehicle integrity."*

- **囲った空間を「質量の無い物理形状」で埋める部品。** 大きな中空構造の物理演算を軽くするためのもの
- **【ユーザー確認 2026-09-23】カスタムタンク(§4.9.4b)とは競合しない。** 同じ密閉空間に両方成立する
- **【ユーザー知見】ただし人間が中に入れなくなるので、整備点検ができない。** ユーザーは基本的に採用していない。
  → **区画に人が入る設計(機関室・ビルジ・バラスト点検)なら使わない。** 使うのは本当に触らない空洞だけ

#### 4.18.4 可動部の従ブロック(category 0 側)

§4.3c / §4.13 の可動部は **A 側(ノードを持つ本体)と B 側(従ブロック)の2部品**で1組になっている。

| 部品 | ノード |
|---|---|
| `multibody_pivot_b` / `multibody_velocity_pivot_b` | 無し |
| `multibody_robotic_pivot_01_b` / `_b_fluid` | power または fluid 1本(通すだけ) |
| `multibody_velocity_pivot_01_b_fluid` / `_b_torque` | 同上 |
| `multibody_turret_small_b` / `medium_b` / `large_b`(Turret Ring) | 無し。**voxel 16 / 24 / 40、mass 14 / 22 / 36** |
| `multibody_pivot_torque_b`(Pivot (Power)) | power 1本 |

**B 側に power / fluid ノードがあるのが、§4.9.8・§4.11.1 の「可動部を跨いで動力・流体を通せる」の実体。**

#### 4.18.5 その他(表のみ)

| 部品群 | ファイル | 備考 |
|---|---|---|
| 窓 41種 | `window_*` | 1x1〜4x4、角度・ダイヤ・コーナー・舷窓(`window_port` / `window_porthole`)。**カスタムタンクの境界面になる**(§4.9.4b) |
| 手すり 12種 | `railing_*` | 直線・角・曲線・傾斜・対角。mass 2(曲線のみ 5) |
| 旗 3種 | `flag_*` | Small / Medium / Large |
| タイヤ(防舷材) | `tyre_small` / `tyre_large` | mass 4 / 12。着岸時の緩衝に |
| ハシゴ | `ladder_small` | 12 voxel。*"attach by interacting ... automatically detach at the top"* |
| 階段 | `stair_segment` / `stair_top` | 各 6 voxel、mass 6 |
| Static Block | `01_block_static` | *"This block cannot be removed."* **非掲載部品**(§4.10.6) |
| Handle | `handle` | 1 voxel、bool 1本。掴まり用 |

---
