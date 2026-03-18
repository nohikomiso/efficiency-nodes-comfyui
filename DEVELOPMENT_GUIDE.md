# Efficiency Nodes Fork 開発ガイド

## 1. ワークスペース構造
本プロジェクトは、`efficiency-nodes-comfyui` の「Efficient Loader」に `BREAK` 構文のネイティブサポート（文脈分割機能）を追加するためのフォーク開発環境です。

### ディレクトリ配置
*   **開発用リポジトリ (Fork)**: `/home/ytsubame/src/image_test/efficiency-nodes-fork`
    *   ここでコードの編集、Git管理を行います。
*   **ComfyUI 実稼働環境**: `/home/ytsubame/comfy/ComfyUI/custom_nodes/`
    *   ここにシンボリックリンクを貼ることで、ComfyUI上で動作させます。

---

## 2. 環境セットアップ手順 (Symlink)

以下のコマンドを実行して、開発用フォルダをComfyUIに認識させます。
※既存の `efficiency-nodes-comfyui` がある場合は、事前にリネームまたは削除して競合を防いでください。

```bash
# 既存のノードを無効化（バックアップ）
cd /home/ytsubame/comfy/ComfyUI/custom_nodes
mv efficiency-nodes-comfyui efficiency-nodes-comfyui.bak

# シンボリックリンクの作成
ln -s /home/ytsubame/src/image_test/efficiency-nodes-fork efficiency-nodes-comfyui
```

---

## 3. 改造計画 (Plan)

### 目標
`TSC_EfficientLoader` ノード（およびSDXL版）において、A1111形式の重み付けだけでなく、**`BREAK` キーワードによるClipのチャンク分割・結合** を有効にする。

### ターゲットファイル
*   `efficiency_nodes.py`: メインのロジック。`encode_prompts` 関数がエントリーポイント。
*   `py/bnk_adv_encode.py`: テキストエンコードのコア実装。現在ここには `Advanced CLIP` の古いロジックが含まれていますが、`BREAK` 処理（chunk split）が欠けている可能性があります。

### 実装ステップ
1.  **解析**: `bnk_adv_encode.py` の `advanced_encode` 関数が、入力テキストをどのようにトークナイズしているか確認する。
2.  **機能追加**:
    *   `BREAK` 文字列を検出し、テキストを複数のチャンクに分割するロジックを追加（または有効化）。
    *   分割された各チャンクを個別にエンコードし、`ConditioningConcat` で結合する処理を実装。
3.  **確認**: ComfyUIを起動し、プロンプトに `BREAK` を入れてトークン数が75を超えても正しく分離されるか（Color Bleedingが減るか）テストする。

---

## 4. デバッグ・運用手順

### ログの確認
ComfyUIのコンソールログ（起動したターミナル）にエラーが出力されます。
コード修正後は **ComfyUIの再起動** が必要です（Pythonファイルのリロードのため）。

### テスト用プロンプト例
```text
(best quality, masterpiece), 1girl, silver hair, blue dress
BREAK
red background, fire elements
```
期待動作: `blue dress` の青色が背景の `red` や `fire` と混ざらず、明確に分離されること。
