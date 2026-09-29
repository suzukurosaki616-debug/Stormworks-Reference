### 4.12 計器・表示部品(Dial / Gauge / Digital Display / Instrument Panel / 指示灯・音)

**【一次資料 2026-09-23】`rom/data/definitions/*.xml` のうち category=6(表示)24 ファイルを読んだ結果。**
Monitor 10種は §4.5b、Clock / Compass Ball / Laser Beacon / Paintable Indicator は §4.5・§4.5c に既載。

> **ノード構成はほぼ全部が「num か bool を1〜2本 + bool `Backlight` + elec `Electric`」で、見たまま。**
> 差が出るのは**プロパティで何が設定できるか**と**範囲外の値をどう扱うか**なので、そこだけ書く。

#### 4.12.1 Instrument Panel(`instrument_display`、mass 1)— **1ブロックに計器4つ**

| ノード | mode | 型 | 内容 |
|---|---|---|---|
| `In Signal` | 入力 | comp | 表示するデータ |
| `Out Signal` | 出力 | comp | **パネル上のボタン等で操作した結果**。原文: *"data set by buttons on the display"* |
| `Electric` | — | elec | |
| `Backlight` | 入力 | bool | |

原文: *"The 4 instruments can be selected in the properties window. Each type of instrument has options for choosing
the composite channels that they read from or write to. Composite signals are bridged from the input node to the
output node to enable displays to be chained together."*

- **【ユーザー知見 2026-09-23】非常に便利。1ブロックに4つ入り、選べる計器は
  Dial / Indicator / Gauge / Button / Arrow button / Flip switch / Seven segment / Radial segment / Bar segment。**
  検証時の確認用にも、実機の計器盤にもそのまま使える
- **各計器が読む/書く composite チャンネルをプロパティで個別に指定する。** Lua 側は composite を1本出すだけでよく、
  **Dial を何個も並べて配線する必要がない**
- **`In Signal` → `Out Signal` へブリッジされる**ので、パネルを数珠つなぎにできる。入力のチャンネルはそのまま下流へ流れ、
  ボタン類の結果が乗って出てくる
- 原文: *"Instruments marked as '(On/Off)' require multiple on/off signals to control each of their segments individually."*
  → **'(On/Off)' 型のセグメント計器はセグメント数ぶんの bool を食う。**
  **【ユーザー知見 2026-09-23】セグメント計器は num で渡すのが普通なので、bool 型は使わなくてよい。**
  **num で渡す場合も「2進数」と「数値」をプロパティで選べる**
- **【ユーザー知見】いずれにせよ composite のチャンネルは不足する。** bool 32 / num 32 しかないので、
  **計器盤と警報バス(`Obj 1951 ALARM`)を同じ composite に相乗りさせる設計にはしないこと。**
  パネル用の composite は分ける

> **入力と出力が同じ composite を共有するので、Lua 側は「自分が書いたチャンネル」と「パネルが書き返すチャンネル」を
> 分けて設計すること。** 警報バス(`Obj 1951 ALARM`)のチャンネル表と衝突させない。

#### 4.12.2 針・数値表示

| 部品 | ファイル | mass | 入力 |
|---|---|---|---|
| Dial | `dial` | 1 | num `Value to Display` + `Backlight` + elec |
| Gauge Display | `gauge_display` | 2 | num `Primary Display Value`(白針)/ num `Secondary Display Value`(**赤針**)+ `Backlight` + elec |
| Digital Display | `digital_display` | 2 | num `Value to Display` + `Backlight` + elec |

**【ユーザー知見 2026-09-23】Dial と Gauge は範囲外の扱いが逆**:

- **Dial**: Min/Max は「枠に対する針の振れ幅」を決めるだけで、**値が増え続ければ針は回り続ける(ラップする)**。
  原文も *"Values outside this range will cause the needle to wrap around"*。**回転計・積算計のように「回ってよい」量に向く**
