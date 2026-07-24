# 講師用ガイド (Instructor Guide) — SLM構築ワークショップ

## ノートブック構成と推奨タイムテーブル

| 時間 | ノートブック | 内容 | 備考 |
| :--- | :--- | :--- | :--- |
| 9:00–9:45 | `lab0_intro_and_setup.ipynb` | 概念講義 + Jupyter練習 + 環境チェック | 環境チェックのセルで全員のGPU/ライブラリを確認 |
| 9:45–11:45 | `lab1_morning_bigram.ipynb` | Bigramモデル（穴埋め演習あり） | 穴埋め3箇所。詰まった人は🆘正解コードへ誘導 |
| 11:45–13:00 | 昼休み | | |
| 13:00–14:30 | `lab2` 前半（ステップ0–4） | Transformer構築 + 訓練 + チャット | Transformerのセルは「実行するだけ」と明言して安心させる |
| 14:30–15:30 | `lab2` 後半（ステップ5） | 生成AIで自分のFAQ作成 → `my_faq.txt` → 再訓練 | 一日のハイライト。データ生成に各自のスマホ/PCのChatGPT等を使ってもらう |
| 15:30–16:45 | `lab3_lora_and_mcp.ipynb` | LoRAファインチューニング + 保存 + MCPデモ | MCPは講師デモでも可 |
| 16:45–17:00 | まとめ・宿題案内 | | |

## 事前準備チェックリスト

- [ ] SageMakerインスタンス: **ml.g4dn.xlarge**（T4 GPU）推奨。lab1/lab2はCPUでも可、lab3はGPU推奨
- [ ] カーネル: `conda_pytorch_p310`（ノートブックのメタデータに設定済み）
- [ ] 事前に一度 `%pip install japanize-matplotlib transformers peft accelerate fastmcp` を実行してキャッシュしておくと当日が速い
- [ ] lab3のQwenモデル（約1GB）を事前に一度ダウンロードしておく（HFキャッシュに残る）
- [ ] 受講者がFAQ生成に使う生成AI（Claude / ChatGPT等）へのアクセス手段を確認

## 期待される実行結果（検証済みチェックポイント）

| 箇所 | 期待値 |
| :--- | :--- |
| lab1 Bigram, 3000 iters | Loss 約2.0前後で頭打ち。生成は「それっぽい文字列だが支離滅裂」 |
| lab2 Transformer, 3000 iters | Loss 0.1以下まで急降下（データが反復のため）。「客：おすすめは？」に正しい応答を返す |
| lab2 学習曲線 | 訓練/検証Lossがほぼ重なって下がる（ノートブック内の注意書き通り。データ反復による正常な挙動） |
| lab3 LoRA | trainable% が1%未満と表示される。Before（知らない/作り話）→ After（FAQ通りの回答）の対比がデモの山場 |
| lab3 MCP | list_tools で2ツール表示、call_tool が正しい文字列を返す |

## 既知の注意点・トラブルシューティング

1. **japanize_matplotlib が無い**: lab0に救済セルあり（`%pip install japanize-matplotlib`）。インストール後は該当セルの再実行が必要。
2. **transformers のバージョン**: lab3は `pip install -U` で v5系が入る前提。`dtype=` 引数を使用済み（旧 `torch_dtype` は非推奨）。
3. **my_faq.txt の文字コード**: 受講者がWindowsでコピペしたファイルをアップロードするとShift-JISになることがある。ノートブックは `encoding='utf-8'` 固定なので、JupyterLab内の File → New → Text File で作らせるのが安全（手順に記載済み）。
4. **FAQのQ&A抽出が0件**: 全角コロン（Q：）にも対応する正規表現にしてあるが、「Q1:」のような番号付きは非対応。テンプレート通りの形式を守らせる。
5. **lab3のLoss挙動**: データ件数が少ない（30件程度）ため20エポックでLossが1台まで下がれば十分。過学習気味でも「FAQを覚える」のが目的なので問題ない。
6. **input() セルの中断**: チャットループ中にカーネルが固まったように見えたら、「終了」入力かカーネルのInterruptを案内。
7. **fastmcp のバージョン**: v3系で検証済み（`@mcp.tool` 裸デコレータ、`Client(mcp)` インメモリ接続、`result.content[0].text`）。

## このリフレッシュ版での変更点（旧版との差分）

- SageMaker Studio Lab依存を除去（新規受付終了のため）。汎用SageMaker/JupyterLab前提に書き換え
- テキストのエンコードを `get_batch` 内の毎回実行から、事前の一括テンソル化に修正（PDF Lab 1と整合）
- 訓練/検証分割 + `estimate_loss` 評価ループを追加（PDF Lab 4の過学習の教えをコードに反映。反復データゆえの限界も正直に注記）
- Matplotlibによる学習曲線の可視化を追加（ライブラリ学習の目標にも寄与）
- チャレンジラボを lab2 本編に昇格（自分のFAQで訓練）
- lab3を新規作成: LoRAファインチューニング（Qwen2.5-0.5B-Instruct）+ 持ち帰りガイド + MCPデモ
