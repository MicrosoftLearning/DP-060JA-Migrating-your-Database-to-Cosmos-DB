---
lab:
    title: 'Cassandra を Cosmos DB に移行'
    module: 'モジュール 3: Cassandra ワークロードを Cosmos DB に移行'
---

# 課題: Cassandra を Cosmos DB に移行

## 推定時間

90 分

## シナリオ

この課題では、2 つのデータセットを Cassandra から Cosmos DB に移行します。データは 2 つの方法で移動します。まず、Cassandra からデータをエクスポートし、CQLSH COPY コマンドを使用してデータベースを Cosmos DB にインポートします。次に、Spark を使用してデータを移行します。移行が成功したことを確認するには、元の Cassandra データベースに保持されているデータを照会するアプリケーションを実行し、Cosmos DB に接続するようにアプリケーションを再構成します。再構成されたアプリケーションを実行した結果は、同じままである必要があります。

この課題シナリオは、e コマース システムに関するものです。顧客は商品の注文を行うことができます。顧客と注文の詳細は、Cassandra データベースに記録されます。顧客の注文の一覧、特定の製品の注文、注文数などのさまざまな集計など、要約を生成するアプリケーションがあります。

この課題は、Azure Cloud Shell と Azure portal を使用して実行されます。

## 目的

この課題では、次の内容を学習します。

* スキーマのエクスポート
* CQLSH COPY を使用したデータの移動
* Spark を使用したデータの移動
* 移行の確認

## メモ

この課題の詳細な手順については、次で入手できます。https://github.com/MicrosoftLearning/DP-060JA-Migrating-your-Database-to-Cosmos-DB/blob/master/Labs/Lab%203%20-%20Migrate%20Cassandra%20Workloads%20to%20Cosmos%20DB.md

この課題のすべてのファイルは、 https://github.com/MicrosoftLearning/DP-060JA-Migrating-your-Database-to-Cosmos-DB にあります。

受講者は、課題を実行する前に、課題ファイルのコピーをダウンロードする必要があります。

## ラボ エクササイズ

* スキーマのエクスポート
* CQLSH COPY を使用したデータの移動
* Spark を使用したデータの移動
* 移行の確認

## 課題レビュー

この課題では、Cassandra から Cosmos DB に 2 つのデータセットを移行しました。2 つの方法でデータを移動しました。まず、Cassandra からデータをエクスポートし、CQLSH COPY コマンドを使用してデータベースを Cosmos DB にインポートしました。次に、Spark を使用してデータを移行しました。移行が成功したことを確認するには、元の Cassandra データベースに保持されているデータを照会するアプリケーションを実行し、Cosmos DB に接続するようにアプリケーションを再構成します。
