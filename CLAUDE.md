# CLAUDE.md

roBa（トラックボール付き分割キーボード）用の ZMK ファームウェア設定リポジトリ。

## 構成

- `config/roBa.keymap` — キーマップ本体。keymap-editor（keymap-editor[bot]）からも更新される
- `config/roBa.json` — keymap-editor / keymap-drawer 用のレイアウト定義
- `config/west.yml` — ZMK は `v0.3-branch`、`zmk-pmw3610-driver` はコミット固定
- `boards/shields/roBa/` — シールド定義。右（roBa_R）が central でトラックボール（PMW3610）付き
- `build.yaml` — ビルド対象（roBa_R は ZMK Studio 用 snippet 付き、roBa_L、settings_reset）
- `keymap-drawer/` — キーマップ画像（自動生成）。Draw Keymap ワークフローを手動実行して更新する

## レイヤー番号

0 O24 / 1 NUM / 2 NAV / 3 FUNCTION / 4 MOUSE / 5 SCROLL / 6 game

トラックボールの `automouse-layer = <4>`、`scroll-layers = <5>` がこの番号に依存しているので、レイヤーを追加・並べ替えする時は合わせて更新すること。

## 前提・方針

- ロータリーエンコーダーは使っていない（関連設定は削除済み）。
- 数字は `N0`〜`N9` を使う（`KP_NUMBER_x` は NumLock に依存するため使わない）。
- **ホスト PC は JIS 配列。** キーマップは US 配列のキーコードで書き、PC 側の USKey2JP で変換している。
  - キーマップを編集する時、必要に応じて「ファームウェア側で JIS 配列に対応する方法」（例: `(` → `&kp LS(N8)`、`@` → `&kp LBKT`、Shift 時の記号が異なるキーは mod-morph）を提案すること。ただし現状は US キーコード + USKey2JP の運用を継続しているので、勝手に書き換えない。

## CI

push / PR ごとに `.github/workflows/build.yml`（ZMK 公式 build-user-config）が走り、ファームウェア（.uf2）を生成する。変更後はこのビルドが通ることを確認する。
