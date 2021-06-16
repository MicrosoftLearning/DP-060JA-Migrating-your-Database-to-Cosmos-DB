- [ラボ 3: Cassandra ワークロードを Cosmos DB に移行する](#lab-3-migrate-cassandra-workloads-to-cosmos-db)
  - [演習 1: 設定](#exercise-1-setup)
    - [タスク 1: リソース グループと Virtual Network を作成する](#task-1-create-a-resource-group-and-virtual-network)
    - [タスク 2: Cassandra データベース サーバーを作成する](#task-2-create-a-cassandra-database-server)
    - [タスク 3: Cassandra データベースにデータを読み込む](#task-3-populate-the-cassandra-database)
  - [演習 2: CQLSH COPY コマンドを使用して Cassandra から Cosmos DB にデータを移行する](#exercise-2-migrate-data-from-cassandra-to-cosmos-db-using-the-cqlsh-copy-command)
    - [タスク 1: Cosmos アカウントとデータベースを作成する](#task-1-create-a-cosmos-account-and-database)
    - [タスク 2: Cassandra データベースからデータをエクスポートする](#task-2-export-the-data-from-the-cassandra-database)
    - [タスク 3: Cosmos DB にデータをインポートする](#task-3-import-the-data-to-cosmos-db)
    - [タスク 4: 正常にデータが移行されたことを検証する](#task-4-verify-that-data-migration-was-successful)
    - [タスク 5: クリーンアップする](#task-5-clean-up)
  - [演習 3: Spark を使用して Cassandra から Cosmos DB にデータを移行する](#exercise-3-migrate-data-from-cassandra-to-cosmos-db-using-spark)
    - [タスク 1: Spark クラスターの作成](#task-1-create-a-spark-cluster)
    - [タスク 2: データを移行するためのノートブックを作成する](#task-2-create-a-notebook-for-migrating-data)
    - [タスク 3: Cosmos DB に接続してテーブルを作成する](#task-3-connect-to-cosmos-db-and-create-tables)
    - [タスク 4: Cassandra データベースに接続してデータを取得する](#task-4-connect-to-the-cassandra-database-and-retrieve-data)
    - [タスク 5: Cosmos DB テーブルにデータを挿入し、ノートブックを実行する](#task-5-insert-data-into-cosmos-db-tables-and-run-the-notebook)
    - [タスク 6: 正常にデータが移行されたことを検証する](#task-6-verify-that-data-migration-was-successful)
    - [タスク 7: クリーンアップする](#task-7-clean-up)

# ラボ 3: Cassandra ワークロードを Cosmos DB に移行する

このラボでは、2 つのデータセットを Cassandra から Cosmos DB に移行します。データは 2 つの方法で移行します。まず、Cassandra からデータをエクスポートし、**CQLSH COPY** コマンドを使用してデータベースを Cosmos DB にインポートします。次に、Spark を使用してデータを移行します。Cosmos DB データベースに保持されているデータに対してクエリを実行することで、移行が成功したかどうかを確認します。 

このラボ シナリオは、e コマース システムに関するものです。顧客は商品の注文を行うことができます。顧客と注文の詳細は、Cassandra データベースに記録されます。 

このラボは、Azure Cloud Shell と Azure portal を使用して実行されます。

## 演習 1: 設定

最初の演習では、顧客と注文データを保持するための Cassandra データベースを作成します。

### タスク 1: リソース グループと Virtual Network を作成する

1. インターネット ブラウザで、https://portal.azure.com に移動してサインインします。
1. Azure portal で「**リソース グループ**」を選択し、「**+ 追加**」を選択します。
1. 「**リソース グループを作成する**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | cassandradbrg |
    | リージョン | 最も近い場所を選択します |

1. 「**Review + create**」を選択します。
1. 「**作成**」を選択し、リソース グループが作成されるまで待ちます。
1. Azure portal の左ペインにある「**+ リソースの作成**」を選択します。
1. 「**新規**」ページの「**Marketplace を検索**」ボックスに、「**Virtual Network**」と入力し、Enter キーを押します。
1. 「**仮想ネットワーク**」ページで、「**作成**」を選択します。
1. 「**基本**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | cassandradbrg |
    | 名前 | databasevnet |
    | リージョン | リソース グループに指定したのと同じ場所を選択します |

1. 「**次へ :IP アドレス**」を選択します。
1. 「**IP アドレス**」ページで、「**IPv4 アドレス空間**」を「**10.0.0.0/24**」に設定します。
1. 既定のサブネットを選択し、「**サブネットの削除**」を選択します。
1. 「**+ サブネットの追加**」を選択します。「**サブネットの追加**」ペインで、「**サブネット名**」を「**default**」に設定し、「**サブネット アドレス範囲**」を「**10.0.0.0/28**」に設定して、「**追加**」を選択します。
1. 「**IP アドレス**」ページで、「**次へ: セキュリティ**」を選択します。
1. 「**セキュリティ**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | BastionHost | Disabled |
    | DDoS Protection Standard  | Disabled |
    | ファイアウォール | Disabled |

1. 「**Review + create**」を選択する
1. 「**Review + create**」ページで「**作成**」を選択し、仮想ネットワークが作成されるまで待ってから続けます。

### タスク 2: Cassandra データベース サーバーを作成する

1. Azure portal の左ペインにある「**+ リソースの作成**」を選択します。
1. 「**Marketplace の検索**」ボックスに「**Cassandra Certified by Bitnami**」と入力し、Enter キーを押します。
1. 「**Bitnami 認定済み Cassandra**」ページで、「**作成**」を選択します。
1. 「**仮想マシンの作成**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | cassandradbrg |
    | 仮想マシン名 | cassandraserver |
    | リージョン | リソース グループに指定したのと同じ場所を選択します |
    | 可用性オプション | インフラストラクチャの冗長性は必要ありません |
    | 画像 | Bitnami 認定済み Cassandra - Gen1 |
    | サイズ | Standard D2 v2 |
    | 認証の種類 | パスワード |
    | ユーザー名 | azureuser |
    | パスワード | Pa55w.rdPa55w.rd |
    | パスワードの確認 | Pa55w.rdPa55w.rd |

1. 「**次へ: Disks \>**」を選択します。
1. 「**ディスク**」ページで、既定の設定をそのままにし、「**次へ: ネットワーク \>**」を選択します。
1. 「**ネットワーク**」ページで、以下の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | バーチャル ネットワーク | databasevnet |
    | サブネット | 既定値 (10.0.0.0/28) |
    | パブリック IP | (新規) cassandraserver-ip |
    | NIC ネットワーク セキュリティ グループ | 詳細 |
    | ネットワーク セキュリティ グループを構成する | (新規) cassandraserver-nsg |
    | 高速ネットワーク | オフ |
    | 負荷分散 | No |

1. 「**次へ: 管理 \>**」を選択します。
1. 「**管理**」ページで、既定の設定をそのままにし、「**次へ: 詳細 \>**」を選択します。
1. 「**詳細**」ページで、既定の設定をそのままにし、「**次へ: タグ >**」を選択します。
1. 「**タグ**」ページで、既定の設定をそのままにし、「**次へ: Review + create \>**」を選択します。
1. 検証ページで、「**作成**」を選択します。
1. 仮想マシンがデプロイされるまで待ってから続けます。
1. Azure portal の左ペインにある「**すべてのリソース**」を選択します。
1. 「**すべてのリソース**」ページで、「**cassandraserver-nsg**」を選択します。
1. 「**cassandraserver-nsg**」のページの「**設定**」で、「**受信セキュリティ規則**」を選択します。
1. 「**cassandraserver-nsg - 受信セキュリティ規則**」ページで、「**+ 追加**」を選択します。
1. 「**受信セキュリティ規則の追加**」ペインで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | ソース | Any |
    | 送信元ポートの範囲 | * |
    | 宛先 | Any |
    | 宛先ポート範囲 | 9042 |
    | プロトコル | Any |
    | アクション | 許可 |
    | 優先度 | 1020 |
    | 名前 | Cassandra-port |
    | 説明 | クライアントが Cassandra への接続に使用するポート |

1. 「**追加**」を選択します。

### タスク 3: Cassandra データベースにデータを読み込む

1. Azure portal の左ペインにある「**すべてのリソース**」を選択します。
1. 「**すべてのリソース**」ページで、「**cassandraserver-ip**」を選択します。
1. 「**cassandraserver-ip**」ページで、「**IP アドレス**」を記録しておきます。
1. Azure portal の上部にあるツール バーで、「**Cloud Shell**」を選択します。
1. 「**ストレージがマウントされていません**」というメッセージ ボックスが表示されたら、「**ストレージの作成**」を選択します。
1. Cloud Shell が起動したら、Cloud Shell ウィンドウの上にあるドロップダウン リストで **Bash** を選択します。
1. Cloud Shell で、課題 2 を実行していない場合は、次のコマンドを実行して、このワークショップのサンプル コードとデータをダウンロードします。

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

1. 「**migration-workshop-apps/Cassandra**」フォルダーに移動します。

    ```bash
    cd ~/migration-workshop-apps/Cassandra
    ```

1. 次のコマンドを入力して、セットアップ スクリプトとデータを「**cassandraserver**」仮想マシンにコピーします。*\<ip アドレス\>* を **cassandraserver-ip** IP アドレスの値に置き換えます。

    ```bash
    scp *.* azureuser@<ip address>:~
    ```

1. プロンプトで、「**yes**」と入力して接続を続行します。
1. 「**Password**」プロンプトで、パスワード「**Pa55w.rdPa55w.rd**」を入力します
1. 次のコマンドを入力して、「**cassandraserver**」仮想マシンに接続します。「**cassandraserver**」仮想マシンの IP アドレスを指定します。

    ```bash
    ssh azureuser@<ip address>
    ```

1. 「**Password**」プロンプトで、パスワード「**Pa55w.rdPa55w.rd**」を入力します
1. 次のコマンドを実行して Cassandra データベースに接続し、このラボに必要なテーブルを作成して、データを読み込みます。

    ```bash
    bash upload.sh
    ```

    このスクリプトにより、「**customerinfo**」および「**orderinfo**」という名前の 2 つのキースペースが作成されます。このスクリプトは、「**customerinfo**」キースペースに「**customerdetails**」という名前のテーブルを作成し、「**受注明細**」と「**注文明細行**」という名前の 2 つのテーブルを「**orderinfo**」キースペースに作成します。

1. 次のコマンドを実行し、このファイルの既定のパスワードを記録しておきます。

    ```bash
    cat bitnami_credentials
    ```

1. ユーザー「**cassandra**」 (これは、仮想マシンのセットアップ時に作成された既定の Cassandra ユーザーの名前です) として Cassandra Query Shell を起動します。*\<パスワード\>* を前の手順の既定のパスワードに置き換えます。

    ```bash
    cqlsh -u cassandra -p <password>
    ```

1. 「**cassandra@cqlsh**」プロンプトで、次のコマンドを実行します。このコマンドを実行すると、「**customerinfo.customerdetails**」テーブルの最初の 100 行が表示されます。

    ```cqlsh
    select *
    from customerinfo.customerdetails
    limit 100;
    ```

    データが「**stateprovince**」列によってグループ化された後、「**customerid**」で並べ替えられていることに注意してください。このグループ化により、アプリケーションで同じリージョンにいるすべての顧客をすばやく見つけることができます。

1. 次のコマンドを実行します。このコマンドを実行すると、「**orderinfo.orderdetails**」テーブルの最初の 100 行が表示されます。

    ```cqlsh
    select *
    from orderinfo.orderdetails
    limit 100;
    ```

    「**orderinfo.orderdetails**」テーブルには、各顧客によって行われた注文の一覧が含まれます。記録されるデータには、注文が行われた日付と注文の値が含まれます。データは「**customerid**」列によってグループ化されているので、指定された顧客のすべての注文をアプリケーションですばやく検索できます。

1. 次のコマンドを実行します。このコマンドを実行すると、「**orderinfo.orderline**」テーブルの最初の 100 行が表示されます。

    ```cqlsh
    select *
    from orderinfo.orderline
    limit 100;
    ```

    このテーブルには、各注文の品目が含まれています。データは「**orderid**」列によってグループ化され、「**orderline**」で並べ替えられています。

1. Cassandra Query Shell を終了します。

    ```cqlsh
    exit;
    ```

1. 「**Bitnami@cassandraserver**」プロンプトで、次のコマンドを入力して Cassandra サーバーから切断し、Cloud Shell に戻ります。

    ```bash
    exit
    ```

## 演習 2: CQLSH COPY コマンドを使用して Cassandra から Cosmos DB にデータを移行する 

これで、Cassandra データベースが作成され、設定されました。この演習では、Cassandra API を使用して Cosmos DB アカウントを作成し、Cassandra データベースから Cosmos DB アカウントのデータベースにデータを移行します。

### タスク 1: Cosmos アカウントとデータベースを作成する

1. Azure portal に戻ります。
1. 左側のウィンドウで、「**+ リソースの作成**」を選択します。
1. 「**新規**」ページの「**Marketplace を検索**」ボックスに「**Azure Cosmos DB**」と入力し、Enter キーを押します。
1. 「**Azure Cosmos DB**」ページで、「**作成**」を選択します。
1. 「**Azure Cosmos DB アカウントの作成**」ページで、次の設定を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | サブスクリプションを選択します |
    | リソース グループ | cassandradbrg |
    | アカウント名 | cassandra*nnn*、 *nnn*はあなたが選択した乱数です。 |
    | API | Cassandra |
    | ノートブック | オフ |
    | 場所 | Cassandra サーバーと仮想ネットワークに使用したのと同じ場所を指定します |
    | 容量モード | プロビジョニング スループット |
    | Free レベル割引の適用 | 適用 |
    | 勘定タイプ | 非運用 |
    | geo 冗長性 | Disable |
    | マルチ リージョン書き込み | Disable |
    | 可用性ゾーン | Disable |

1. 「**Review + create**」を選択します。
1. 検証ページで、「**作成**」を選択し、Cosmos DB アカウントがデプロイされるのを待ちます。
1. 左側のペインで、「**すべてのリソース**」をクリックします。
1. 「**すべてのリソース**」ページで、お使いの Cosmos DB アカウント (*cassandrannn***) を選択します。
1. 「**cassandra*nnn***」ページで、「**データ エクスプローラー**」を選択します。
1. 「**データ エクスプローラー**」ペインで、「**新しいテーブル**」を選択します。
1. 「**テーブルの追加**」ペインで、次の設定を指定してから、「**OK**」を選択します。

    | プロパティ  | 値  |
    |---|---|
    | キースペース名 | 「**新規作成**」を選択し、「**customerinfo**」と入力します |
    | キースペース スループットをプロビジョニングする | 選択解除 |
    | tableId を入力する | customerdetails |
    | *テーブルの作成*ボックス | (customerid int、firstname text、lastname text、email text、stateprovince text、PRIMARY KEY ((stateprovince)、customerid)) |
    | スループット | 10000 |

1. 「**データ エクスプローラー**」ペインで、「**新しいテーブル**」を選択します。
1. 「**テーブルの追加**」ペインで、次の設定を指定します。

    | プロパティ  | 値  |
    |---|---|
    | キースペース名 | 「**新規作成**」を選択し、「**orderinfo**」と入力します |
    | キースペース スループットをプロビジョニングする | 選択解除 |
    | tableId を入力する | orderdetails |
    | *テーブルの作成*ボックス | (orderid int、customerid int、orderdate date、ordervalue decimal、PRIMARY KEY ((customerid)、orderdate、orderid)) |
    | スループット | 10000 |

1. 「**OK**」を選択します。
1. 「**データ エクスプローラー**」ペインで、「**新しいテーブル**」を選択します。
1. 「**テーブルの追加**」ペインで、次の設定を指定します。

    | プロパティ  | 値  |
    |---|---|
    | キースペース名 | 「**既存の使用**」を選択してから、「**myResourceGroup**」を選択する |
    | tableId を入力する | orderline |
    | *テーブルの作成*ボックス | (orderid int、orderline int、productname text、quantity smallint、orderlinecost decimal、PRIMARY KEY ((orderid)、productname、orderline)) |
    | スループット | 10000 |

1. 「**OK**」を選択します。

### タスク 2: Cassandra データベースからデータをエクスポートする

1. Cloud Shell に戻ります。
1. 以下のコマンドを実行し、cassandra サーバーに接続します。*\<ip アドレス\>* を仮想マシンの IP アドレスに置き換えます。以下のプロンプトが表示されたら、パスワード「**Pa55w.rdPa55w.rd**」を入力します。

    ```bash
    ssh azureuser@<ip address>
    ```

1. Cassandra クエリ シェルを起動します。「**bitnami_credentials**」ファイルからパスワードを指定します。

    ```bash
    cqlsh -u cassandra -p <password>
    ```

1. 「**cassandra@cqlsh**」プロンプトで、次のコマンドを実行します。このコマンドは、**customerinfo.customerdetails** テーブルのデータをダウンロードし、**customerdata** という名前のファイルに書き込みます。このコマンドは、19119 行をエクスポートするはずです。

    ```cqlsh
    copy customerinfo.customerdetails
    to 'customerdata';
    ```

1. 次のコマンドを実行して、**orderinfo.orderdetails** テーブルのデータを **orderdata** という名前のファイルにエクスポートします。このコマンドは、31465 行をエクスポートするはずです。

   ```cqlsh
    copy orderinfo.orderdetails
    to 'orderdata';
    ```

1. 次のコマンドを実行して、**orderinfo.orderline** テーブルのデータを **orderline** という名前のファイルにエクスポートします。このコマンドは、121317 行をエクスポートするはずです。

   ```cqlsh
    copy orderinfo.orderline
    to 'orderline';
    ```

1. Cassandra Query Shell を終了します。

    ```cqlsh
    exit;
    ```

### タスク 3: Cosmos DB にデータをインポートする

1. Azure portal で Cosmos DB アカウントに切り替えます。
1. 「**設定**」で「**接続文字列**」をクリックし、次の項目を記録しておきます。

   - コンタクト ポイント
   - ポート
   - ユーザー名
   - プライマリ パスワード

1. Cassandra サーバーに戻り、Cassandra クエリ シェルを起動します。今回は、Cosmos DB アカウントに接続します。**cqlsh** コマンドの引数を、先ほど書き留めた値に置き換えます。

    ```bash
    export SSL_VERSION=TLSv1_2
    export SSL_VALIDATE=false

    cqlsh <contact point> <port> -u <username> -p <primary password> --ssl
    ```

    Cosmos DB には SSL 接続が必要であることに注意してください。

1. 「**cqlsh**」プロンプトで、次のコマンドを実行して、Cassandra データベースから以前にエクスポートしたデータをインポートします。

    ```cqlsh
    copy customerinfo.customerdetails from 'customerdata' with chunksize = 200;
    copy orderinfo.orderdetails from 'orderdata' with chunksize = 200;
    copy orderinfo.orderline from 'orderline' with chunksize = 100;
    ```

    これらのコマンドは、19119 **customerdetails** 行、31465 **orderdetails** 行、および 121317 **orderline** 行をインポートするはずです。これらのコマンドがタイムアウト エラーで失敗した場合は、次のいずれかの方法で問題を解決できます。
    - Azure portal の対応するテーブルのスループットを増やす (その後、スループット料金が過剰に発生しないように、再度減らす)、または
    - 各コピー操作のチャンク サイズを小さくする。チャンク サイズを小さくするとインジェスト速度が遅くなりますが、スループットを上げるとコストが増加します。

### タスク 4: 正常にデータが移行されたことを検証する

1. Azure portal で Cosmos DB アカウントに戻り、「**データ エクスプローラー**」を選択します。
1. 「**データ エクスプローラー**」ペインで、「**customerinfo**」キースペースを展開し、「**customerdetails**」テーブルを展開して、「**行**」を選択します。一連の顧客が表示されることを確認します。
1. 「**新しい節の追加**」を選択します。
1. 「**フィールド**」ボックスで、「**stateprovince**」を選択し、「**値**」ボックスに「**タスマニア**」と入力します。
1. ツールバーで、「**クエリの実行**」を選択します。クエリが 106 行を返すことを確認します。
1. 「**データ エクスプローラー**」ペインで、「**orderinfo**」キースペースを展開し、「**orderdetails**」テーブルを展開してから、「**行**」を選択します。
1. 「**新しい節の追加**」を選択します。
1. 「**フィールド**」ボックスで、「**customerid**」選択し、「**値**」ボックスに「**13999**」と入力します。
1. ツールバーで、「**クエリの実行**」を選択します。クエリが 2 行を返すことを確認します。最初の行の **orderid** に注意してください (46899 である必要があります)。
1. 「**データ エクスプローラー**」ペインで、「**orderinfo**」キースペースを展開し、「**orderline**」テーブルを展開してから、「**行**」を選択します。
1. 「**新しい節の追加**」を選択します。
1. 「**フィールド**」ボックスで、「**orderid**」を選択し、「**値**」ボックスに「**46899**」と入力します。
1. ツールバーで、「**クエリの実行**」を選択します。クエリが 1 行を返すことを確認し、**Road-550-W Yellow, 38** として注文されている製品を一覧表示します。

CQLSH COPY コマンドを使用して、Cassandra データベースを Cosmos DB に正常に移行しました。

### タスク 5: クリーンアップする

1. Cassandra サーバーで実行されている Cassandra クエリ シェルに切り替えます。このシェルは引き続き Cosmos DB アカウントに接続されている必要があります。前にシェルを閉じた場合は、次のようにシェルを再度開きます。

    ```bash
    export SSL_VERSION=TLSv1_2
    export SSL_VALIDATE=false

    cqlsh <contact point> <port> -u <username> -p <primary password> --ssl
    ```

1. Cassandra クエリ シェルで、次のコマンドを実行してキースペース (およびテーブル) を削除します。

    ```cqlsh
    drop keyspace customerinfo;
    drop keyspace orderinfo;
    exit;
    ```

## 演習 3: Spark を使用して Cassandra から Cosmos DB にデータを移行する

この演習では、前に使用したのと同じデータを移行しますが、今回は Azure Databricks ノートブックから Spark を使用します。

### タスク 1: Spark クラスターの作成

1. Azure portal の左側のペインで、「** + リソースの作成**」を選択します。
1. 「**新規**」ペインの「**Marketplace の検索**」ボックスに「**Azure Databricks**」と入力し、Enter キーを押します。
1. 「**Azure Databricks**」ページで「**作成**」を選択します。
1. 「**Azure Databricks サービス**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | 既存の cassandradbrg を使用します |
    | ワークスペース名 | CassandraMigration |
    | 場所 | リソース グループに指定したのと同じ場所を選択します |
    | 価格レベル | Standard |

1. 「**Review + create**」を選択します。
1. 「**Review + create**」ページで「**作成**」を選択し、Databricks サービスがデプロイされるまで待ちます。
1. 左側のペインで、「**リソース グループ**」を選択し、「**cassandradbrg**」を選択して、「**CassandraMigration**」 Databricks サービスを選択します。
1. 「**CassandraMigration**」ページで、「**ワークスペースの起動**」を選択します。
1. 「**Azure Databricks**」ページの「**主なタスク**」で、「**新しいクラスター**」を選択します。
1. 「**新しいクラスター**」ページで、次の設定を入力します。

    | プロパティ  | 値  |
    |---|---|
    | クラスター名 | MigrationCluster |
    | クラスター モード | Standard |
    | プール | None |
    | Databricks ランタイム バージョン | ランタイム: 5.5 LTS (Scala 2.11、Spark 2.4.3) |
    | Python バージョン | 3 |
    | 自動スケーリングを有効にする | 選択済み |
    | 終了までの時間 | 60 |
    | worker の種類 | 設定は既定値のままにしておきます |
    | ドライバーの種類 | ワーカーと同じ |

    > [!注]
    > ランタイム バージョンを選択するときは、GPU サポートのないバージョンを選択してください。そうしないと、非互換エラー メッセージが表示されます。

1. 「**クラスターの作成**」を選択します。
1. クラスターが作成されるまで待ちます。クラスターの準備が完了すると、「**MigrationCluster**」の状態が「**実行中**」として報告されます。このプロセスには数分かかります。

### タスク 2: データを移行するためのノートブックを作成する

1. 左側のペインで、「**クラスター**」を選択し、「**MigrationCluster**」を選択して、「**ライブラリ**」タブを選択してから、「**今すぐインストール**」を選択します。
1. 「**ライブラリのインストール**」ダイアログ ボックスで、次の設定を入力します。

    | プロパティ  | 値  |
    |---|---|
    | ライブラリ ソース | Maven |
    | 座標 | com.datastax.spark:spark-cassandra-connector_2.11:2.4.3 |
    | リポジトリ | 空白のままにします |
    | 除外 | 空白のままにします |

1. 「**インストール**」を選択します。このライブラリには、Spark から Cassandra に接続するためのクラスが含まれています。
1. コネクタ ライブラリがインストールされたら、「**今すぐインストール**」を選択して、別のライブラリをインストールします。
1. 「**ライブラリのインストール**」ダイアログ ボックスで、次の設定を入力します。

    | プロパティ  | 値  |
    |---|---|
    | ライブラリ ソース | Maven |
    | 座標 | com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.2.0 |
    | リポジトリ | 空白のままにします |
    | 除外 | 空白のままにします |

1. 「**インストール**」を選択します。このライブラリには、Spark から Cosmos DB に接続するためのクラスが含まれています。
1. 左側のペインで、「**Azure Databricks**」を選択します。
1. 「**Azure Databricks**」ページの「**主なタスク**」で、「**新しいノートブック**」を選択します。
1. 「**ノートブックの作成**」ダイアログ ボックスで、次の設定を入力します。

    | プロパティ  | 値  |
    |---|---|
    | 名前 | MigrateData |
    | 言語 | Scala |
    | クラスター | MigrationCluster |

1. 「**作成**」を選択します。

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

    このコードにより、Spark から Cosmos DB と Cassandra に接続するために必要な型がインポートされます。

1. セルの右側にあるツール バーで、ドロップダウン矢印を選択し、「**下にセルを追加**」を選択します。
1. 新しいセルに次のコードを入力します。コンタクト ポイント、ユーザー名、プライマリ パスワードに、Cosmos DB アカウントの値を指定します (これらの値は、前の演習で記録したものです)。

    ```scala
    // Cosmos DB の接続パラメーターを構成する

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

    このコードにより、Cosmos DB アカウントに接続するための Spark セッション パラメーターが設定されます

1. 現在のセルの下に別のセルを追加し、次のコードを入力します。

    ```scala
    // キースペースとテーブルを作成する

    val cosmosDBConnector = CassandraConnector(cosmosDBConf)

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE KEYSPACE customerinfo WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}"))
    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE customerinfo.customerdetails (customerid int, firstname text, lastname text, email text, stateprovince text, PRIMARY KEY ((stateprovince), customerid)) WITH cosmosdb_provisioned_throughput=10000"))

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE KEYSPACE orderinfo WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}"))
    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE orderinfo.orderdetails (orderid int, customerid int, orderdate date, ordervalue decimal, PRIMARY KEY ((customerid), orderdate, orderid)) WITH cosmosdb_provisioned_throughput=10000"))

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE orderinfo.orderline (orderid int, orderline int, productname text, quantity smallint, orderlinecost decimal, PRIMARY KEY ((orderid), productname, orderline)) WITH cosmosdb_provisioned_throughput=10000"))
    ```

    これらのステートメントにより、orderinfo および customerinfo のキースペースとテーブルが再構築されます。各テーブルは、10000 RU/s のスループットでプロビジョニングされます。

### タスク 4: Cassandra データベースに接続してデータを取得する

1. ノートブックに別のセルを追加し、次のコードを入力します。*\<ip アドレス\>* を仮想マシンの IP アドレスに置き換え、**bitnami_credentials** ファイルから以前に取得したパスワードを指定します。

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

1. 別のセルを追加し、次のコードを入力します。

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

    このコード ブロックにより、Cassandra データベース内のテーブルから Spark DataFrame オブジェクトに、データが取得されます。このコードにより、各テーブルから読み取られた行数が表示されます。

### タスク 5: Cosmos DB テーブルにデータを挿入し、ノートブックを実行する

1. 最後のセルを追加し、次のコードを入力します。

    ```scala
    // 顧客データを Cosmos DB に書き出す

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

    // 同じ戦略を使用して、注文データを Cosmos DB に書き出す
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

    このコードにより、Cosmos DB データベースのテーブルごとに別の DataFrame が作成されます。各 DataFrame は、最初は空の状態です。その後、コードにより、**union** 関数を使用して、各 Cassandra テーブルに対応する DataFrame からデータが追加されます。最後に、コードにより、追加された DataFrame が Cosmos DB のテーブルに書き戻されます。

    DataFrame API は、Spark によって提供される非常に強力な抽象化であり、大量のデータを非常に高速に転送するための効率的な構造です。

1. ノートブックの上部にあるツール バーで、「**すべて実行**」を選択します。  クラスターが起動中であることを示すメッセージが表示されます。クラスターの準備が整うと、ノートブックにより各セルのコードが順番に実行されます。各セルの下に、さらにメッセージが表示されます。DataFrame の読み取りと書き込みを行うデータ転送操作は、Spark ジョブとして実行されます。ジョブを展開して進行状況を表示できます。各セルのコードは、エラー メッセージを表示せずに、正常に完了する必要があります。

### タスク 6: 正常にデータが移行されたことを検証する

1. Azure portal で Cosmos DB アカウントに戻ります。
1. 「**データ エクスプローラー**」を選択します。
1. 「**データ エクスプローラー**」ペインで、「**customerinfo**」キースペースを展開し、「**customerdetails**」テーブルを展開して、「**行**」を選択します。最初の 100 行が表示されます。「**データ エクスプローラー**」ペインにキースペースが表示されない場合は、「**更新**」を選択して表示を更新します。
1. 「**orderinfo**」キースペースを展開し、「**orderdetails**」テーブルを展開して、「**行**」を選択します。このテーブルについても、最初の 100 行が表示されます。
1. 最後に、「**orderline**」テーブルを展開して、「**行**」を選択します。このテーブルの最初の 100 行が表示されることを確認します。

Databricks ノートブックから Spark を使用して、Cassandra データベースを Cosmos DB に正常に移行しました。

### タスク 7: クリーンアップする

1. Azure portal の左側のペインで、「**リソース グループ**」を選択します。
1. 「**リソース グループ**」ウィンドウで、「**cassandradbrg**」を選択します。
1. 「**リソース グループの削除**」を選択します。
1. 「**"cassandradbrg" を削除しますか**」ページで、「**リソース グループ名を入力**」ボックスに「**cassandradbrg**」と入力してから、「**削除**」を選択します。

---
© 2019 Microsoft Corporation. All rights reserved.

このドキュメントのテキストは、 クリエイティブ・コモンズ表示 3.0 ライセンスで入手できます。
ライセンス](https://creativecommons.org/licenses/by/3.0/legalcode)、追加条件が適用される場合があります。このドキュメントに含まれるその他すべてのコンテンツ (商標、ロゴ、画像等を含むがこれに限定されない) は、
クリエイティブ・コモンズ・ライセンス付与には**含まれていません**。このドキュメントは、Microsoft 製品に含まれる知的財産に対するいかなる法的権利も提供するものではありません。お客様の社内での参照目的に限り、このドキュメントをコピーし使用することができます。

このドキュメントは "現状のまま" 提供されます。このドキュメントに記載された情報および見解は、URL やその他のインターネット Web サイトの参照も含め、予告なく変更する可能性があります。お客様は、その使用に関するリスクを負うものとします。一部の例は説明のみを目的としており、架空のものです。実際の団体とは一切関係ありません。Microsoft は、以下の情報に対して、明示または黙示を問わず、一切の保証を行わないものとします。
