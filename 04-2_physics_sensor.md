### 4.2 Physics Sensor

**【一次資料で確認済み】** `rom/data/definitions/physics_sensor.xml` の `Composite Output` ノード(type=5)のdescriptionを直接読解。**過去にチャンネル順が変更されたとの情報があるため、以下は「インストール済みの現行版」の内容**。バージョンが上がったら再確認すること。

原文(そのまま):
> Outputs [1] x position, [2] y position, [3] z position, [4] euler rotation x, [5] euler rotation y, [6] euler rotation z, [7] linear velocity x, [8] linear velocity y, [9] linear velocity z, [10] angular velocity x, [11] angular velocity y, [12] angular velocity z, [13] absolute linear velocity, [14] absolute angular velocity, [15] local z tilt, [16] local x tilt and [17] compass heading (-0.5 to 0.5) to the respective composite channels.

| ch | 内容 | 備考 |
|---|---|---|
| 1-3 | 位置 X, Y, Z | **要電力**(下記)。GPS同様パーツ自身の位置 |
| 4-6 | オイラー回転 X, Y, Z | **yaw/pitch/rollに素直には対応しない。** 軸の割当は X軸まわり=pitch / Y軸まわり=yaw / Z軸まわり=roll と推定されるが、順序と符号は**未検証** |
| 7-9 | 線速度 X, Y, Z | **【確定】ローカル系**(下記) |
| 10-12 | 角速度 X, Y, Z | **単位は turn/s**(ユーザー 2026-09-30)。ローカル系かワールド系かは未確認(線速度 ch7-9 はローカル) |
| 13 | 絶対線速度(スカラー) | |
| 14 | 絶対角速度(スカラー) | |
| 15 | local z tilt | 原文が明示的に "local" と書いている |
| 16 | local x tilt | 同上 |
| 17 | コンパス方位(-0.5〜0.5) | ※部品をxml改造で非表示にするとこの出力も乱れる、との報告あり |

`tooltip_properties` 原文: *"Outputs physics information to a composite logic node. **Requires Electric for GPS position functionality.** This component is directed along its local Y axis."*

- **位置出力(ch1-3)には電力配線が必要**。他のchは電力なしでも出るが位置だけ死ぬ、という嵌り方をする
  - 【ユーザー談 2026-09-28】機体の電源が落ちると位置は **0, 0, 0** になる。マイコンは電源と関係なく動き続けるので、Lua からは「位置が 0,0,0 = 電源断(入力不良)」と見分けられる(原点が地中である件は §7)
- 部品の指向軸は**ローカルY軸**(他のセンサーの「青矢印=Z前方」とは異なるので設置時に注意)
- 矢印の色対応: 赤=X軸、緑=Y軸、青=Z軸

**【実機検証済み・web情報とは軸符号が異なる】**(`Obj 1822 MAWS`より): ローカル軸は **X=右方向が正、Y=上方向が正、Z=前方向が正**。一般に説明される「Stormworksローカル軸」の解説とは符号/割当が異なるため、他資料の軸定義を鵜呑みにせず、この実機検証結果を優先すること。

#### 【確定】ch4-6 のオイラー角の規約と単位

**【一次情報: コミュニティガイド + `Obj 9101 EULERTEST` の実機テスト】**

- **単位は radian。turn ではない。** ゲーム内の他のほとんどの角度(レーダーaz/el、チルト、コンパス、砲塔のCurrent Rotation)が turn なので**極めて間違えやすい**。実際に `Obj 1872 AAFCS` の3ブロックと `Obj 1812 RDR` が揃って `×2π` していた
- **ch4 = euler x、ch5 = euler y、ch6 = euler z**(素直な対応)
- **適用順は x → y → z。しかも extrinsic(固定world軸まわり)の回転**
- **Stormworksは左手系**で、回転は左手則に従う(**時計回りが正**)

機体ローカル → world の変換はこれだけ:

```lua
function rot2(a,b,r) local c,s=math.cos(r),math.sin(r) return a*c-b*s, a*s+b*c end
-- ex,ey,ez は Physics Sensor ch4,5,6 をそのまま(ラジアン、スケーリング不要)
function b2w(x,y,z) y,z=rot2(y,z,ex) z,x=rot2(z,x,ey) x,y=rot2(x,y,ez) return x,y,z end
-- 逆変換は順序を逆にして符号を反転
function w2b(x,y,z) x,y=rot2(x,y,-ez) z,x=rot2(z,x,-ey) y,z=rot2(y,z,-ex) return x,y,z end
```

> **教訓**: この規約は**コミュニティガイドに明記されていた**。実機治具で96候補を総当たりする前に検索すべきだった。**未知の仕様に当たったらまず調べる**こと。治具(`Obj 9101 EULERTEST`)自体は残してあるが、規約が確定した今は「配線ミスの検出」用途に格下げ。

#### 【確定】ch7-9 の線速度はローカル系

**【ユーザー確認済み】ch7-9 はローカル系。** XMLのdescriptionが ch15/16 にだけ "local" を付けているため一時world説を検討したが、書き分けは単なる表記の揺れだった。

- **worldで使いたい場合は機体姿勢で回す必要がある。** 変換は1か所に閉じ込めること
- 回し忘れると**機体が北向き(yaw=0)の時だけ正しく動き、他の方位で壊れる**。自機25m/s・東進(yaw=90°)でTOF1.5秒なら弾着が53mずれる(方位別の誤差表は `Obj 1872 AAFCS` の設計記録にある。非公開)。机上テストでは踏みにくい種類のバグなので注意

#### 座標系: Physics Sensor は Y-up、GPS は Z-up

**【ユーザー確認済み】この2つは軸の取り方が違う。**

| 系 | 軸 |
|---|---|
| Physics Sensor / Astronomy Sensor | **Y-up** (X=右/東, **Y=上**, Z=前/北) |
| GPS / ワールドマップ | **Z-up** (X=東, Y=北, Z=上) |

**プロジェクト全体をどちらかに統一する必要がある。** `Obj 1872 AAFCS` は **Y-up (X=東, Y=高度, Z=北)** を採用した。理由は、Physics Sensorの姿勢・速度・傾きが全てY-up前提であること、Astronomy Sensorの原文が "[2] y position in metres (earth sea level is 0)" と明記していること、弾道式の重力項がY軸と一致すること。

Z-upの方が直感的という意見はもっともだが、**変換対象がGPSの2chと高度計の1chだけで済むY-upの方がシステム的に妥当**という判断。変換は次の1か所のみ:

```
world.X = GPS.Number1 (東)
world.Y = Altimeter    (高度)
world.Z = GPS.Number2 (北)
```

### 4.2b Astronomy Sensor

**【一次資料で確認済み】** `astronomy_sensor.xml` の `Data` ノード(type=5)。Physics Sensorと似ているが**11ch構成で線速度が無い**。

> Outputs [1] x position in metres, [2] y position in metres (earth sea level is 0), [3] z position in metres, [4] euler rotation x, [5] euler rotation y, [6] euler rotation z, [7] angular velocity x, [8] angular velocity y, [9] angular velocity z, [10] local z tilt and [11] local x tilt to the respective composite channels.

**注目点: 「[2] y position in metres (earth sea level is 0)」** — この部品系統では **Yが高度**であることが原文で確定している。ワールド座標をY-up(X=東, Y=高度, Z=北)で扱う根拠のひとつ。

