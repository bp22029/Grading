# 採点エージェント

ローカルVLMで手書き課題を一括採点するツール。

## ディレクトリ構成

```
work_grading/
├── grader.py                   # メインスクリプト
├── test_single.py              # 動作確認用（1ファイル試す）
├── assignment_config.json      # 課題ごとの用紙向き設定
├── student_list.csv            # 学生名簿（1列目: ローマ字, 2列目: 漢字氏名）
├── .env                        # API設定（git管理外）
├── .env.example                # .envのテンプレート
├── grading_criteria/           # 採点基準（課題ごとに1ファイル）
│   ├── 事前課題1.md
│   ├── 事前課題1　最終版.md
│   └── （課題が増えたらここに追加）
├── Submitted files/            # 提出物（学生フォルダ群、git管理外）
│   ├── 学生A/
│   │   └── 事前課題1/
│   │       └── バージョン 1/
│   │           └── 提出ファイル.pdf
│   └── 学生B/
│       └── ...
└── grading_summary/            # 全体集計CSV出力先（自動生成、git管理外）
    └── summary_事前課題1_YYYYMMDD_HHMMSS.csv
```

## 使い方

### 1. 環境準備
```bash
pip install openai pillow pdf2image python-dotenv
# Mac: brew install poppler
# Windows: poppler を別途インストール
```

### 2. `.env` を作成
`.env.example` をコピーして `.env` を作成し、環境に合わせて編集する：
```
API_BASE_URL=http://localhost:1234/v1
API_KEY=lm-studio
MODEL_NAME=<使用するモデル名>
SUBMITTED_DIR=./Submitted files   # 省略時はデフォルト値を使用
STUDENT_LIST_PATH=./student_list.csv  # 省略時はデフォルト値を使用
```

### 3. APIサーバー起動
LM Studio、vLLM 等で APIサーバーを立ち上げる。

### 4. 採点する課題を指定
`grader.py` の冒頭を編集：
```python
TARGET_ASSIGNMENT = "事前課題1"   # ← 採点したい課題名
```

### 5. 採点基準を用意
`grading_criteria/事前課題1.md` のように、課題名と同じ名前のmdファイルを作る。
（既存のサンプルを参考に編集してください）

### 6. 実行
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
- **向き補正**: `assignment_config.json` に課題ごとの用紙向き（`landscape` / `portrait`）を設定し、実際の画像サイズと異なる場合に90度回転してから採点

## assignment_config.json について

課題ごとに期待する用紙向きを設定するファイル。`landscape`（横長）か `portrait`（縦長）を指定する。
`TARGET_ASSIGNMENT` に指定した課題名がここに登録されていないとエラーになる。

```json
{
    "事前課題1": "landscape",
    "リフレクションシート1": "portrait"
}
```

## 課題が変わったら
1. `grading_criteria/<新しい課題名>.md` を作成
2. `assignment_config.json` に課題名と用紙向きを追加
3. `grader.py` の `TARGET_ASSIGNMENT` を新しい課題名に変更
4. 実行
