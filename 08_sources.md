## 8. 出典

- **ゲーム内 Lua ブロックの Help "Meta from the devs"**(開発者の記述。§0a。最優先)
- [teinishi「Stormworks の数値信号に30bit詰める」](https://teinishi.hateblo.jp/entry/stormworks-number-30bit)(数値信号の float32 表現と Composite Binary 変換のビット対応。§2)
- [Radar - Official Stormworks Wiki](https://stormworks.fandom.com/wiki/Radar)
- [Gameplay/Workbench/Components/Sensors - Official Stormworks Wiki](https://stormworks.fandom.com/wiki/Gameplay/Workbench/Components/Sensors)
- [Wiki/Guides/Lua/Exploring the Stormworks Lua API](https://stormworks.fandom.com/wiki/Wiki/Guides/Lua/Exploring_the_Stormworks_Lua_API)
- [Wiki/Guides/Lua/Special Advice for Lua](https://stormworks.fandom.com/wiki/Wiki/Guides/Lua/Special_Advice_for_Lua)
- [Gameplay/Workbench/Lua Programming - Official Stormworks Wiki](https://stormworks.fandom.com/wiki/Gameplay/Workbench/Lua_Programming)
- [GitHub - Cuh4/StormworksMCLuaDocumentation](https://github.com/Cuh4/StormworksMCLuaDocumentation)(intellisense.lua、v1.15.1時点のMC Lua関数を網羅。本資料の3章のベース)
- [Steam Guide - C4V's guide on radar](https://steamcommunity.com/sharedfiles/filedetails/?id=2757245531)
- [Steam Guide - Creating a Simple Marine Radar](https://steamcommunity.com/sharedfiles/filedetails/?id=2673443823)
- [Steam Guide - C4V's guide on composite](https://steamcommunity.com/sharedfiles/filedetails/?id=2787297027)
- 作者のプロジェクト群(`Obj NNNN`。§10)の設計記録(非公開。実機検証済みの一次情報源)
- [Stormworks Asset Modding Wiki - Geometa](https://geometa.co.uk/wiki/stormworks/view/asset_modding)(コンポーネント定義XMLの一次ドキュメント。ボクセルグリッド0.25m・Y-up等の裏付け)
- [Stormworks Component Modding Wiki - Geometa](https://geometa.co.uk/wiki/stormworks/view/component_modding)(LUAコンポーネント・ロジックノードのインデックス規則)
- `Steam/steamapps/common/Stormworks/rom/data/definitions/*.xml`(バニラコンポーネント定義。プレーンテキストで直接読める一次情報源。758ファイル以上)
- **`Steam/steamapps/common/Stormworks/sdk/data/game_constants.xml`(ビークル物理・空力・流体・浮力・センサーノイズの全定数。説明と既定値つきの平文。§4.3d の出典)**
- `Steam/steamapps/common/Stormworks/stormworks64.exe` の `.data` 内静的テーブル(砲弾の初速・抗力・発射間隔。§4.3b の出典。オフセットは `Obj 9901 SWSIM` の手元資料。非公開)
  - 4.1(レーダー)・4.2(Physics Sensor)・4.2b(Astronomy Sensor)・4.3(各種センサー)・4.3b(砲・砲塔)の記述は、`Obj 1872 AAFCS` の作業中にこれらの定義ファイルを直接読解して作成・訂正したもの。**公式Wikiや旧版資料と食い違う箇所は定義ファイル側を優先している**(Wind Sensorのノード順、`Radar (Missile)` の `Gimbal Input` 不在など)
- [Steam Guide - The Math behind the XML](https://steamcommunity.com/sharedfiles/filedetails/?id=3350900327)(`r`回転行列の構造の一次情報)
- [Steam Guide - Physics Sensor / euler angles](https://steamcommunity.com/sharedfiles/filedetails/?id=3302971632)(**オイラー角の単位・軸対応・適用順・左手系の一次情報**。4.2節の出典)
- [Reddit - Stormworks XML rotations to Euler rotations (r/Stormworks)](https://www.reddit.com/r/Stormworks/comments/1hpvvdx/stormworks_xml_rotations_to_euler_rotations/)(`r`行列の構造の補強、XML編集ウェッジの歪み現象の説明)
- Steamガイド(Physics/Astronomy SensorのEuler出力に関するもの、URL未記録)- Stormworksが左手系座標系である旨の記載。要再確認・裏取り。
- [FLUID - Stormworks: Build and Rescue_JP Wiki](https://wikiwiki.jp/sbarjp/FLUID)(タンク容量・バルブの無給電時挙動・ポート系の性能差なし。§4.9 の出典。**ユーザー推奨: 疑問が出たらまず JP Wiki を覗く**)
- JP Wiki の [VEHICLE CONTROL](https://wikiwiki.jp/sbarjp/VEHICLE%20CONTROL) / [PROPULSION](https://wikiwiki.jp/sbarjp/PROPULSION) / [ELECTRIC](https://wikiwiki.jp/sbarjp/ELECTRIC) / [MECHANICS](https://wikiwiki.jp/sbarjp/MECHANICS) / [クラフトガイド/エンジン系](https://wikiwiki.jp/sbarjp/クラフトガイド/エンジン系)(§4.10・§4.11・§4.9.7 の出典、2026-09-18。**非公式。XML と一致する数値は XML 転記の可能性があり、独立の裏付けとしない**。§0b 規則5)
- [検証 - Stormworks: Build and Rescue_JP Wiki](https://wikiwiki.jp/sbarjp/%E6%A4%9C%E8%A8%BC)(Emergency Beacon のパルス間隔と距離の式。§4.5f の出典。非公式・本資料では未検証)
- [note - 爆速流体輸送法](https://note.com/mumenry/n/n50fc3c069e62)(二次資料。圧力差駆動・ポンプ最大圧・合流で流量低下)
- 本セッションでの実機サンプル観察(ユーザー提供の複数ビークルXML: primitiveブロック単体、寄棟屋根小屋モデル、24パターン回転グリッドテスト等)

---

