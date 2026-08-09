# nikki

Markdownで日記を書き、MkDocs + Materialで静的サイトを生成してGitHub Pagesへ公開します。

## ローカル環境の準備

Python 3.12を使用します。PowerShellで次を実行してください。

```powershell
C:/Python312/python.exe -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

## ローカルプレビュー

```powershell
.\.venv\Scripts\python.exe -m mkdocs serve
```

表示された `http://127.0.0.1:8000/` をブラウザで開きます。ファイルを保存すると自動的に再ビルドされます。

公開用と同じ条件でビルドする場合は、次を実行します。

```powershell
.\.venv\Scripts\python.exe -m mkdocs build --strict
```

生成物は `site/` に出力されます。このディレクトリはGitの管理対象外です。

## 記事を書く

1. `docs/articles/<年>/` に `YYYY-MM-DD-内容.md` という名前でファイルを作成します。
2. ファイル先頭に次のメタデータを記載します。
3. `docs/index.md`、`docs/articles/index.md`、年ごとの `index.md` から記事へリンクします。
4. ローカルビルドでリンク切れや構文エラーがないことを確認します。

```markdown
---
title: 記事タイトル
date: 2026-08-10
description: 記事の概要
tags:
  - タグ
---

# 記事タイトル

本文
```

下書きは `docs/drafts/` に置いてください。このディレクトリはサイト生成時に除外され、CIでも公開物に含まれないことを確認します。

画像は `docs/assets/images/` に保存し、記事から相対パスで参照します。

```markdown
![画像の説明](../../assets/images/example.png)
```

## GitHub Pagesを有効にする

1. このディレクトリをGitHubリポジトリへpushします。
2. GitHubのリポジトリで `Settings` → `Pages` を開きます。
3. `Build and deployment` の `Source` に `GitHub Actions` を選択します。
4. `main` ブランチへpushすると `.github/workflows/pages.yml` がビルドと公開を行います。

Pull Requestでは `.github/workflows/check.yml` が公開前のビルドを検証します。

公開前に、記事や画像へ個人情報、認証情報、公開してはいけない内容が含まれていないことを確認してください。
