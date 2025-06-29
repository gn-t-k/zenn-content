# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

Zenn CLIを使用して技術記事や本を管理するためのZennコンテンツリポジトリです。Zenn.devでの公開用にYAMLフロントマターメタデータを含むMarkdownベースの記事を格納しています。

## よく使用するコマンド

### コンテンツ管理
- `npx zenn preview` - ローカルでコンテンツをブラウザプレビュー
- `npx zenn new:article` - 新しい記事を作成
- `npx zenn new:book` - 新しい本を作成
- `npx zenn list:articles` - 記事一覧を表示
- `npx zenn list:books` - 本一覧を表示

### パッケージ管理
- `yarn install` - 依存関係をインストール
- `yarn upgrade` - 依存関係を更新

## コンテンツ構造

### 記事
- `articles/` ディレクトリに配置
- ユニークなIDで命名（例: `244df825a55103.md`）
- 各記事には必須フィールドを含むYAMLフロントマターが必要:
  - `title`: 記事タイトル
  - `emoji`: 記事アイコン用の絵文字（1文字）
  - `type`: "tech"（技術記事）または"idea"（アイデア記事）
  - `topics`: タグの配列
  - `published`: 公開状態のboolean値
  - `publication_name`: オプションの出版名（例: "praha"）

### 本
- `books/` ディレクトリに配置（現在は空）
- 記事と同様の構造に従う

### 画像
- `images/articles/{記事ID}/` ディレクトリに保存
- 記事内で相対パスを使用して参照

## 開発メモ

- このリポジトリはZenn CLI バージョン0.1.158を使用（更新版あり）
- コンテンツはMarkdownで記述し、コードブロックやZenn固有の記法をサポート
- 記事は公開前に下書き状態（published: false）にできる
- 画像は記事IDごとに整理して配置する