# 採点エージェント

ローカルVLMで手書き課題を一括採点するツール。

## ディレクトリ構成

```
grader/
├── grader.py                   # メインスクリプト
├── test_single.py              # 動作確認用（1ファイル試す）
├── grading_criteria/           # 採点基準（課題ごとに1ファイル）
│   ├── 事前課題1.md
│   ├── 事前課題1　最終版.md
│   └── （課題が増えたらここに追加）
├── Submitted files/            # 提出物（学生フォルダ群）
│   ├── 学生A/
│   │   └── 事前課題1/
│   │       └── バージョン 1/
│   │           └── 提出ファイル.pdf
│   └── 学生B/
│       └── ...
└── grading_summary/            # 全体集計CSV出力先（自動生成）
    └── summary_事前課題1_YYYYMMDD_HHMMSS.csv
```

## 使い方

### 1. 環境準備
```bash
pip install openai pillow pdf2image
# Mac: brew install poppler
# Windows: poppler を別途インストール
```

### 2. VLMサーバー起動
LM Studio または mlx-vlm で `localhost:1234` にAPIサーバーを立ち上げる。

### 3. 採点する課題を指定
`grader.py` の冒頭を編集：
```python
TARGET_ASSIGNMENT = "事前課題1"   # ← 採点したい課題名
```

### 4. 採点基準を用意
`grading_criteria/事前課題1.md` のように、課題名と同じ名前のmdファイルを作る。
（既存のサンプルを参考に編集してください）

### 5. 実行
```bash
python grader.py
```

## 出力

### 各学生の課題フォルダ内
- `grading_result.json` — 機械可読・再処理用
- `grading_result.md`   — 人間可読・確認用

### 全体集計
- `grading_summary/summary_<課題名>_<日時>.csv`
- 全学生の状態（採点完了 / 採点済みスキップ / 未提出 / エラー）と点数を一覧化

## 仕様

- **最新バージョン優先**: `バージョン N` フォルダの数字が大きい方を採点対象にする
- **採点済みスキップ**: `grading_result.json` がある学生は再採点しない
- **未提出検出**: 課題フォルダ自体が無い、または中身が空の学生は「未提出」として記録
- **PDF / 画像対応**: `.pdf` を優先、なければ `.jpg .jpeg .png .webp .bmp .tiff` を採点
- **縦長補正**: スマホ縦撮影など縦長画像は90度回転してから採点

## 課題が変わったら
1. `grading_criteria/<新しい課題名>.md` を作成
2. `grader.py` の `TARGET_ASSIGNMENT` を新しい課題名に変更
3. 実行
