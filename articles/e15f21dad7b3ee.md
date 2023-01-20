---
title: "PlanetScaleでは`prisma migrate`は使わない"
emoji: "🌏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["planetscale", "prisma"]
published: false
---

## TL;DR

## PlanetScaleとPrismaについて

PlanetScaleはサーバレスデータベースサービスです。利用料金の無料枠でできることが充実している、スケーラビリティが高い、などの理由で注目を集めています。個人的には、個人開発における「RDBのランニングコストを抑えたい」「けどバズったときもちゃんと対応したい」という不安を解消できそうなサービスだと思っています。

Prismaは、Node.jsとTypeScriptのためのORMです。宣言的なデータモデリングとマイグレーション機能を提供する「Prisma Migrate」、データモデルから自動生成される型安全なデータベースクライアント「Prisma Client」などを提供しています。データベースのモデルとアプリケーションのモデルの型が一致することで、安全なコードを書くことができます。

アプリケーション全体をTypeScriptで開発するのであれば、PlanetScaleとPrismaを組み合わせて使うことで、非常に良い開発者体験を得ることができます。

この記事では、PlanetScaleとPrismaを組み合わせて使うときのデータベーススキーマのマイグレーションの方法について紹介します。

## 前提

PlanetScaleとPrismaは、それぞれ別のデータベーススキーマのマイグレーションのための仕組みを持っています。

### PlanetScaleの[Branching](https://planetscale.com/docs/concepts/branching)

PlanetScaleでは「Branching」という機能を使うことで、gitのブランチ機能に近い感覚で、「ブランチ」を作ってデータベーススキーマを分岐させることができます。

本番用のデータベーススキーマを変更する必要がある場合、以下の手順で作業を実施します。

1. 本番ブランチから「development branch」を作成します。このブランチ上のデータベースは、本番DBと同様のスキーマを持った別のデータベースなので、自由に変更を加えることができます。
2. development branch上のデータベースに対して、カラムの削除やテーブルの追加などのスキーマの変更を行います。
3. アプリケーションなどからdevelopment branch上のデータベースを操作し、問題なく動作するかを確認します。
4. 「deploy request」を作成します。
5. PlanetScaleは、deploy requestの開発スキーマと本番スキーマとを比較してスキーマ差分を作成します。スキーマ差分を作成することによって、どのような変更が行われるか、デプロイ可能な要求かどうか（ユニークキーが欠落していないか、などのスキーマの問題点がないかどうかなど）を知ることができます。
6. 開発チームはdeploy requestを確認し、デプロイを承認します。
7. PlanetScaleが新しいスキーマのデータベースをデプロイ開始します。
8. このデプロイはダウンタイムがゼロになるような方法で実施されるため、テーブルがロックされたり、移行中に本番環境が遅くなるなどの問題が起こることはありません。

```mermaid
sequenceDiagram
  production->>development: development branchを作成
  development->>development: スキーマの変更
  development->>development: deploy requestの作成
  development->>development: スキーマのコンフリクトを解消
  development->>development: 動作確認
  development->>production: デプロイ
```

詳細は[こちら](https://planetscale.com/docs/concepts/branching)。

### Prismaの[Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)

Prisma Migrateは、データベーススキーマの状態を追跡するために次の情報を使います

- **prismaのスキーマ** … `schema.prisma`のことで、データベーススキーマの構造を定義するsource of truthです。
- **マイグレーション履歴** … `prisma/migrations`フォルダーにあるSQLファイルのことで、データベーススキーマの変更履歴を表します。
- **`prisma_migrations`テーブル** … データベースにある`prisma_migrations`テーブルのことで、データベースに適用されたマイグレーションのメタデータを格納します。
- **データベーススキーマ** … データベースの実際の状態のことです。

本番用データベースのスキーマを変更するときは、以下の手順で作業を実施します。

1. prismaのスキーマを更新します。
2. `prisma migrate dev`もしくは`prisma db push`コマンドを実行します。
3. `prisma migrate dev`コマンドを実行した場合、マイグレーション履歴と`prisma_migrations`テーブルが更新されます。
4. たとえばGitHubでコードのバージョン管理を行っている場合、prismaのスキーマやマイグレーション履歴などの変更のプルリクエストを作成します。
5. GitHub ActionsなどのCIシステムを利用して、スキーマ等の変更をトリガーに、`prisma migrate deploy`コマンドを実行します。
6. `prisma migrate deploy`コマンドにより、prismaのスキーマやマイグレーション履歴から、プレビュー用データベースのデータベーススキーマが更新されます。
7. プルリクエストのマージをトリガーにして、CIシステムで`prisma migrate deploy`コマンドを実行します
8. `prisma migrate deploy`コマンドにより、prismaのスキーマやマイグレーション履歴から、本番用データベースのデータベーススキーマが更新されます。

```mermaid
sequenceDiagram
  
```

<!-- TODO: https://www.prisma.io/docs/concepts/components/prisma-migrate/mental-model#what-is-prisma-migrate -->

## PrismaでPlanetScaleのDBスキーマを変更するには

<!-- TODO: https://www.prisma.io/docs/guides/database/using-prisma-with-planetscale -->

## 参考

<https://planetscale.com/docs/concepts/branching>
<https://www.prisma.io/docs/concepts/components/prisma-migrate/mental-model#what-is-prisma-migrate>
<https://www.prisma.io/docs/guides/database/using-prisma-with-planetscale>
