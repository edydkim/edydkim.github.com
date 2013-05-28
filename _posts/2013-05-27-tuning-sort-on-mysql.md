---
layout: post
title: "Tuning Sort on MySQL"
description: "tuning sort on mysql"
category: 
tags: [tuning, sort, mysql]
---

**DBチューニング（改善）事例を現行と他システム（別サービス）の観点から行ったことについて記述。**

### 問題点
**DBアクセス時に全件、2400件のSELECTが5秒以上掛かり、大量扱いされ500msから1000msのsleep待ちをかけないとレスポンスタイムが落ちる謎について・・**

DBに対する総合的な知識が必要、単純にslow queryだけで判断してはならず、DBの論理・物理設計の両面から原因を極め、根本的な解決に挑めなければならない。
上記一テーブルの場合に全件が少量にも関わらず、5秒以上掛かったのはDBアクセス時にクエリを作成できず、whereの条件句が効かない構造（マルチインデックスが存在しない）にも問題があるが、
システムのcoreモジュールから自動生成されるクエリ中のorder by句にも直接起因する。DB上のクエリでorder by（ソート）句というのはデータのFULL SCANを誘発する原因であるため、使い方には慎重を期する必要がある。

なお、単純に一テーブルのデータを無理ありに減らそうとすると、目的を洗い出せば仕様上必要な情報を集めた集合体（エンティティ）であり、一丸に不要とは断言出来ず、仕様に対して不知案内である。

先述した理由により全件、2400件で大量というのは、十分誤解を招く恐れがあるため、このような言葉には選びに注意を要する。
また、システムの作り上、coreモジュールだけに解決が難しい場合、前書きとして現状を深く理解することにより、
単純実行時間で遅い、早いで物事の全てを判断することにより根本的改善の努力を怠慢してはいけない。

しかし、こんなに遅いDBも久しぶりで非常に興味深い。おそらくクエリ実行時に既に実行中の片方のクエリが相互に影響を与えているか、単に物理的にテーブル数などがスペックを超えてる可能性もあると思われる。
これらを0.01秒も惜しいのにわざとsleep待ちを掛けなくても少しDBチューニングするだけで爆速になりそう。

考えられる改善案としては、現状を変えず、踏襲しながらスキーマを一部変更することによりデータアクセスの時間、及び量の両方を根本的に改善することができる。

* クエリモジュールにて不要なソートを取りやめ、セカンダリインデックスをwhere句に指定することにより

* 頻度が高いデータバリュー（データ項目）を全体、単独で抽出することができる。止むを得ずData redundancy（データの重複）は発生するが、これ以上のパフォーマンス劣化を止められる。また各モジュールのどこからロスが発生するか見直す必要がある。

チューニングの詳細は下記にて記述。

### 現行システムチューニング
#### **論理アプローチ**
* 一columnのサイズを小か中盛りにする。

* 不要のorder by句の削除

現状サービスロジックに照らして原則DB内のソート不要、かつ障害の原因特定のために可能な限り、候補要素が排除すべき。

{% capture text %} 
mysql> explain select id, data from imitated_kvs_table order by id asc;
+----+-------------+------------------------------+-------+---------------+---------+---------+------+------+-------+
| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+----+-------------+------------------------------+-------+---------------+---------+---------+------+------+-------+
| 1 | SIMPLE | imitated_kvs_table | index | NULL | PRIMARY | 257 | NULL | 2526 | |
+----+-------------+------------------------------+-------+---------------+---------+---------+------+------+-------+
1 row in set (0.00 sec)

{% endcapture %}
{% include JB/liquid_raw %}

上記の分析結果からkeyの「PRIMARY」 keyが貼り与えられインデックスが効いてるように見えるが
「possible_keys」がNULLのため、実際INDEX_FULL_SCANが行われている、PKはUNIQUE属性のため、TABLE_FULL_SCANとパフォーマンス上相違はなく、結果的にFULL_SCANが発生している。

* セカンダリインデックス（Secondary Index）の付与：条件検索を行い、スキャンの効率を向上させるため、longblob内の部分値に対してインデックスを追加。

現状、2カラム
{% capture text %} 
create table...
id varbinary(255)
data longblob
primary key (id)
{% endcapture %}
{% include JB/liquid_raw %}
改善案、
{% capture text %} 
create table...
id varbinary(255)
data longblob
SECONDARY_INDEX_1 varbinary(255)...
PRIMARY KEY (id),
INDEX (SECONDARY_INDEX_1)
...
{% endcapture %}
{% include JB/liquid_raw %}

* テーブルpartition不可：バイナリタイプPKではLookupのI/Oが急増するため、パーティション無意味。

* PKのソートの用途：PKの条件検索に対してその値は文字コード(UTF8)になるため、adjacent（隣接する）データのパータン検索としては有用。

#### 物理アプローチ
**ソートに関するチェックリスト**

