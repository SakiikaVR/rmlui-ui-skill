# RmlUi UI Skill

RmlUiを使ったC++デスクトップアプリケーションのUI設計・実装・デバッグを支援するCodexスキルです。特に、DAWのタイムライン、ピアノロール、ミキサー、VSTホスト画面のような高密度UIを対象にしています。

## 主な用途

- RML／RCSSの設計、実装、リファクタリング
- Flex、スクロール、クリッピング、絶対配置の崩れの調査
- DPIスケーリングとフォントレンダリングの改善
- タイムライン、ノート、クリップ、フェーダーのマウス操作
- マウスキャプチャ、ドラッグ終了、カーソル復帰などの入力不具合の修正
- HTML比較環境と実際のRmlUiランタイムを使った画面検証
- インラインSVGアイコンとオン／オフ状態色を含むRmlUiコントロールの設計
- VSTエディター表示中も音声処理を止めないUI／音声スレッド分離とブロック欠落検証
- KituneTone固有のDAW／VST UI規約に沿った実装

## 収録内容

- [`SKILL.md`](SKILL.md): Codexが最初に読むスキル本体
- [`references/layout-and-rendering.md`](references/layout-and-rendering.md): レイアウト、スクロール、DPI、フォント、入力処理の設計指針
- [`references/verification.md`](references/verification.md): スクリーンショット監査と操作テストの手順
- [`references/kitunetone.md`](references/kitunetone.md): KituneTone固有のUI構造と受け入れ条件
- [`agents/openai.yaml`](agents/openai.yaml): Codex上の表示情報と既定プロンプト

## 導入

このリポジトリを、Codexのスキルディレクトリ内でフォルダー名が`rmlui-ui`になるように配置します。

Windowsの例：

```powershell
git clone https://github.com/SakiikaVR/rmlui-ui-skill.git "$env:USERPROFILE\.codex\skills\rmlui-ui"
```

すでに導入済みの場合は、対象ディレクトリで`git pull`して更新できます。プライベートリポジトリとして利用する場合は、先にGitHub CLIまたはGitの認証を完了してください。

## 呼び出し例

Codexは対象作業に応じてこのスキルを自動選択できます。明示的に指定する場合は、依頼文に`$rmlui-ui`を含めます。

```text
$rmlui-ui を使って、ピアノロールのノートリサイズとカーソル復帰を実装し、実アプリで検証して
```

```text
$rmlui-ui を使って、1280x720で崩れるミキサー画面を調査して直して
```

## 設計方針

- RMLは意味構造、RCSSは静的な見た目、C++は状態と操作を担当します。
- 再生位置、メーター、ドラッグ中の要素は、毎フレーム大きなDOMを再構築せず安定した要素を更新します。
- RCSSの指定だけで判断せず、RmlUiバックエンド、DPI、入力座標、ネイティブウィンドウまで確認します。
- ビルド成功だけで完了とせず、実行ファイルの操作とスクリーンショットで確認します。
- KituneTone本体ではRmlUiを維持し、JUCEを本番依存として導入しません。
- アイコン専用ボタンは明示的な正方形ヒット領域とSVGサイズを持たせ、`image-color`で状態色を切り替えます。
- VST GUIのイベント処理、RmlUi更新、描画、画面提示は音声要求スレッドから分離し、実プラグイン操作中の処理ブロック欠落を計測します。

## 検証

スキルの構造はCodex付属の`quick_validate.py`で検査できます。

```powershell
python path\to\skill-creator\scripts\quick_validate.py .
```

UI実装の検証方法は[`references/verification.md`](references/verification.md)を参照してください。KituneToneでは、HTMLとRmlUiの比較用デザインラボに加えて、本番実行ファイルでのドラッグ、解放、追加マウス移動まで確認します。

## メンテナンス

新しい知見は、特定の不具合だけに依存しない再利用可能な判断基準として追加します。人向け説明はこのREADME、Codexが実行時に従う指示は`SKILL.md`と`references/`を正本とします。
