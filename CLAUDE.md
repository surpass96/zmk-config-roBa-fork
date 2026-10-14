# CLAUDE.md

roBa（トラックボール付き分割キーボード）用の ZMK ファームウェア設定リポジトリ。

## 構成

- `config/roBa.keymap` — キーマップ本体。keymap-editor（keymap-editor[bot]）からも更新される
- `config/roBa.json` — keymap-editor / keymap-drawer 用のレイアウト定義
- `config/west.yml` — ZMK は `v0.3-branch`、`zmk-pmw3610-driver` はコミット固定
- `boards/shields/roBa/` — シールド定義。右（roBa_R）が central でトラックボール（PMW3610）付き
- `build.yaml` — ビルド対象（roBa_R は ZMK Studio 用 snippet 付き、roBa_L、settings_reset）
- `keymap-drawer/` — キーマップ画像（自動生成）。Draw Keymap ワークフローを手動実行して更新する

## レイヤー構成

0 BASE（大西配列＋ホームロウモッド） / 1 SYM（左親指 Space 長押し：左手に演算子・計算、右手に括弧・引用符・その他。対の記号は上が開き・下が閉じの縦並び） / 2 NUM（左親指 Tab 長押し：右手に数字（ホーム段 1 2 3）と数字に付く記号・括弧、左手は SYM の左手と同じ） / 3 NAV（右親指 Enter 長押し：右手に矢印・音量・輝度、左手に F キーと修飾キー） / 4 SYS（Space＋Enter の同時押しコンボ：左手に BT・出力切替、内側の列に再起動・ブートローダ） / 5 MOUSE（自動） / 6 SCROLL（全 &trans のモード用）

利き手が右なので、数字・記号・矢印は右手に置く方針。SYS の BT・出力切替・再起動・ブートローダは 1 回押しで発動（SYS 自体が両親指の同時押しでしか入れないため）。

- ホームロウモッドは I/A/O = Alt/Ctrl/Shift、T/N/S = Shift/Ctrl/Alt（`hml` / `hmr`）。NAV の左ホームロウも同じ指＝同じ修飾キー。
- 自動マウスはドライバの automouse ではなく ZMK 標準の `zip_temp_layer`（`boards/shields/roBa/roBa_R.overlay`）。`input-processors = <&zip_temp_layer 5 700>` のレイヤー番号と `excluded-positions` は keymap の MOUSE レイヤーと合わせること。
- IME 切替はホーム段の人差し指＋中指の同時押しコンボ（左 A+O = 英数 LANG2 / 右 T+N = かな LANG1）。左親指外側はワンショット Shift。左手下段の同時押しコンボ Z+X / X+C / C+V = Undo / Copy / Paste。
- スクロールは keymap の `&trackball { scroll-layers = <SCROLL>; }`。

レイヤーを追加・並べ替えする時は、keymap 冒頭の `#define` と roBa_R.overlay のレイヤー番号を合わせて更新すること。

## 前提・方針

- ロータリーエンコーダーは使っていない（関連設定は削除済み）。
- 数字は `N0`〜`N9` を使う（`KP_NUMBER_x` は NumLock に依存するため使わない）。
- **ホスト PC は JIS 配列。** キーマップは US 配列のキーコードで書き、PC 側の USKey2JP で変換している。
  - キーマップを編集する時、必要に応じて「ファームウェア側で JIS 配列に対応する方法」（例: `(` → `&kp LS(N8)`、`@` → `&kp LBKT`、Shift 時の記号が異なるキーは mod-morph）を提案すること。ただし現状は US キーコード + USKey2JP の運用を継続しているので、勝手に書き換えない。

## CI

push / PR ごとに `.github/workflows/build.yml`（ZMK 公式 build-user-config）が走り、ファームウェア（.uf2）を生成する。変更後はこのビルドが通ることを確認する。
