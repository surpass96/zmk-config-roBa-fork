# CLAUDE.md

roBa（トラックボール付き分割キーボード）用の ZMK ファームウェア設定リポジトリ。

## 構成

- `config/roBa.keymap` — キーマップ本体。keymap-editor（keymap-editor[bot]）からも更新される
- `config/roBa.json` — keymap-editor / keymap-drawer 用のレイアウト定義
- `config/west.yml` — ZMK は `v0.3-branch`、`zmk-pmw3610-driver` はコミット固定
- `boards/shields/roBa/` — シールド定義。右（roBa_R）が central でトラックボール（PMW3610）付き
  - `roBa_R.overlay` — 自動マウス（`zip_temp_layer`）の設定
  - `roBa_R.conf` — トラックボールの CPI・精密モードの CPI など
- `naginata/` — 薙刀式の ZMK モジュール（surpass96/zmk-naginata の 6d852bf を取り込んだもの）。`zephyr/module.yml` でこのリポジトリ自体をモジュールとしてビルドする。右（central）でだけ動く。OS は roBa_R.conf で Windows
- `build.yaml` — ビルド対象（roBa_R は ZMK Studio 用 snippet 付き、roBa_L、settings_reset）
- `keymap-drawer/` — キーマップ画像（自動生成）。Draw Keymap ワークフローを手動実行して更新する
- `wallpaper/` — キーマップの壁紙（PC 用 2560×1600（WQHD と 1920×1200 の両方で使える）・3840×2160、iPhone 12 用 1170×2532、iPad 11 インチ横向き用 2360×1640（ロック画面用・ホーム画面用）。それぞれ `-light` 付きのライトテーマ版もある。HTML を Chromium で撮影した PNG（ライト版の HTML はダーク版の色を置き換えたもの）。キーマップを変えたら手で作り直す

## レイヤー構成

| # | 名前 | 入り方 | 中身 |
|---|---|---|---|
| 0 | BASE | – | 大西配列＋ホームロウモッド |
| 1 | SYM | 左親指 Space 長押し | 左手 = 演算子・計算、右手 = 括弧・引用符・その他。全 32 記号を含む |
| 2 | NUM | 左親指 Tab 長押し | 右手 = 数字（上 7 8 9 / ホーム 1 2 3 / 下 4 5 6、親指 0）と数字に付く記号、左手 = SYM の左手と完全に同じ |
| 3 | NAV | 右親指 Enter 長押し | 右手 = 矢印（逆 T 字）と Esc・CapsLock、人差し指の内側の列 = 音量＋/−/ミュート、小指 = 輝度＋/−・範囲スクショ（Win+Shift+S）、下段 = 前の曲/再生・停止/次の曲。左手 = F1〜F12・修飾キー |
| 4 | SYS | Space＋Enter の同時押しコンボ | 左手 = BT 接続先 0〜4・BT 解除・USB/BT 切替、左右の内側の列 = 再起動・ブートローダ（押した側の半分にだけ効く）。すべて 1 回押し |
| 5 | MOUSE | ボールを動かすと自動 | 右手 = クリック・戻る/進む・タブ操作・スクロール・精密モード |
| 6 | SCROLL | R を押したまま（BASE / MOUSE） | 全 &trans。このレイヤー中はボールでスクロール |
| 7 | SNIPE | MOUSE の右小指ホームを押したまま | 全 &trans。このレイヤー中はカーソルが半分の速さ |

レイヤーを追加・並べ替えする時は、keymap 冒頭の `#define`、`&trackball` の `scroll-layers` / `snipe-layers`、roBa_R.overlay の `zip_temp_layer` のレイヤー番号を合わせて更新すること。

### 親指キー（BASE）

左：外側 = 1 回押しでかな（IME オン）・2 回押しで英数（IME オフ）（tap-dance `td_ime`）/ Space（長押し SYM）/ Tab（長押し NUM）、右：BS / Enter（長押し NAV）。右下の端は Caps（他のレイヤーでは使わない）。

### BASE の記号（Shift の組み合わせは US/JIS の配列ではなく意味で決めている）

