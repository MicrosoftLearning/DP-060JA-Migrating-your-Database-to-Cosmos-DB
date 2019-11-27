---
lab:
    title: 'Cassandra ワークロードを Cosmos DB に移行'
    module: 'モジュール 3: Cassandra ワークロードを Cosmos DB に移行'
---

- [課題 3: Cassandra ワークロードを Cosmos DB に移行](#lab-3-migrate-cassandra-workloads-to-cosmos-db)
  - [エクササイズ 1: セットアップ](#exercise-1-setup)
    - [タスク 1: リソース グループと Virtual Network の作成](#task-1-create-a-resource-group-and-virtual-network)
    - [タスク 2: Cassandra データベース サーバーの作成](#task-2-create-a-cassandra-database-server)
    - [タスク 3: Cassandra データベースの設定](#task-3-populate-the-cassandra-database)
  - [エクササイズ 2: CQLSH COPY コマンドを使用した Cassandra から Cosmos DB へのデータの移行](#exercise-2-migrate-data-from-cassandra-to-cosmos-db-using-the-cqlsh-copy-command)
    - [タスク 1: Cosmos アカウントとデータベースの作成](#task-1-create-a-cosmos-account-and-database)
    - [タスク 2: Cassandra データベースからデータをエクスポートする](#task-2-export-the-data-from-the-cassandra-database)
    - [タスク 3: データを Cosmos DB にインポートする](#task-3-import-the-data-to-cosmos-db)
    - [タスク 4: データ移行が成功したことを確認する](#task-4-verify-that-data-migration-was-successful)
    - [タスク 5: クリーンアップ](#task-5-clean-up)
  - [エクササイズ 3: Spark を使用して Cassandra から Cosmos DB にデータを移行する](#exercise-3-migrate-data-from-cassandra-to-cosmos-db-using-spark)
    - [タスク 1: Spark クラスターの作成](#task-1-create-a-spark-cluster)
    - [タスク 2: データを移行するためのノートブックを作成する](#task-2-create-a-notebook-for-migrating-data)
    - [タスク 3: Cosmos DB に接続してテーブルを作成する](#task-3-connect-to-cosmos-db-and-create-tables)
    - [タスク 4: Cassandra データベースへの接続とデータの取得](#task-4-connect-to-the-cassandra-database-and-retrieve-data)
    - [タスク 5: Cosmos DB テーブルにデータを挿入し、ノートブックを実行する](#task-5-insert-data-into-cosmos-db-tables-and-run-the-notebook)
    - [タスク 6: データ移行が成功したことを確認する](#task-6-verify-that-data-migration-was-successful)
    - [タスク 7: クリーンアップ](#task-7-clean-up)

# 課題 3: Cassandra ワークロードを Cosmos DB に移行

この課題では、2 つのデータセットを Cassandra から Cosmos DB に移行します。データは 2 つの方法で移動します。まず、Cassandra からデータをエクスポートし、**CQLSH COPY** コマンドを使用してデータベースを Cosmos DB にインポートします。次に、Spark を使用してデータを移行します。Cosmos DB データベースに保持されているデータに対してクエリを実行して、移行が成功したことを確認します。 

この課題シナリオは、e コマース システムに関するものです。顧客は商品の注文を行うことができます。顧客と注文の詳細は、Cassandra データベースに記録されます。 

この課題は、Azure Cloud Shell と Azure portal を使用して実行されます。

## エクササイズ 1: セットアップ

最初の演習では、顧客と注文データを保持するための Cassandra データベースを作成します。

### タスク 1: リソース グループと Virtual Network の作成

1. インターネット ブラウザで、https://portal.azure.com に移動してサインインします。
2. Azure portal で **リソース グループ** をクリックし、**+追加** をクリックします。   
3. **リソース グループの作成ページ** で、次の詳細を入力してから、**確認 + 作成** をクリックします。   

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | cassandradbrg |
    | リージョン | 最寄りの場所を選択します |

4. **作成** をクリックし、リソース グループが作成されるまで待ちます。
5. Azure portal の左側のウィンドウで、**+ リソースの作成** をクリックします。
6. **新規** ページの **マーケットプレースの検索** ボックスに、**Virtual Network** と入力し、Enter キーを押します。     
7. **Virtual Network** ページで、**作成** をクリックします。
8. **仮想ネットワークの作成** ページで、次の詳細を入力してから、**作成** をクリックします。   

    | プロパティ  | 値  |
    |---|---|
    | 名前 | databasevnet |
    | アドレス空間 | 10.0.0.0/24 |
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | cassandradbrg |
    | リソース グループ | mongodbrg |
    | リージョン | リソース グループに指定したのと同じ場所を選択する |
    | サブネット名 | デフォルト |
    | サブネットアドレス範囲 | 10.0.0.0/28 |
    | DDoS 保護 | Basic |
    | サービス エンドポイント | 無効 |
    | ファイアウォール | 無効 |

9. 続行する前に、仮想ネットワークが作成されるのを待ちます。

### タスク 2: Cassandra データベース サーバーの作成

1. Azure portal の左側のウィンドウで、**+ リソースの作成** をクリックします。
2. **マーケットプレースの検索** ボックスに、***Bitnami により認定された Cassandra** と入力し、Enter キーを押します。  
3. **Bitnami により認定された Cassandra** ページで、**作成** をクリックします。
4. **仮想マシンの作成** ページで、次の詳細を入力し、**次へ** をクリックします。 **Disks \>**。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | cassandradbrg |
    | 仮想マシン名 | cassandraserver |
    | リージョン | リソース グループに指定したのと同じ場所を選択する |
    | 可用性オプション | インフラストラクチャの冗長性は不要 |
    | イメージ | Bitnami により認定された Cassandra |
    | サイズ： | Standard D2 v2 |
    | 認証タイプ | パスワード |
    | ユーザー名 | azureuser |
    | パスワード | Pa55w.rdPa55w.rd |
    | パスワードを確定する | Pa55w.rdPa55w.rd |

5. **ディスク** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。**Networking \>**。
6. **ネットワーク** ページで、次の詳細を入力してから、**次へ:** をクリックします。 **Management \>**。

    | プロパティ  | 値  |
    |---|---|
    | 仮想ネットワーク | databasevnet |
    | サブネット | 規定値 (10.0.0.0/28) |
    | パブリック IP | (新規) cassandraserver-ip |
    | NIC ネットワーク セキュリティ グループ | 詳細 |
    ! ネットワーク セキュリティ グループの構成 | (新規) cassandraserver-nsg |
    | 高速ネットワーキング | オフ |
    | 負荷分散 | いいえ |

7. **管理** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。**Advanced \>**。
8. **詳細** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。**Tags \>**。
9. **タグ** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。**Review + create \>**。
10. 検証ページで、**作成** をクリックします。
11. 続行する前に、仮想ネットワークがデプロイされるのを待ちます。
12. Azure portal の左側のウィンドウで、**すべてのリソース** をクリックします。
13. **すべてのリソース** ページで、**cassandraserver-nsg** をクリックします。
14. **cassandraserver-nsg** ページの **設定** で、**受信セキュリティ規則** をクリックします。     
15. **cassandraserver-nsg - 受信セキュリティ規則** ページで、**+ 追加** をクリックします。   
15. **受信セキュリティ規則の追加** ウィンドウで、次の詳細を入力してから、**追加** をクリックします。   

    | プロパティ  | 値  |
    |---|---|
    | ソース | 任意 |
    | ソースのポート範囲 | * |
    | 宛先 | 任意 |
    | ターゲットのポート範囲 | 9042 |
    | プロトコル | 任意 |
    | アクション | 許可 |
    | 優先度 | 1020 |
    | 名前 | Cassandra-port |
    | 説明 | クライアントが Cassandra への接続に使用するポート |

### タスク 3: Cassandra データベースの設定

1. Azure portal の左側のウィンドウで、**すべてのリソース** をクリックします。
2. **すべてのリソース** ページで、**cassandraserver-ip** をクリックします。
3. **cassandraserver-ip** ページで、**IP アドレス** を書き留めます。   
4. Azure portal の上部にあるツール バーで、**クラウド シェル**をクリックします。 
5. **記憶域がマウントされていない** メッセージ ボックスが表示されたら、**記憶域の作成** をクリックします。   
6. Cloud Shell が起動したら、Cloud Shell ウィンドウの上にあるドロップダウン リストで **Bash** を選択します。 
7. Cloud Shell で、課題 2 を実行していない場合は、次のコマンドを実行して、このワークショップのサンプル コードとデータをダウンロードします。

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

8. **migration-workshop-apps/Cassandra** フォルダーに移動します。 

    ```bash
    cd ~/migration-workshop-apps/Cassandra
    ```

9. 次のコマンドを入力して、セットアップ スクリプトとデータを **cassandraserver** 仮想マシンにコピーします。 *\<ip address\>* を **cassandraserver-ip** IP アドレスの値に置き換えます。

    ```bash
    scp *.* azureuser@<ip address>:~
    ```

10. プロンプトで、**yes** と入力して接続を続行します。 
11. **パスワード** プロンプトで、パスワード **Pa55w.rdPa55w.rd** を入力します
12. 以下のコマンドを入力し、**cassandraserver** 仮想マシンに接続します。**cassandraserver**仮想マシンの IP アドレスを指定します。

    ```bash
    ssh azureuser@<ip address>
    ```

13. **パスワード** プロンプトで、パスワード **Pa55w.rdPa55w.rd** を入力します
14. 次のコマンドを実行して Cassandra データベースに接続し、この課題に必要なテーブルを作成して設定します。

    ```bash
    bash upload.sh
    ```

    このスクリプトは、**customerinfo** と **orderinfo** という名前の 2 つのキースペースを作成します。 このスクリプトは、**customerinfo** キースペースに **customerdetails** という名前のテーブルを作成し、**受注明細** と **注文明細行** という名前の 2 つのテーブルを **orderinfo**キースペースに作成します。        

15. 次のコマンドを実行し、このファイルの既定のパスワードをメモします。

    ```bash
    cat bitnami_credentials
    ```

16. Cassandra クエリ シェルをユーザー **cassandra** として起動します (これは、仮想マシンのセットアップ時に作成された既定の Cassandra ユーザーの名前です)。  *\<パスワード\>* を前の手順の既定のパスワードに置き換えます。

    ```bash
    cqlsh -u cassandra -p <password>
    ```

17. **cassandra@cqlsh** プロンプトで、次のコマンドを実行します。  このコマンドは、**customerinfo.customerdetails** テーブルの最初の 100 行を表示します。

    ```cqlsh
    select *
    from customerinfo.customerdetails
    limit 100;
    ```

    データは **stateprovince** 列によってクラスター化され、**customerid** で並べ替えられます。 このグループ化により、アプリケーションは同じリージョンにあるすべての顧客をすばやく見つけることができます。

18. 次のコマンドを実行します。このコマンドは、**orderinfo.orderdetails** テーブルの最初の 100 行を表示します。

    ```cqlsh
    select *
    from orderinfo.orderdetails
    limit 100;
    ```

    **orderinfo.orderdetails** テーブルには、各顧客が行った注文の一覧が含まれています。  記録されるデータには、注文が行われた日付と注文の値が含まれます。データは **customerid** 列によってクラスター化されるため、アプリケーションは指定された顧客のすべての注文をすばやく検索できます。 

19. 次のコマンドを実行します。このコマンドは、**orderinfo.orderline** テーブルの最初の 100 行を表示します。

    ```cqlsh
    select *
    from orderinfo.orderline
    limit 100;
    ```

    このテーブルには、各注文の teh 品目が含まれています。データは  **orderid** 列でクラスター化され、 **orderline** で並べ替えられます。   

20. Cassandra クエリ シェルを終了します。

    ```cqlsh
    exit;
    ```

21. **Bitnami@cassandraserver** プロンプトで、次のコマンドを入力して Cassandra サーバーから切断し、Cloud Shell に戻ります。 

    ```bash
    exit
    ```

## エクササイズ 2: CQLSH COPY コマンドを使用した Cassandra から Cosmos DB へのデータの移行 

これで、Cassandra データベースが作成され、設定されました。この練習では、Cassandra API を使用して Cosmos DB アカウントを作成し、Cassandra データベースから Cosmos DB アカウントのデータベースにデータを移行します。

### タスク 1: Cosmos アカウントとデータベースの作成

1. Azure portal に戻ります。
2. 左側のウィンドウで、**+ リソースの作成** をクリックします。
3. **新規** ページの **マーケットプレースの検索** ボックスに　***Azure CosmosDB** と入力してから、Enter キーを押します。    
4. **Azure Cosmos DB** ページで、**作成** をクリックします。
5. **Azure Cosmos DB アカウントの作成** ページで、次の設定を入力してから、**確認 + 作成** をクリックします。   

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | サブスクリプションを選択します |
    | リソース グループ | cassandradbrg |
    | アカウント名 | cassandra*nnn*、 *nnn*はあなたが選択した乱数です。 |
    | API | Cassandra |
    | 場所 | Cassandra サーバーと仮想ネットワークに使用したのと同じ場所を指定します |
    | 地理的冗長性 | 無効にする |
    | 複数リージョンの書き込み | 無効にする |

6. 検証ページで、**作成**をクリックし、Cosmos DB アカウントがデプロイされるまで待ちます。 
7. 左側のウィンドウで、 **Azure Cosmos DB** をクリックします。
8. **Azure Cosmos DB** ページで、Cosmos DB アカウント (**cassandra*nnn***) をクリックします。 
9. **cassandra*nnn*** ページで、**データ エクスプローラー** をクリックします。
10. **データ エクスプローラー**で、**新しいテーブル** をクリックします。
11. **テーブルの追加** ウィンドウで、次の設定を指定してから、**OK** をクリックします。   

    | プロパティ  | 値  |
    |---|---|
    | キースペース名 | **新規作成**をクリックしてから、**customerinfo** と入力します。 |
    | キースペーススループットのプロビジョニング | 選択解除 |
    | テーブル ID の入力 | customerdetails |
    | *テーブルの作成*ボックス | (customerid int、firstname text、lastname text、email text、stateprovince text、PRIMARY KEY ((stateprovince)、customerid)) |
    | スループット | 10000 |

12. **データ エクスプローラー**で、**新しいテーブル** をクリックします。
13. **テーブルの追加** ウィンドウで、次の設定を指定してから、**OK** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | キースペース名 | **新規作成**をクリックし、**orderinfo**と入力します。  |
    | キースペーススループットのプロビジョニング | 選択解除 |
    | テーブル ID の入力 | orderdetails |
    | *テーブルの作成*ボックス | (orderid int、customerid int、orderdate date、ordervalue decimal、PRIMARY KEY ((customerid)、orderdate、orderid)) |
    | スループット | 10000 |

14. **データ エクスプローラー**で、**新しいテーブル** をクリックします。
15. **テーブルの追加** ウィンドウで、次の設定を指定してから、**OK** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | キースペース名 | **既存を使用する** をクリックしてから、**orderinfo** を選択します。  |
    | テーブル ID の入力 | 注文明細行 |
    | *テーブルの作成*ボックス | (orderid int、orderline int、productname text、quantity smallint、orderlinecost decimal、PRIMARY KEY ((orderid)、productname、orderline)) |
    | スループット | 10000 |

### タスク 2: Cassandra データベースからデータをエクスポートする

1. Cloud Shell に戻ります。
2. 以下のコマンドを実行し、cassandra サーバーに接続します。 *\<ip address\>* を仮想マシンの IP アドレスに置き換えます。以下のプロンプトが表示されたら、パスワード **Pa55w.rdPa55w.rd** を入力します。

    ```bash
    ssh azureuser@<ip address>
    ```

3. Cassandra クエリ シェルを起動します。**bitnami_credentials** ファイルからパスワードを指定します。 

    ```bash
    cqlsh -u cassandra -p <password>
    ```

4. **cassandra@cqlsh** プロンプトで、次のコマンドを実行します。  このコマンドは、**customerinfo.customerdetails** テーブルのデータをダウンロードし、 **customerdata**という名前のファイルに書き込みます。  このコマンドは、19119 行をエクスポートする必要があります。

    ```cqlsh
    copy customerinfo.customerdetails
    to 'customerdata';
    ```

5. 次のコマンドを実行して、**orderinfo.orderdetails** テーブルのデータを **orderdata** という名前のファイルにエクスポートします。このコマンドは、31465 行をエクスポートする必要があります。

   ```cqlsh
    copy orderinfo.orderdetails
    to 'orderdata';
    ```

6. 次のコマンドを実行して、**orderinfo.orderline** テーブルのデータを **orderline** という名前のファイルにエクスポートします。このコマンドは、121317 行をエクスポートする必要があります。

   ```cqlsh
    copy orderinfo.orderline
    to 'orderline';
    ```

7. Cassandra クエリ シェルを終了します。

    ```cqlsh
    exit;
    ```

### タスク 3: データを Cosmos DB にインポートする

1. Azure portal で Cosmos DB アカウントに切り替えます。
2. **設定** で、**接続文字列** をクリックし、次の項目を書き留めます。   

   - コンタクト ポイント
   - ポート
   - ユーザー名
   - プライマリ パスワード

3. Cassandra サーバーに戻り、Cassandra クエリ シェルを起動します。今回は、Cosmos DB アカウントに接続します。**cqlsh** コマンドの引数を、先ほど書き留めた値に置き換えます。

    ```bash
    export SSL_VERSION=TLSv1_2
    export SSL_VALIDATE=false

    cqlsh <contact point> <port> -u <username> -p <primary password> --ssl
    ```

    Cosmos DB には SSL 接続が必要であることに注意してください。

4. **cqlsh**プロンプトで、次のコマンドを実行して、Cassandra データベースから以前にエクスポートしたデータをインポートします。 

    ```cqlsh
    copy customerinfo.customerdetails from 'customerdata' with chunksize = 200;
    copy orderinfo.orderdetails from 'orderdata' with chunksize = 200;
    copy orderinfo.orderline from 'orderline' with chunksize = 100;
    ```

    これらのコマンドは、19119 **customerdetails** 行、31465 **orderdetails** 行、および 121317 **orderline** 行をインポートする必要があります。これらのコマンドがタイムアウト エラーで失敗した場合は、次のいずれかの方法で問題を解決できます。
    - Azure portal の対応するテーブルのスループットを増やす (その後、スループット料金が過剰に発生しないように、再度減らす)、または
    - 各コピー操作のチャンク サイズを小さくする。チャンク サイズを小さくするとインジェスト速度が遅くなりますが、スループットを上げるとコストが増加します。

### タスク 4: データ移行が成功したことを確認する

1. Azure portal で Cosmos DB アカウントに戻り、**データ エクスプローラー** をクリックします。 
2. **データ エクスプローラー** ウィンドウで、 **customerinfo** キースペースを展開し、**customerdetails** テーブルを展開して、**行** をクリックします。 一連の顧客が表示されることを確認します。
3. **新しい節の追加** をクリックします。
4. **フィールド** ボックスで、**stateprovince** を選択し、**値** ボックスに **タスマニア** と入力します。
5. ツール バーで、**クエリの実行** をクリックします。クエリが 106 行を返すことを確認します。
6. **データ エクスプローラー** ウィンドウで、 **customerinfo** キースペースを展開し、**customerdetails** テーブルを展開して、**行** をクリックします。
7. **データ エクスプローラー** ウィンドウで、**orderinfo** キースペースを展開し、**orderdetails** テーブルを展開してから、**行** をクリックします。       
8. **新しい節の追加** をクリックします。
9. **フィールド** ボックスで、**customerid** を選択し、**値** ボックスに **13999** と入力します。       
10. ツール バーで、**クエリの実行** をクリックします。クエリが 2 行を返すことを確認します。最初の行の **orderid** に注意してください (46899 である必要があります)。 
11. **データ エクスプローラー** ウィンドウで、**orderinfo** キースペースを展開し、**orderline** テーブルを展開してから、**行** をクリックします。       
12. **データ エクスプローラー** ウィンドウの **orderinfo** キースペースで、**orderline** テーブルを展開してから、**行** をクリックします。       
13. **新しい節の追加** をクリックします。
14. **フィールド** ボックスで、**orderid** を選択し、**値** ボックスに **46899** と入力します。
15. ツール バーで、**クエリの実行** をクリックします。クエリが 1 行を返すことを確認し、**Road-550-W Yellow, 40** として注文されている製品を一覧表示します。

CQLSH COPY コマンドを使用して、Cassandra データベースを Cosmos DB に正常に移行しました。

### タスク 5: クリーンアップ

1. Cassandra サーバーで実行されている Cassandra クエリ シェルに切り替えます。このシェルは引き続き Cosmos DB アカウントに接続されている必要があります。前にシェルを閉じた場合は、次のようにシェルを再度開きます。

    ```bash
    export SSL_VERSION=TLSv1_2
    export SSL_VALIDATE=false

    cqlsh <contact point> <port> -u <username> -p <primary password> --ssl
    ```

2. Cassandra クエリ シェルで、次のコマンドを実行してキースペース (およびテーブル) を削除します。

    ```cqlsh
    drop keyspace customerinfo;
    drop keyspace orderinfo;
    exit;
    ```

## エクササイズ 3: Spark を使用して Cassandra から Cosmos DB にデータを移行する

この演習では、前に使用したのと同じデータを移行しますが、今回は Azure Databricks ノートブックから Spark を使用します。

### タスク 1: Spark クラスターの作成

1. Azure portal の左側のウィンドウで、**+ リソースの作成** をクリックします。 
2. **新規** ウィンドウの **Marketplace の検索** ボックスで、**Azure Databricks** と入力してから、Enter キーを押します。
3. **Azure Databricks** ページで、**作成** をクリックします。
4. **Azure Databricks サービス** ページで、次の詳細を入力してから、**作成** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | ワークスペース名 | CassandraMigration |
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | 既存の cassandradbrg を使用する |
    | 場所 | リソース グループに指定したのと同じ場所を選択する |
    | 価格レベル | 標準 |
    | 仮想ネットワークに Azure Databricks ワークスペースをデプロイする | いいえ |

5. Databricks サービスがデプロイされるのを待ちます。
6. 左側のウィンドウで、**リソース グループ** をクリックし、**cassandradbrg** をクリックして、**CassandraMigration** Databricks サービスをクリックします。   
7. **CassandraMigration** ページで、**ワークスペースの起動** をクリックします。   
8. **Azure Databricks** ページの **共通タスク** で、**新しいクラスター** をクリックします。     
9. **新しいクラスター** ページで、次の設定を入力してから、**クラスターの作成** をクリックします。   

    | プロパティ  | 値  |
    |---|---|
    | クラスター名 | MigrationCluster |
    | クラスター モード | 標準 |
    | Databrick Runtime バージョン | ランタイム: 5.3 (Scala 2.11、Spark 2.4.0) |
    | Python バージョン | 3 |
    | 自動スケーリングを有効にする | 選択済み |
    | 以降に終了 | 60 |
    | worker タイプ | 既定の設定を受け入れる |
    | ドライバー タイプ | worker と同じ |

10. クラスターが作成されるのを待ちます。**移行クラスター** の状態は、クラスターの準備ができたときに **実行中** として報告されます。   この処理には数分かかります。

### タスク 2: データを移行するためのノートブックを作成する

1. **クラスター** ページの左側のウィンドウで、**Azure Databricks** をクリックします。   
2. **Azure Databricks** ページの **共通タスク** で、**ライブラリのインポート** をクリックします。     
3. **ライブラリの作成** ページで、次の設定を入力してから、**作成** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | ライブラリ ソース | Maven |
    | リポジトリ | 空白のままにする |
    | 座標 | com.datastax.spark:spark-cassandra-connector_2.11:2.4.0 |
    | 除外 | 空白のままにする |

    このライブラリには、Spark から Cassandra に接続するためのクラスが含まれています。

4. **実行中のクラスターの状態** セクションが表示されたら、**MigrationCluster** 行の **インストールされていません** の横にあるチェック ボックスをオンにし、**インストール** をクリックします。 
5. 続行する前に、ライブラリの状態が **インストール済み** に変わるまで待ちます。 
6. 左側のウィンドウで、**Azure Databricks** をクリックします。 
7. **Azure Databricks** ページの **共通タスク** で、もう一度 **ライブラリのインポート** をクリックします。   
8. **ライブラリの作成** ページで、次の設定を入力してから、**作成** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | ライブラリ ソース | Maven |
    | リポジトリ | 空白のままにする |
    | 座標 | com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0 |
    | 除外 | 空白のままにする |

    このライブラリには、Spark から Cosmos DB に接続するためのクラスが含まれています。

9. **実行中のクラスターの状態** セクションが表示されたら、**MigrationCluster** 行の **インストールされていません** の横にあるチェック ボックスをオンにし、**インストール** をクリックします。
10. 続行する前に、ライブラリの状態が **インストール済み** に変わるまで待ちます。
11. 左側のウィンドウで、**Azure Databricks** をクリックします。
12. **Azure Databricks** ページの **共通タスク** で、**新しいノートブック** をクリックします。
13. **ノートブックの作成** ダイアログボックスで、次の設定を入力してから、**作成** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | 名前 | MigrateData |
    | 言語 | Scala |
    | クラスター | MigrationCluster |

### タスク 3: Cosmos DB に接続してテーブルを作成する

1. ノートブックの最初のセルに、次のコードを入力します。

    ```scala
    // ライブラリをインポートする

    import org.apache.spark.sql.cassandra._
    import org.apache.spark.sql._
    import org.apache.spark._
    import com.datastax.spark.connector._
    import com.datastax.spark.connector.cql.CassandraConnector
    import com.microsoft.azure.cosmosdb.cassandra
    ```

    このコードは、Spark から Cosmos DB および Cassandra に接続するために必要なタイプをインポートします。

2. セルの右側のツールバーで、ドロップダウン矢印をクリックし、**下にセルを追加** をクリックします。 
3. 新しいセルに、次のコードを入力します。Cosmos DB アカウントの値を使用して、連絡先ポイント、ユーザー名、およびプライマリ パスワードを指定します (前の演習でこれらの値を記録しました)。

    ```scala
    // Cosmos DB の接続パラメータを構成する

    val cosmosDBConf = new SparkConf()
        .set("spark.cassandra.connection.host", "<contact point>")
        .set("spark.cassandra.connection.port", "10350")
        .set("spark.cassandra.connection.ssl.enabled", "true")
        .set("spark.cassandra.auth.username", "<username>")
        .set("spark.cassandra.auth.password", "<primary password>")
        .set("spark.cassandra.connection.factory",
            "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")
        .set("spark.cassandra.output.batch.size.rows", "1")
        .set("spark.cassandra.connection.connections_per_executor_max", "1")
        .set("spark.cassandra.output.concurrent.writes", "1")
        .set("spark.cassandra.concurrent.reads", "1")
        .set("spark.cassandra.output.batch.grouping.buffer.size", "1")
        .set("spark.cassandra.connection.keep_alive_ms", "600000000")
    ```

    このコードは、Cosmos DB アカウントに接続するための Spark セッション パラメーターを設定します。

4. 現在のセルの下に別のセルを追加し、次のコードを入力します。

    ```scala
    // キースペースとテーブルを作成する

    val cosmosDBConnector = CassandraConnector(cosmosDBConf)

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE KEYSPACE customerinfo WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}"))
    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE customerinfo.customerdetails (customerid int, firstname text, lastname text, email text, stateprovince text, PRIMARY KEY ((stateprovince), customerid)) WITH cosmosdb_provisioned_throughput=10000"))

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE KEYSPACE orderinfo WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}"))
    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE orderinfo.orderdetails (orderid int, customerid int, orderdate date, ordervalue decimal, PRIMARY KEY ((customerid), orderdate, orderid)) WITH cosmosdb_provisioned_throughput=10000"))

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE orderinfo.orderline (orderid int, orderline int, productname text, quantity smallint, orderlinecost decimal, PRIMARY KEY ((orderid), productname, orderline)) WITH cosmosdb_provisioned_throughput=10000"))
    ```

    これらのステートメントは、テーブルと共に orderinfo キースペースと customerinfo キースペースを再構築します。各テーブルは、10000 RU/s のスループットでプロビジョニングされます。

### タスク 4: Cassandra データベースへの接続とデータの取得

1. ノートブックに別のセルを追加し、次のコードを入力します。*\<ip address\>* を仮想マシンの IP アドレスに置き換え、**bitnami_credentials** ファイルから以前に取得したパスワードを指定します。   

    ```scala
    // ソース Cassandra データベースの接続パラメーターを構成する

    val cassandraDBConf = new SparkConf()
        .set("spark.cassandra.connection.host", "<ip address>")
        .set("spark.cassandra.connection.port", "9042")
        .set("spark.cassandra.connection.ssl.enabled", "false")
        .set("spark.cassandra.auth.username", "cassandra")
        .set("spark.cassandra.auth.password", "<password>")
        .set("spark.cassandra.connection.connections_per_executor_max", "10")
        .set("spark.cassandra.concurrent.reads", "512")
        .set("spark.cassandra.connection.keep_alive_ms", "600000000")
    ```

2. 別のセルを追加し、次のコードを入力します。

    ```scala
    // ソース データベースから顧客データと注文データを取得する

    val cassandraDBConnector = CassandraConnector(cassandraDBConf)
    var cassandraSparkSession = SparkSession
        .builder()
        .config(cassandraDBConf)
        .getOrCreate()

    val customerDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .load

    println("Read " + customerDataframe.count + " rows")

    val orderDetailsDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .load

    println("Read " + orderDetailsDataframe.count + " rows")

    val orderLineDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .load

    println("Read " + orderLineDataframe.count + " rows")
    ```

    このコード ブロックは、Cassandra データベース内のテーブルから Spark DataFrame オブジェクトにデータを取得します。このコードは、各テーブルから読み取られた行数を表示します。

### タスク 5: Cosmos DB テーブルにデータを挿入し、ノートブックを実行する

1. 最終的なセルを追加し、次のコードを入力します。

    ```scala
    // 顧客データを Cosmos DB に書き込む

    val cosmosDBSparkSession = SparkSession
        .builder()
        .config(cosmosDBConf)
        .getOrCreate()

    // Cosmos DB から既存のテーブルに接続する
    var customerCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .load

    // Cassandra データベースの結果を DataFrame にマージする
    customerCopyDataframe = customerCopyDataframe.union(customerDataframe)

    // 結果を Cosmos DB に書き戻す
    customerCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()

    // 同じ戦略を使用して、Cosmos DB に注文データを書き込む
    var orderDetailsCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .load

    orderDetailsCopyDataframe = orderDetailsCopyDataframe.union(orderDetailsDataframe)

    orderDetailsCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()

    var orderLineCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .load

    orderLineCopyDataframe = orderLineCopyDataframe.union(orderLineDataframe)

    orderLineCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()
    ```

    このコードは、Cosmos DB データベース内のテーブルごとに別の DataFrame を作成します。各 DataFrame は、最初は空になります。次に、**union** 関数を使用して、Cassandra テーブルごとに対応する DataFrame のデータをアペンドします。  最後に、そのコードは、アペンドされた DataFrame を Cosmos DB テーブルに書き戻します。

    DataFrame API は、Spark が提供する非常に強力な抽象化であり、大量のデータを非常に迅速に転送するための非常に効率的な構造です。

2. ノートブックの上部にあるツール バーで、**すべて実行**をクリックします。  クラスターが起動中であることを示すメッセージが表示されます。クラスターの準備ができたら、ノートブックは各セルで順番にコードを実行します。各セルの下にさらにメッセージが表示されます。DataFrame の読み取りと書き込みを行うデータ転送操作は、Spark ジョブとして実行されます。ジョブを展開して進行状況を表示できます。エラー メッセージを表示せずに、各セルのコードが正常に完了する必要があります。

### タスク 6: データ移行が成功したことを確認する

1. Azure portal で Cosmos DB アカウントに戻ります。
2. **データ エクスプローラー** をクリックします。
3. **データ エクスプローラー** ウィンドウで、 **customerinfo** キースペースを展開し、**customerdetails** テーブルを展開して、**行** をクリックします。最初の 100 行が表示されます。キースペースが **データ エクスプローラー** ウィンドウに表示されない場合  は、**最新の情報に更新** をクリックして表示を更新します。   
4. **orderinfo** キースペースを展開し、**orderdetails** テーブルを展開してから、**行** をクリックします。      このテーブルの最初の 100 行も表示されます。
5. 最後に、 **orderline** テーブルを展開し、**行** をクリックします。    このテーブルの最初の 100 行が表示されることを確認します。

データブリック ノートブックの Spark を使用して、Cassandra データベースを Cosmos DB に正常に移行しました。

### タスク 7: クリーンアップ

1. Azure portal の左側のウィンドウで、**リソース グループ** をクリックします。
2. **リソース グループ** ウィンドウで、**cassandradbrg** をクリックします。
3. **リソース グループの削除** をクリックします。
4. **「cassandradbrg」を削除しますか** ページで、**リソース グループ名を入力** ボックスに **cassandradbrg** と入力してから、**削除** をクリックします。

---
© 2019 Microsoft Corporation.All rights reserved.

このドキュメントのテキストは、 クリエイティブ・コモンズ表示 3.0 ライセンスで入手できます。
ライセンス](https://creativecommons.org/licenses/by/3.0/legalcode)、追加条件が適用される場合があります。このドキュメントに含まれるその他すべてのコンテンツ (商標、ロゴ、画像等を含むがこれに限定されない) は、
クリエイティブ・コモンズ・ライセンス付与には**含まれていません**。このドキュメントは、マイクロソフト製品の知的財産権に関する法的権利をお客様に提供するものではありません。内部での参照目的でこのドキュメントをコピーして使用することができます。

このドキュメントは「現状のまま」提供されています。このドキュメントに記載されている情報および見解 (URL およびその他のインターネット Web サイトの参照を含む) は、予告なしに変更されることがあります。その使用にはリスクが伴います。一部の例は説明のみを目的としており、架空のものです。実際の団体とは一切関係ありません。マイクロソフトは、ここに記載されている情報に関して、明示または黙示を問わず、いかなる保証も行いません。