* max_length_for_sort_data値のサイズをあげる：厳密に言うと下げ過ぎでないかを確認、ソート時の二重読み込みを防ぐためにタプルを用いたfilesortアルゴリズムが有効、従ってoptimizerに左記アルゴリズムを適用するためにはmax_length_for_sort_data（システム変数）を上げる。
※注意：上記サイズを上げ過ぎるとDisk I/O↑、CPU activity↓発生

* sort_buffer_sizeを上げる：セッションごとに割当、Max 4GB、Engineに属しない

* read_rnd_buffer_sizeを上げる :セッションごとに割当、Maxは2GB、Engineに属しない

* tmpdirには大容量の物理ディスクを割り当てる。 : Temporarily file領域がFullになってため、rotationがはしてないか確認


※注意：OSのmemori allocation threshold（しきい値）を考慮。
※個人的に検証時倍数を割り当てることにより最適値を見つけやすいかも。


{% capture text %}
もし、本当に、まじで、本物のKVSにしたいなら、下記鉄則を守ること。
　1.テーブルのデータを100～200Mbytes程度の大きさに分割したテーブル（「タブレット」）を管理
　2.1台のテーブル（「タブレットサーバ」）は100個以下のタブレットを保存
　※グーグルの場合、低スペックPCサーバ1台当たり約10～20Gbytes程度

クエリ +---+ recipe（レシピー）：metaサーバ
+
|
+
ID range, tablename, サーバIP, fetch
1~100 , 格納テーブル名, タブレットサーバの場所, 現在の件数（増加分）

タブレットサーバ：実データ
オプション
コピーサーバ：タブレットを複製し、分散する。

※移行する場合、metaサーバにてtable名をそのまま使用可
{% endcapture %}
{% include JB/liquid_raw %}


#### 他システム
**他システムから本体のDBが参照される場合**

実際異なるDBを股がってデータ取得する際に、APIやjsonやメッセージングなどその方法は数多く存在する。しかし元の（下記のOrigin）DBにて遅延が発生する場合、あらゆる方法でもドミノ遅延が発生してしまう。

なるべくI/Oを減らす方法に検討する必要がある。

{% capture text %}
Newbie System（他システム）
|
+----------------------+
Older DB（現行DB） Newbie DB（他DB）


タイプはそのまま引き継ぎながら、

Older
[OLD_SAMPLE Table]
ID varbinary-------------+
DATA　　　　　　　    　 　 |
                         |
Newbie                   |
[SAMPLE Table]           |
SAMPLE_CODE int          |
SAMPLE_NAME varchar(10)  |
FK_ID varbinary----------+
{% endcapture %}
{% include JB/liquid_raw %}

ここで問題はOlderはインデックスがないため、PKで特定できるデータではないとOlderからデータ取得にBottleneckが発生。

恐らくOlderからリアルタイム取得に必要なデータ（エンティティ）は決まっていると予想できるため、コンバート（変換）サーバを通じて、OlderとNewbie DBから取得したデータをマージさせる。

マージ方法は、NewbieのFKカラムを設け、OlderのPKを取得（SELECT）し、保持（INSERT）する。初回以降FKが有効になってるため、DBの全体アクセスパフォーマンスが向上。


### 統合システム
現行のDBと他システムDBをHypertable(NoSQL)を用いて一つに統合する。

Hypertableの概要は以下、
[http://edydkim.github.io/2013/04/26/hypertable-distilled/](http://edydkim.github.io/2013/04/26/hypertable-distilled/)

クエリキャッシュ、マルチインデックスが可能など機能が豊富。


#### ※おまけ、
**テーブルの分散方法**

{% capture text %}
mysql> desc test;
+-------+----------------+------+-----+---------+-------+
| Field | Type | Null | Key | Default | Extra |
+-------+----------------+------+-----+---------+-------+
| id | varbinary(255) | NO | PRI | NULL | |
| data | longblob | NO | | NULL | |
+-------+----------------+------+-----+---------+-------+
2 rows in set (0.17 sec)

mysql> select id, data from test;
+------+------+
| id | data |
+------+------+
| test | test |
+------+------+

mysql> select hex(id) from test;
+----------+
| hex(id) |
+----------+
| 74657374 |
+----------+
1 row in set (0.00 sec)

mysql> select hex(id) from test where hex(id) > hex(0) and hex(id) < hex(8);
+----------+
| hex(id) |
+----------+
| 74657374 |
+----------+
1 row in set (0.00 sec)

mysql> select hex(id) from test where hex(id) < hex(15);
+----------+
| hex(id) |
+----------+
| 31 |
| 74657374 |
+----------+
2 rows in set (0.00 sec)

1~F 15分割が可能=15分割のテーブルを用意

課題はhow to divide to 15 equalliy??
{% endcapture %}
{% include JB/liquid_raw %}

