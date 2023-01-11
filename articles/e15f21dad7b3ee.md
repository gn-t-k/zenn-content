---
title: "PlanetScaleでは`prisma migrate`は使わない"
emoji: "🌏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["planetscale", "prisma"]
published: false
---

## TL;DR

## PlanetScaleとPrismaについて

PlanetScaleはサーバレスDBサービスです。利用料金の無料枠でできることが充実している、スケーラビリティが高い、などの理由で注目を集めています。個人的には、個人開発における「RDBのランニングコストを抑えたい」「けどバズったときもちゃんと対応したい」という不安を解消できそうなサービスだと思っています。

Prismaは、Node.jsとTypeScriptのためのORMです。宣言的なデータモデリングとマイグレーション機能を提供する「Prisma Migrate」、データモデルから自動生成される型安全なデータベースクライアント「Prisma Client」などを提供しています。DBのモデルとアプリケーションのモデルの型が一致することで、安全なコードを書くことができます。

アプリケーション全体をTypeScriptで開発するのであれば、PlanetScaleとPrismaを組み合わせて使うことで、非常に良い開発者体験を得ることができます。

この記事では、PlanetScaleとPrismaを組み合わせて使うときのDBスキーマのマイグレーションの方法について紹介します。

## 前提

PlanetScaleとPrismaは、それぞれ別のDBスキーマのマイグレーションのための仕組みを持っています。

### PlanetScaleの[Branching](https://planetscale.com/docs/concepts/branching)

PlanetScaleでは「Branching」という機能を使うことで、gitのブランチ機能に近い感覚で、「ブランチ」を作ってDBスキーマを分岐させることができます。

本番用のDBスキーマを変更する必要がある場合、以下の手順で作業を実施します。

1. 本番ブランチから「development branch」を作成します。このブランチ上のDBは、本番DBと同様のスキーマを持った別のDBなので、自由に変更を加えることができます。
2. development branch上のDBに対して、カラムの削除やテーブルの追加などのスキーマの変更を行います。
3. アプリケーションなどからdevelopment branch上のDBを操作し、問題なく動作するかを確認します。
4. 「deploy request」を作成します。
5. 
6. 
7. 
8. 

### Prismaの[Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)

## PrismaでPlanetScaleのDBスキーマを変更するには

## 参考

<https://planetscale.com/docs/concepts/branching>