- **Gauge Display**: **Min/Max でクランプされる。** 範囲外に振り切れない。**燃料残量・温度のように「振り切り」を見せたい量に向く**
- → **Lua 側でクランプするかどうかが部品によって変わる。** Dial に生値を入れると一周して嘘を表示するので、
  **Dial を使うなら Lua 側でクランプする**

**Digital Display**【ユーザー知見】:
- **小数点位置はプロパティで `0.0000` 〜 小数点無しの5段階**
- **表示は6桁。符号(±)は桁とは別に頭に付く**
- → 6桁に収まらない値(GPS 座標の生値など)は Lua 側でスケールするか分割する

#### 4.12.3 指示灯・その他

| 部品 | ファイル | 入力 | 備考 |
|---|---|---|---|
| Indicator Light | `indicator` | bool `Indicator Light` + elec | 色は塗装で決まる |
| Indicator Light (RGB) | `indicator_rgb` | **comp `Color Data`** + elec | 原文: composite の3チャンネルで RGB(各0〜1)。**プロパティで HSV モードに切替可**、読むチャンネルも指定可 |
| Artificial Horizon | `artificial_horizon` | `Backlight` + elec のみ | **信号入力が無い。** 部品自身が機体姿勢を見て動く。Lua からは何もできない |
| Compass Ball | `compass` | 同上 | 同じく自律。§4.3 の Compass **Sensor** とは別物 |
| Paintable Indicator | `sign` | `Backlight` + elec | 面を1マスずつ塗れる。加算塗りでバックライト面を塗る(原文)。銘板・ラベル用 |
| Viewing Scope | `viewing_scope` | video `Video Signal` + elec + bool `Occupied`(出力) | 4.12.4 |

**RGB 指示灯は composite 入力**なので、**Lua から色を出せる数少ない表示部品**。警報の色分け(緑/黄/赤)に使える。
`Indicator Light` は bool しか受けないので、色を変えたければ色違いを複数置くことになる。

#### 4.12.4 音と、覗き込み表示

**Buzzer**(`buzzer`、mass 1): bool `Buzzer` + elec。原文: *"Plays the selected sound when toggled from off to on."*
可聴範囲 30m。音の種類・音量・ピッチはプロパティ。

- **【ユーザー知見 2026-09-23】「1回しか鳴らない音」と「鳴り続ける音」の2系統があり、プロパティで切り替わる。**
  → **警報の設計時に、鳴動の継続を Lua で作るか部品に任せるかが変わる。** 一発音を選んだなら、継続警報は Lua 側で
  周期的に on/off を打ち直す必要がある。`Obj 1951 ALARM` の鳴動仕様はこれを踏まえて決めること

**Speaker (Small)**(`speaker`、mass 1): audio 入力 + elec + **bool `Is Transmit`(出力)**。可聴 15m。
Megaphone Speaker(`speaker_large` / `speaker_medium`、category 4)は範囲が広い。

**Viewing Scope**(`viewing_scope`、mass 5): 原文 *"Interact to display video data at full detail."*
**【ユーザー知見 2026-09-23】覗き込むと映像が全画面で出る、という認識でよい。**
描画は HUD と同じ扱いで、**黒が透明として処理される**(明度がそのままアルファになる、という理解)。
ただし**それが効くのはカメラ映像に重ねたときで、映像入力が無ければ背景は普通に黒になる。**

> **カメラ映像に MC の描画を重ねる構成でだけ効く話。** 照準環・数値オーバーレイをカメラ画に重ねると、
> **黒で塗ったつもりの背景が抜けてカメラ画が透ける**(重ねる目的ならそれが正しい)。
> 暗い色の線も薄くなるので、**重ねる画は明るい色で描く。**
> 映像入力なしで計器画面として使うなら、通常のモニタと同じ感覚で描いてよい。
> §7 の「モニタ描画はGPUベンダで変わる」報告とは別問題。

---
