# 漂流島 DRIFT ISLE — Claude向け開発メモ

ブラウザで動くローポリ無人島サバイバルFPS。`index.html` 1ファイル完結（three.js r128 を cdnjs から UMD で読み込み）。
セーブは localStorage キー `driftisle_save_v4`（地形生成を変えたら番号を上げる）。

## 作業ルール（ユーザー方針）
- 返答の説明は短く。テストは変更部分に関係するものだけ（Node での構文チェック程度）。
- 大きなコードを丸ごと表示しない。編集はアンカー文字列を grep してから置換。
- 返答は必ず日本語。説明は手順を具体的に短く。
- 変更は作業ブランチに push → PR を作成 → main へ自動マージ（merge commit 方式）。1 回の依頼 = 1 PR にして、問題があれば PR ページの「Revert」で戻せるようにする。
- モンスターは出さない。舞台は日本の南の島（亜熱帯）。採取キーは F。アイコンはドット絵（Pix クラスで生成）。

## 主な構成
- 地形: 自作の非インデックスメッシュ（SEG=150, `terrainH` は三角補間）。`rawHeight` に第2の山・尾根・段丘の崖・西側の海食崖。池 `ponds`、鉱脈 `veins`。
- 洞窟 `caves`（`genCaves`）: 格子軸にそろえた崖（`caveHeight`）に横穴。崖面の3マスは地形から抜き（`caveSkip`）、`buildCaves` の岩面＋トンネルで置換。中の床は `caveFloor`／壁は `collide`／雨よけは `inCave`。
- 木・岩・採取物: 40m 区画×種類ごとの InstancedMesh。`cullChunks` で手動カリング。選択判定は自前（`rayEnt`）。
- 竹林: `take`（ホウライチク）。fbm マスクで群生。斧で伐ると `bamboo`（燃料・竹槍）。
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
Tab・I 持ち物 / M 地図 / X しゃがむ切替（`pl.crouchOn`、ダッシュで解除。低速・静音・動物が逃げにくい）/ Z 解体 / Q 捨てる / R 回転 / Esc 閉じる
