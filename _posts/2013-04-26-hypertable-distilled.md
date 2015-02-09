---
layout: post
title: "Hypertable Distilled"
description: "Hypertable Distilled"
category: Tech
tags: [Hypertable, HBase, BigTable]
---

Hypertable Distilled uploaded as pdf on slideshare : 
[http://www.slideshare.net/edydkim/hypertable-distilled-edydkimgithubcom](http://www.slideshare.net/edydkim/hypertable-distilled-edydkimgithubcom)

Here is as Japanese.

※日本語は下記にてサマリーしてあります。詳しくは添付ドキュメントを参照して下さい。

#### 概要
データベースの選定、特にNoSQLデータベースにおいて、一番重要視されることは拡張性と生産性である。本書では左記2点の観点からログングなどの用途に限らず、リアルアプリケーション開発のCRUDにてどのように使われるかをその仕組み、簡単なアーキテクチャとソースコードから説明する。本書が数多くあるNoSQLデータベースの中、比較や選定に少しでも役に立てればと思う。

**Tip**:DBMS属するデータの種類
* RDBMS & ORDBMS（関係データベース管理システムとオブジェクト関係データベース管理システム）：構造型、又はオブジェクト指向データ
* Document-DBMS（ドキュメントデータベース管理システム）：xmlやjsonのようなテキスト形の準構造型データ
* Graph-DBMS（グラフデータベース）：頂点と節点の関係性を表すデータ
* Column-oriented DBMS（列指向データベース管理システム）：非定型の類似属性データ
* 一般的にターミノロジーとして、Row-ColumnをTurple-Attribute、又はRecord-Elementと呼ぶ。数学やコンピュータサイエンスなど学文ごとに呼び方が異なるが、大抵は同一の意味を持つことが多い。

#### 目次
Contents 参照。

#### 序論
Hypertable（ハイパーテーブル）はMap/Reduceアーキテクチャ基盤のグーグルBigTableクーロンプロジェクトあり、同様のHBaseのように分散ファイルシステムに特化されている。データはカラムベースに既定情報とバージョンやタイムスタンプなどの複数Rowキーに構成される。なお、各Cell（hypertableではKey-ValueペアをCellと呼ぶ）はカラムとカラムqualitifier制限子（タグのようなもの）より複数のインデックスを作成することができる。
インデックス情報はプライマリーテーブル内、同ネームスペースに保持される。（OracleなどRDBMSに例えるとプライマリーテーブルはシステムテーブル、ネームスペースはデータベース)

#### 内容
下記順にシステム構成図を基に、インストール方法、実装方法、ベンチマークについて説明する。
詳細はDesign、Installation、Implementation、Benchmark released (Hypertable vs HBase)] 参照。

#### まとめ
Map/Reduceモデルより分散環境の可用性を向上する試みはGoogleなどに実検証されている。課題は如何に利便性を向上させられるかにあり、HypertableはC++基盤の安定性、多様なデータアクセス方法、かつクエリキャッシュなど独自機能を持つ柔軟性あるデータベース管理システムで実用価値は十分あると考えられる。

#### 参考文献
References 参照

*The article PDF - <http://edydkim.github.com/assets/pdf/kim_daiki-HypertableDistilled.pdf>*