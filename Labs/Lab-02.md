---
lab:
    title: 'MongoDB を Cosmos DB に移行する'
    module: 'モジュール 2: MongoDB ワークロードを Cosmos DB に移行する'
---

# 課題: MongoDB を Cosmos DB に移行する

## 推定時間

90 分

## シナリオ

この課題では、既存の MongoDB データベースを Cosmos DB に移行します。Azure Database Migration Service を使用します。代わりに、MongoDB データベースを使用して Cosmos DB データベースに接続する既存のアプリケーションを再構成する方法についても説明します。

この課題は、一連の IoT デバイスから温度データをキャプチャするシステム例に基づいています。温度は、タイムスタンプと共に MongoDB データベースに記録されます。各デバイスには一意の ID があります。これらのデバイスをシミュレートし、データベースにデータを保存する MongoDB アプリケーションを実行します。また、ユーザーが各デバイスに関する統計情報を照会できるようにする 2 つ目のアプリケーションも使用します。MongoDB から Cosmos DB にデータベースを移行した後、両方のアプリケーションを構成して Cosmos DB に接続し、それらが正しく機能することを確認します。

この課題は、Azure Cloud Shell と Azure portal を使用して実行されます。

## 目的

この課題では、次の内容を学習します。

* 移行プロジェクトの作成
* 移行のソースとターゲットを定義する
* 移行の実行
* 移行の確認

## メモ

この演習の完全な手順は、https://github.com/MicrosoftLearning/DP-060T00A-Migrating-your-Database-to-Cosmos-DB/blob/master/Labs/Lab%202%20-%20Migrate%20MongoDB%20Workloads%20to%20Cosmos%20DB.md で入手できます。

この課題のすべてのファイルは、 https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB にあります。

受講者は、課題を実行する前に、課題ファイルのコピーをダウンロードする必要があります。

## ラボ エクササイズ

* 移行プロジェクトの作成
* ソースとターゲットの定義
* 移行の実行
* 移行の確認

## 課題レビュー

この課題では、既存の MongoDB データベースを Cosmos DB に移行しました。Azure Database Migration Service を使用しました。代わりに、MongoDB データベースを使用して Cosmos DB データベースに接続する既存のアプリケーションを再構成する方法についても確認しました。
