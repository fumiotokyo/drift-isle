# 漂流島 DRIFT ISLE — Claude向け開発メモ

ブラウザで動くローポリ無人島サバイバルFPS。`index.html` 1ファイル完結（three.js r128 を cdnjs から UMD で読み込み）。
セーブは localStorage キー `driftisle_save_v3`（地形変更で v3 に更新）。

## 作業ルール（ユーザー方針）
- 返答の説明は短く。テストは変更部分に関係するものだけ（Node での構文チェック程度）。
- 大きなコードを丸ごと表示しない。編集はアンカー文字列を grep してから置換。
- モンスターは出さない。舞台は日本の南の島（亜熱帯）。採取キーは F。アイコンはドット絵（Pix クラスで生成）。

## 主な構成
- 地形: 自作の非インデックスメッシュ（SEG=150, `terrainH` は三角補間）。`rawHeight` に第2の山・尾根・段丘の崖・西側の海食崖。池 `ponds`、鉱脈 `veins`。
- 洞窟 `caves`: 斜面にドーム状の岩（`buildCaves`）。床は平坦化。`collide` で壁、`caveRoof` で屋根に乗れる、`inCave` で雨よけ（`sheltered`）。
- 木・岩・採取物: 40m 区画×種類ごとの InstancedMesh。`cullChunks` で手動カリング。選択判定は自前（`rayEnt`）。
- 草・シダ: 24m 区画を起動時に全生成（`DEC`）。
- 立てる場所: `standAt`（地形＋岩の上面 `rockSurf`＋設備の上面 `stTop`）。
- インベントリ: Tarkov 式グリッド（`bag{w,h,items[{id,n,d,x,y,r}]}`）＋手持ち8枠 `hot`＋防具 `armor`。
- 設備: `PROC` / `SLOTS` / `FUEL`。`s.st={fuel,tool?,in,out,prog,lit,burn}` を `updateStations` で処理。
- クラフト: 配列 `R`。`r.s` の設備を F で開いている（火の設備は点火中）時のみ可能。`cheat` で無条件。
- 時間付き動作: `startAction(label,dur,done,ctx)`。走る・インベントリ開閉で中断。
- 動物: ノヤギ／リュウキュウイノシシ。倒すと死体 → ナイフで解体（`BUTCH`, 収量 `KQ`）。
- 音: WebAudio で合成（`SFX`, `updateAudio` が環境音）。
- 地図: `buildMap`（最初から全域表示）/ M キー。洞窟も表示。
- 焚き火の上に乗ると火傷ダメージ（`updatePlayer`）。

## 操作
WASD / Shift ダッシュ / Space / 左クリック 採集・攻撃・設置 / 右クリック 食べる / F 採取・開く /
Tab・I 持ち物 / M 地図 / X しゃがむ（`pl.crouch`, 低速・静音・動物が逃げにくい）/ Z 解体 / Q 捨てる / R 回転 / Esc 閉じる
