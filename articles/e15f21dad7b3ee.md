---
title: "PlanetScaleでは`prisma migrate`は使わない"
emoji: "🌏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["planetscale", "prisma"]
published: false
---

## TL;DR

## PlanetScaleとPrismaについて

PlanetScaleはサーバレスDBサービスです。利用料金の無料枠が大きい、スケーラビリティが高い、などの理由で注目を集めており、個人開発における「RDBのランニングコストを抑えたい」「けどバズったときもちゃんと対応したい」という不安を解消できそうなサービスだと思っています。

Prismaは、Node.jsとTypeScriptのためのORMです。宣言的なデータモデリングとマイグレーション機能を提供する「Prisma Migrate」、データモデルから自動生成される型安全なデータベースクライアント「Prisma Client」などを提供しています。DBのモデルとアプリケーションのモデルの型が一致することで、安全なコードを書くことができます。

アプリケーション全体をTypeScriptで開発するのであれば、PlanetScaleとPrismaを組み合わせて使うことで、非常に良い開発者体験を得ることができます。

この記事では、PlanetScaleとPrismaを組み合わせて使うときのDBスキーマのマイグレーションの方法について紹介します。

## 前提

PlanetScaleとPrismaは、それぞれ別のDBスキーマのマイグレーションのための仕組みを持っています。

### PlanetScaleの[Branching](https://planetscale.com/docs/concepts/branching)

### Prismaの[Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)

## PrismaでPlanetScaleのDBスキーマを変更するには
