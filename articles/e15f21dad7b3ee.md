---
title: "PlanetScaleでは`prisma migrate`は使わない"
emoji: "🌏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["planetscale", "prisma"]
published: true
publication_name: "praha"
---

個人開発でPlanetScaleとPrismaを使用していて、これらを組み合わせて使う上で調べたことをまとめました。

## TL;DR

- PlanetScaleとPrismaを併用する場合、スキーマ変更はPlanetScaleに任せる
- `prisma migrate`は使わず、`prisma db push`を使う

## PlanetScaleとPrismaについて

PlanetScaleはサーバレスデータベースサービスです。利用料金の無料枠でできることが充実している、スケーラビリティが高い、などの理由で注目を集めています。個人的には、個人開発における「RDBのランニングコストを抑えたい」「けどバズったときもちゃんと対応したい」という不安を解消できそうなサービスだと思っています。

Prismaは、Node.jsとTypeScriptのためのORMです。宣言的なデータモデリングとマイグレーション機能を提供する「Prisma Migrate」、データモデルから自動生成される型安全なデータベースクライアント「Prisma Client」などを提供しています。データベースのモデルとアプリケーションのモデルの型が一致することで、安全なコードを書くことができます。

アプリケーション全体をTypeScriptで開発するのであれば、PlanetScaleとPrismaを組み合わせて使うことで、非常に良い開発者体験を得ることができます。

この記事では、PlanetScaleとPrismaを組み合わせて使うときのデータベーススキーマのマイグレーションの方法について紹介します。

## それぞれのスキーママイグレーションの仕組み

PlanetScaleとPrismaは、それぞれ別のデータベーススキーマのマイグレーションのための仕組みを持っています。

### PlanetScaleのBranching

PlanetScaleでは[Branching](https://planetscale.com/docs/concepts/branching)という機能を使うことで、gitのブランチ機能に近い感覚で、「ブランチ」を作ってデータベーススキーマを分岐させることができます。

本番用のデータベーススキーマを変更する必要がある場合、以下の手順で作業を実施します。

1. 本番ブランチから「development branch」を作成します。このブランチ上のデータベースは、本番DBと同様のスキーマを持った別のデータベースなので、自由に変更を加えることができます。
2. development branch上のデータベースに対して、カラムの削除やテーブルの追加などのスキーマの変更を行います。
3. アプリケーションなどからdevelopment branch上のデータベースを操作し、問題なく動作するかを確認します。
4. 「deploy request」を作成します。
5. PlanetScaleは、deploy requestの開発スキーマと本番スキーマとを比較してスキーマ差分を作成します。スキーマ差分を作成することによって、どのような変更が行われるか、デプロイ可能な要求かどうか（ユニークキーが欠落していないか、などのスキーマの問題点がないかどうかなど）を知ることができます。
6. 開発チームはdeploy requestを確認し、デプロイを承認します。
7. PlanetScaleが新しいスキーマのデータベースをデプロイ開始します。
8. このデプロイはダウンタイムがゼロになるような方法で実施されるため、テーブルがロックされたり、移行中に本番環境が遅くなるなどの問題が起こることはありません。

![PlanetScaleのBlanchingのイメージ図](https://planetscale-images.imgix.net/docs/concepts/branching/diagram.png?auto=compress%2Cformat)

詳細は[こちら](https://planetscale.com/docs/concepts/branching)。

### PrismaのPrisma Migrate

[Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)は、データベーススキーマの状態を追跡するために次の情報を使います

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

![PrismaのPrisma Migrateのイメージ図](https://www.prisma.io/docs/static/19181e590d4e9235f167f8ab7422a6c8/663f3/prisma-migrate-lifecycle.png)

詳細は[こちら](https://www.prisma.io/docs/concepts/components/prisma-migrate/mental-model#what-is-prisma-migrate)

## PrismaでPlanetScaleのDBスキーマを変更するには

PlanetScaleは開発ブランチを本番ブランチにマージするとき自動的に独自のスキーマ差分を生成し、独自のマイグレーション履歴管理を行います。PrismaはSQLファイルや`prisma_migrations`テーブルを使って独自のマイグレーション履歴管理を行います。

Prismaが管理するSQLファイルや`prisma_migrations`テーブルには、PlanetScaleが生成するスキーマ差分の情報は反映されません。また、PlanetScaleが生成するスキーマ差分にもPrismaが管理するSQLファイルや`prisma_migrations`テーブルの情報は含まれません。つまり、PlanetScaleもPrismaもそれぞれ独自のマイグレーション履歴管理方法を持っているため、それらを両立することはできません。

Prismaは、**PlanetScaleでスキーマの変更を行う場合は`prisma migrate`を使用せず、`prisma db push`コマンドを使用する**ことを[推奨しています](https://www.prisma.io/docs/guides/database/using-prisma-with-planetscale#:~:text=Prisma%20recommends%20not%20using%20prisma%20migrate%20when%20making%20schema%20changes%20with%20PlanetScale.%20Instead%2C%20we%20recommend%20that%20you%20use%20the%20prisma%20db%20push%20command.)。

PlanetScaleも、**ダウンタイムにつながるスキーマ変更を防止しつつ変更を適用する責任はPlanetScale側にあり、PlanetScaleといっしょに`prisma migrate`を使用する価値はほとんどない**とし、`prisma db push`コマンドを使用することを[推奨しています](https://planetscale.com/docs/tutorials/automatic-prisma-migrations#:~:text=We%20recommend%20prisma%20db%20push%20over%20prisma%20migrate%20dev%20for%20the%20following%20reasons%3A)。

### 手順

PlanetScaleで`prisma db push`を使用するためには、Prisma Clientで参照エミュレーションを有効にする必要があります。PlanetScaleではデータベーススキーマで外部キーが利用できず、それが関係しているようです。

> To use db push with PlanetScale, you will first need to enable emulation of relations in Prisma Client. Pushing to your branch without referential emulation enabled will give the error message Foreign keys cannot be created on this database.

<https://www.prisma.io/docs/guides/database/using-prisma-with-planetscale#how-to-make-schema-changes-with-db-push>

<!-- TODO: https://www.prisma.io/docs/guides/database/using-prisma-with-planetscale -->

```plaintext
datasource db {
  provider     = "mysql"
  url          = env("DATABASE_URL")
  relationMode = "prisma"
}
```

`schema.prisma`の`relationMode`フィールドを`prisma`に設定することで参照エミュレーションを有効化できます。

あとは、`schema.prisma`に定義してあるデータベーススキーマを変更し、`prisma db push`、PlanetScaleでデプロイリクエストを作成してマージするだけです。もっと詳しい手順が知りたい方は、下記の公式ドキュメントを参照してください。

## 参考

<https://planetscale.com/docs/concepts/branching>
<https://www.prisma.io/docs/concepts/components/prisma-migrate/mental-model#what-is-prisma-migrate>
<https://www.prisma.io/docs/guides/database/using-prisma-with-planetscale>
<https://planetscale.com/docs/tutorials/automatic-prisma-migrations>