`,` ⇧ `?` / `.` ⇧ `!` / `-` ⇧ `=` / `;` ⇧ `+` / `/` ⇧ `\`

### コンボ

- IME 切替：左 A+O または I+A = 英数（LANG2）、右 T+N または N+S = かな（LANG1）。判定 50ms。文字を打った直後 150ms は同時押しにならない（`require-prior-idle-ms`。コピペ・SYS には付けていない：キーボードで範囲選択した直後のコピーが効かなくなるため）。左親指外側の 1 回押し / 2 回押しでもかな / 英数
- コピペ：左手下段 Z+X / X+C / C+V = Undo / Copy / Paste（全レイヤーで有効）
- SYS：Space+Enter

### ホームロウモッド

E/I/A/O = Win/Alt/Ctrl/Shift、T/N/S/H = Shift/Ctrl/Alt/Win（`hml` / `hmr`。外側から Win/Alt/Ctrl/Shift）。反対の手のキーか親指キーと組み合わせた時だけ修飾キーになる（`hold-trigger-key-positions`）。NAV の左ホームロウも同じ並び（E/I/A/O = Win/Alt/Ctrl/Shift）。

### マウス

- 自動マウスはドライバの automouse ではなく ZMK 標準の `zip_temp_layer`（`roBa_R.overlay`、ボールを動かして 700ms 有効）。`excluded-positions` 以外のキーを押すと即解除され、BASE の文字が入力される。左・中・右クリックは離して 250ms 後に解除される（keymap の `click_off_*` マクロ＋`mouse_off`。クリック直後に打った文字がマウス操作にならないように）。`excluded-positions` は MOUSE レイヤーのキー配置と合わせること。
- スクロールと精密モードは keymap の `&trackball { scroll-layers = <SCROLL>; snipe-layers = <SNIPE>; }`。精密モードの速さは roBa_R.conf の `CONFIG_PMW3610_SNIPE_CPI`（通常 400、精密 200）。

## 記号配置の方針（SYM / NUM を編集する時）

優先順位の高い順に：

1. **覚えやすさ** — 左手 = 演算子・計算、右手 = 括弧・引用符・その他。対になる記号は上が開き・下が閉じの縦並び（`[ ]` `( )` `{ }` `< >`）。SYM / NUM でも `, . -` は BASE と同じ位置。NUM の左手は SYM の左手と同じ。`:` は SYM / NUM とも右親指（BS の位置）
2. **頻度** — 日本語 ＞ Java ＞ Excel ＞ Python の順に考慮（日本語の「」になる `[ ]` を右人差し指に置いているなど）
3. **連打する記号**（`== ++ // ** && || \\` など）は人差し指と中指に置き、小指・薬指・人差し指の内側の列には置かない
4. **内向きロール** — `!= <= >= += *=`（左手で `=` へ）、`("` `('`（右手で中指から人差し指側へ）

その他：

- 右利きなので、数字・矢印は右手。
- 内側の 4 キー（人差し指を横に伸ばす列）は押しにくいので、まれにしか使わないものだけを置く。
- 右下の端のキーは使いにくいので、BASE の Caps 以外は空けておく。

## 前提・方針

- ロータリーエンコーダーは使っていない（関連設定は削除済み）。
- 数字は `N0`〜`N9` を使う（`KP_NUMBER_x` は NumLock に依存するため使わない）。
- **ホスト PC は JIS 配列。** キーマップは US 配列のキーコードで書き、PC 側の USKey2JP で変換している。`\` は日本語環境では `¥` として表示・入力される。
  - キーマップを編集する時、必要に応じて「ファームウェア側で JIS 配列に対応する方法」（例: `(` → `&kp LS(N8)`、`@` → `&kp LBKT`、Shift 時の記号が異なるキーは mod-morph）を提案すること。ただし現状は US キーコード + USKey2JP の運用を継続しているので、勝手に書き換えない。
- 主な用途は日本語の文書、Java（Eclipse。閉じ括弧・引用符は自動補完される）、Excel、Python。

## 編集時の注意

- ユーザーは keymap-editor でも直接編集・保存する（ブランチにコミットが入る）。こちらで編集する前に必ず最新を fetch し、ユーザーの変更を上書きしていないか差分を確認すること。
- keymap-editor で保存すると整形（空行・桁揃え）が変わり、`roBa.json` に `id` などが追加されるが、動作には影響しない。
- 試作は `try1` / `try2` などのブランチで行い、固まったら main に反映している。

## CI

push / PR ごとに `.github/workflows/build.yml`（ZMK 公式 build-user-config）が走り、ファームウェア（.uf2）を生成する。変更後はこのビルドが通ることを確認する。
