## 3. Lua API 関数リファレンス (MC Lua, ゲームバージョン v1.15.1時点で確認)

### input / output
```lua
input.getBool(index)     -- index: 1-32, 戻り値: boolean
input.getNumber(index)   -- index: 1-32, 戻り値: number

output.setBool(index, value)   -- index: 1-32
output.setNumber(index, value) -- index: 1-32
```
モニタのcomposite inputには特殊な意味を持つchannelがある(モニタ直結のLuaスクリプトの場合):
- Number 1: モニタ解像度X / 2: 解像度Y / 3,4: Input1(既定Qキー)の押下座標X,Y / 5,6: Input2(既定Eキー)の押下座標X,Y
- Bool 1: Input1押下中か / Bool 2: Input2押下中か

### property
```lua
property.getNumber(label)  -- 数値プロパティ(Sliderタイプもこちらで取得)
property.getBool(label)    -- 真偽値プロパティ
property.getText(label)    -- 文字列プロパティ
```
`label` はMCエディタで設定したプロパティ名(大文字小文字を区別)。ユーザーが調整可能な設定値を持たせたい場合に使う。

**【重要】未設定のプロパティは 0 を返す(nil ではない)。** そのため各プロジェクトは次の形を使っている。

```lua
function pn(n, d) local v = property.getNumber(n) if v == 0 then return d end return v end
```

> **この形は「そのプロパティに 0 を設定できない」という副作用を持つ。** 0 を書いても既定値に
> 読み替えられてしまう。**0 を意味させたいときは負値を入れ、読み出し側で 0 へクランプする。**
> (`Obj 1171 MRM` の遅延プロパティがこの方式。`math.max(0, pn("Ignite Delay", 30))`)

`pn("名前", 既定値)` の**2引数形に統一すること**。MCBENCH がこの正規表現で走査して Property Number
部品を自動配置するため、3引数だと部品が置かれず実機で調整できなくなる。

### screen
```lua
screen.setColor(r, g, b, a)   -- 各0-255、aは省略可(既定255)
screen.drawClear()
screen.drawLine(x1, y1, x2, y2)
screen.drawCircle(x, y, radius)     -- 【実機検証済み】8角形近似で描画される
screen.drawCircleF(x, y, radius)    -- 同上(塗りつぶし)
screen.drawRect(x, y, width, height)
screen.drawRectF(x, y, width, height)
screen.drawTriangle(x1, y1, x2, y2, x3, y3)
screen.drawTriangleF(x1, y1, x2, y2, x3, y3)
screen.drawText(x, y, text)              -- 1文字4x5px
screen.drawTextBox(x, y, w, h, text, h_align, v_align)  -- align: -1〜1
screen.drawMap(x, y, zoom)               -- zoom: 0.1-50、ワールドマップ描画
screen.setMapColorOcean(r, g, b, a)
screen.setMapColorShallows(r, g, b, a)
screen.setMapColorLand(r, g, b, a)
screen.setMapColorGrass(r, g, b, a)
screen.setMapColorSand(r, g, b, a)
screen.setMapColorSnow(r, g, b, a)
screen.setMapColorRock(r, g, b, a)
screen.setMapColorGravel(r, g, b, a)
screen.getWidth()
screen.getHeight()
```
モニタ座標系: 左上が `(0,0)`。x軸は右方向、y軸は下方向に増加。

### map
```lua
map.screenToMap(mapX, mapY, zoom, screenW, screenH, pixelX, pixelY) -- -> worldX, worldY
map.mapToScreen(mapX, mapY, zoom, screenW, screenH, worldX, worldY) -- -> pixelX, pixelY
```
`screen.drawMap` で描いたワールドマップ上の座標変換に使う。引数1〜5は両者共通で `mapX, mapY`(地図中心のワールド座標)、`zoom`、`screenW, screenH`(モニタ解像度[px])。

**【一次資料で確認】`drawMap` の `zoom` は「横方向に何km表示するか」を指定する引数。** `zoom=0.1` で100m幅、`zoom=50` で50km幅。縦横比が1:1でないモニタでも**横幅基準**で描画範囲が決まる。したがって:

```
メートル毎ピクセル = zoom × 1000 / screenW
```

| zoom | 表示幅 | 32px幅(1x1) | 96px幅(3x3) | 288px幅(9x5) |
|---|---|---|---|---|
| 0.1 | 100 m | 3.1 m/px | 1.0 m/px | 0.35 m/px |
| 1 | 1 km | 31 m/px | 10.4 m/px | 3.5 m/px |
| 10 | 10 km | 313 m/px | 104 m/px | 35 m/px |
| 50 | 50 km | 1563 m/px | 521 m/px | 174 m/px |

タッチで座標を指定するUIでは、この値がそのままタップ分解能になる。`screenW` は**モニタのタッチ出力 composite から受け取る**こと(罠8番により `onTick` で `screen.getWidth()` は呼べない。下記4.8節)。

### async (HTTP)
```lua
async.httpGet(port, url)          -- urlは "/" から開始。tickあたり1リクエストまで
function httpReply(port, request_body, response_body) end  -- 応答コールバック
```

**localhost限定。** スクリプトが `async.httpGet` を呼ぶと、**サーバー/クライアントの各マシンがそれぞれ自分のlocalhostへ**リクエストを出す。よってプレイヤー間の直接通信はできず、インターネット上のサイトにも到達しない(1章参照)。

| 項目 | 内容 |
|---|---|
| 宛先 | `127.0.0.1:<port>` 固定。ホスト名は指定できない |
| `url` | **`/` から開始する必要がある**(パス以降のみ)。クエリ文字列を付けられる |
| レート | **1 tick あたり1リクエスト**。超過分はキューに積まれる |
| 失敗時 | ポートで何もlisten していない場合、**`response_body` が正確に `"connect(): Connection refused"` という文字列になる**。例外にはならない |
| コールバック | `httpReply(port, request_body, response_body)`。`request_body` には送った `url` が入る |
| アドオンLua側 | 関数名が異なり `server.httpGet(port, request)` / `onHttpReply(...)`(0章参照) |
| 専用サーバー | ビークルLuaからの `httpGet` を**設定で無効化できる**。マルチ前提の設計では使えない可能性がある |

**【未検証】** 使用可能なポート番号の範囲、`url`・`response_body` の文字数上限。

#### 実測データのロギング用途(本ゲームでのHTTPの主用途)

外部と通信できない以上、**HTTPの実質的な用途は「ゲーム内の数値をホストPC上のプロセスへ吐き出すこと」**である。

- **送信データはクエリ文字列に載せる**。`async.httpGet(8080, "/log?t="..tick.."&x="..x.."&z="..z)` のようにし、PC側で待ち受けるHTTPサーバーがファイルへ追記する。レスポンスは捨ててよい
- **1 tick 1リクエストの制約**が実効サンプリングレートの上限(60 Hz)。複数値は1リクエストにまとめて詰めること。tickごとに複数回呼ぶとキューが伸び続け、実時間からどんどん遅延する
- レスポンスを使えば**PC側からゲーム内へ値を戻す**こともできる(パラメータのライブ調整、リプレイ入力など)
- `"connect(): Connection refused"` が返るだけで落ちないので、**サーバーを立て忘れても機体は正常に飛ぶ**。計測コードを常設したままにできる

実機計測が要る設計(弾道係数、旋回半径、センサーの符号規約など)では、モニタに数字を出して目視するより桁違いに効率が良い。

### callback
```lua
function onTick() end  -- composite input/output はここでのみ操作可能
function onDraw() end  -- screen.* 描画はここでのみ操作可能。モニタ接続数分呼ばれる
```

---

