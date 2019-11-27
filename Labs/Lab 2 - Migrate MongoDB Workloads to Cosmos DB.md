---
lab:
    title: 'MongoDB ワークロードを Cosmos DB に移行する'
    module: 'モジュール 2: MongoDB ワークロードを Cosmos DB に移行する'
---

- [課題 2: MongoDB ワークロードを Cosmos DB に移行する](#lab-2-migrate-mongdb-workloads-to-cosmos-db)
  - [エクササイズ 1: セットアップ](#exercise-1-setup)
    - [タスク 1: リソース グループと Virtual Network の作成](#task-1-create-a-resource-group-and-virtual-network)
    - [タスク 2: MongoDB データベース サーバーの作成](#task-2-create-a-mongodb-database-server)
    - [タスク 3: MongoDB データベースの構成](#task-3-configure-the-mongodb-database)
  - [エクササイズ 2: クエリと MongoDB データベースの設定](#exercise-2-populate-and-query-the-mongodb-database)
    - [タスク 1: アプリをビルドして実行し、MongoDB データベースを設定する](#task-1-build-and-run-an-app-to-populate-the-mongodb-database)
    - [タスク 2: 別のアプリをビルドして実行し、MongoDB データベースをクエリする](#task-2-build-and-run-another-app-to-query-the-mongodb-database)
  - [エクササイズ 3: MongoDB ワークロードを Cosmos DB に移行する](#exercise-3-migrate-the-mongodb-database-to-cosmos-db)
    - [タスク 1: Cosmos アカウントとデータベースの作成](#task-1-create-a-cosmos-account-and-database)
    - [タスク 2: Database Migration Service を作成する](#task-2-create-the-database-migration-service)
    - [タスク 3: 新しい移行プロジェクトの作成と実行](#task-3-create-and-run-a-new-migration-project)
    - [タスク 4: データ移行が成功したことを確認する](#task-4-verify-that-migration-was-successful)
  - [エクササイズ 4: Cosmos DB を使用するために既存のアプリケーションを再構成および実行する](#exercise-4-reconfigure-and-run-existing-applications-to-use-cosmos-db)
    - [タスク 1: MongoDB 集計サポートを有効にする](#task-1-enable-mongodb-aggregation-support)
    - [タスク 2: DeviceDataQuery アプリケーションを再構成する](#task-2-reconfigure-the-devicedataquery-application)
  - [エクササイズ 5: クリーンアップ](#exercise-5-clean-up)

# 課題 2: MongoDB ワークロードを Cosmos DB に移行する

この課題では、既存の MongoDB データベースを Cosmos DB に移行します。Azure Database Migration Service を使用します。代わりに、MongoDB データベースを使用して Cosmos DB データベースに接続する既存のアプリケーションを再構成する方法についても説明します。

この課題は、一連の IoT デバイスから温度データをキャプチャするシステム例に基づいています。温度は、タイムスタンプと共に MongoDB データベースに記録されます。各デバイスには一意の ID があります。これらのデバイスをシミュレートし、データベースにデータを保存する MongoDB アプリケーションを実行します。また、ユーザーが各デバイスに関する統計情報を照会できるようにする 2 つ目のアプリケーションも使用します。MongoDB から Cosmos DB にデータベースを移行した後、両方のアプリケーションを構成して Cosmos DB に接続し、それらが正しく機能することを確認します。

この課題は、Azure Cloud Shell と Azure portal を使用して実行されます。

## エクササイズ 1: セットアップ

最初の演習では、温度デバイスからキャプチャしたデータを保持するための MongoDB データベースを作成します。

### タスク 1: リソース グループと Virtual Network の作成

1. インターネット ブラウザで、https://portal.azure.com に移動してサインインします。
2. Azure portal で **リソース グループ** をクリックし、**+追加** をクリックします。
3. **リソース グループの作成ページ** で、次の詳細を入力してから、**確認 + 作成** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | mongodbrg |
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
    | リソース グループ | mongodbrg |
    | リージョン | リソース グループに指定したのと同じ場所を選択する |
    | サブネット名 | デフォルト |
    | サブネットアドレス範囲 | 10.0.0.0/28 |
    | DDoS 保護 | Basic |
    | サービス エンドポイント | 無効 |
    | ファイアウォール | 無効 |

9. 続行する前に、仮想ネットワークが作成されるのを待ちます。

### タスク 2: MongoDB データベース サーバーの作成

1. Azure portal の左側のウィンドウで、**+ リソースの作成** をクリックします。
2. **マーケットプレースの検索** ボックスに、***Bitnami により認定された MongoDB** と入力し、Enter キーを押します。
3. **Bitnami により認定された MongoDB** ページで、**作成** をクリックします。
4. **仮想マシンの作成** ページで、次の詳細を入力し、**次へ** をクリックします。**Disks \>**。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | mongodbrg |
    | 仮想マシン名 | mongodbserver | 
    | リージョン | リソース グループに指定したのと同じ場所を選択する |
    | 可用性オプション | インフラストラクチャの冗長性は不要 |
    | イメージ | Bitnami により認定された MongoDB |
    | サイズ： | Standard A1 v2 |
    | 認証タイプ | パスワード |
    | ユーザー名 | azureuser |
    | パスワード | Pa55w.rdPa55w.rd |
    | パスワードを確定する | Pa55w.rdPa55w.rd |

5. **ディスク** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。**Networking \>**。
6. **ネットワーク** ページで、次の詳細を入力してから、**次へ:** をクリックします。**Management \>**。

    | プロパティ  | 値  |
    |---|---|
    | 仮想ネットワーク | databasevnet |
    | サブネット | 規定値 (10.0.0.0/28) |
    | パブリック IP | (新規) mongodbserver-ip |
    | NIC ネットワーク セキュリティ グループ | 詳細 |
    ! ネットワーク セキュリティ グループの構成 | (新規) mongodbserver-nsg |
    | 高速ネットワーキング | オフ |
    | 負荷分散 | いいえ |

7. **管理** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。**Advanced \>**。
8. **詳細** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。**Tags \>**。
9. **タグ** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。**Review + create \>**。
10. 検証ページで、**作成** をクリックします。
11. 続行する前に、仮想ネットワークがデプロイされるのを待ちます。
12. Azure portal の左側のウィンドウで、**すべてのリソース** をクリックします。
13. **すべてのリソース** ページで、**mongodbserver-nsg** をクリックします。
14. **mongodbserver-nsg** ページの **設定** で、**受信セキュリティ規則** をクリックします。
15. **mongodbserver-nsg - 受信セキュリティ規則** ページで、**+ 追加** をクリックします。
16. **受信セキュリティ規則の追加** ウィンドウで、次の詳細を入力してから、**追加** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | ソース | 任意 |
    | ソースのポート範囲 | * |
    | 宛先 | 任意 |
    | ターゲットのポート範囲 | 27017 |
    | プロトコル | 任意 |
    | アクション | 許可 |
    | 優先度 | 1020 |
    | 名前 | Mongodb-port |
    | 説明 | クライアントが MongoDB への接続に使用するポート |

### タスク 3: MongoDB データベースの構成

1. Azure portal の左側のウィンドウで、**すべてのリソース** をクリックします。
2. **すべてのリソース** ページで、**mongodbserver-ip** をクリックします。
3. **mongodbserver-ip** ページで、**IP アドレス** を書き留めます。
4. Azure portal の上部にあるツール バーで、**クラウド シェル**をクリックします。
5. **記憶域がマウントされていない** メッセージ ボックスが表示されたら、**記憶域の作成** をクリックします。
6. Cloud Shell が起動したら、Cloud Shell ウィンドウの上にあるドロップダウン リストで **Bash** を選択します。
7. Cloud Shell で、次のコマンドを入力して mongodbserver 仮想マシンに接続します。*\<ip address\>* を **mongodbserver-ip** IP アドレスの値に置き換えます。

    ```bash
    ssh azureuser@<ip address>
    ```

8. プロンプトで、**yes** と入力して接続を続行します。
9. **Pa55w.rdPa55w.rd** パスワードを入力する
10. 次のコマンドを入力し、表示されているルート パスワードをメモします。

    ```bash
    cat bitnami_credentials
    ```

11. 以下のコマンドを実行し、MongoDB データベースに接続します。*\<password\>* を前の手順に表示されているルート パスワードに置き換えます。XFS ファイルシステムの使用に関する警告は無視してください。

    ```bash
    mongo -u root -p <password>
    ```

12. **>** プロンプトで、次のコマンドを実行します。これらのコマンドは、**DeviceData** という名前のデータベースに対して、パスワード **Pa55w.rd** を持つ **DeviceData** という名前の新しいユーザーを作成します。 `db.shutdownserver();` コマンドを実行すると、無視できるエラーが表示されます。

    ```mongosh
    use DeviceData;
    db.createUser(
        {
            user: "deviceadmin",
            pwd: "Pa55w.rd",
            roles: [ { role: "readWrite", db: "DeviceData" } ]
        }
    );
    use admin;
    db.shutdownServer();
    exit;
    ```

13. 次のコマンドを実行して、mongodb サービスを再起動します。サービスがエラー メッセージなしで再起動し、ポート 27017 でリッスンしていることを確認します。

    ```bash
    sudo /opt/bitnami/ctlscript.sh start
    ```

14. 次のコマンドを実行して、mongodb に deviceadmin ユーザーとしてログインできることを確認します。

    ```bash
    mongo -u "deviceadmin" -p "Pa55w.rd" --authenticationDatabase DeviceData
    ```

15. **>**プロンプトで、次のコマンドを実行して mongo シェルを終了します。

    ```mongosh
    exit;
    ```

16. **bitnami@mongodbserver** プロンプトで、次のコマンドを入力して MongoDB サーバーから切断し、Cloud Shell に戻ります。

    ```bash
    exit
    ```

## エクササイズ 2: クエリと MongoDB データベースの設定

これで、MongoDB サーバーとデータベースが作成されました。次の手順では、このデータベースのデータを設定およびクエリできるサンプル アプリケーションをデモンストレーションします。

### タスク 1: アプリをビルドして実行し、MongoDB データベースを設定する

1. Azure Cloud Shell で、次のコマンドを実行して、このワークショップのサンプル コードをダウンロードします。

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

2. **migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceCapture** フォルダーに移動します。

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceDataCapture
    ```

3. **コード** エディターを使用して、**TemperatureDevice.cs** ファイルを調べます。   

    ```bash
    code TemperatureDevice.cs
    ```

    このファイルのコードには、データをキャプチャし、MongoDB データベースに保存する温度デバイスをシミュレートする **TemperatureDevice** という名前のクラスが含まれています。.NET Framework に MongoDB ライブラリを使用します。**TemperatureDevice** コンストラクターは、アプリケーション構成ファイルに保存されている設定を使用してデータベースに接続します。  **RecordTemperatures** メソッドは、読み取り値を生成し、データベースに書き込みます。 

4. コード エディターを閉じ、**ThermometerReading.cs** ファイルを開きます。

   ```bash
   code ThermometerReading.cs
   ```

    このファイルには、アプリケーションがデータベースに格納するドキュメントの構造が表示されます。各ドキュメントには、次のフィールドが含まれています。

    - オブジェクト ID。各ドキュメントを一意に識別するために MongoDB によって生成される 「_id」フィールドです。
    - デバイス ID。各デバイスには、接頭辞「デバイス」とともに番号が付いています。
    - デバイスによって記録された温度
    - 温度が記録された日時。
  
5. コード エディターを閉じてから、**App.config** ファイルを開きます。 

    ```bash
    code App.config
    ```

    このファイルには、MongoDB データベースに接続するための設定が含まれています。**アドレス** キーの値を、前に記録した MongoDB サーバーの IP アドレスに設定してから、ファイルを保存してエディターを閉じます。 

6. 次のコマンドを実行して、アプリケーションを再構築します。

    ```bash
    dotnet build
    ````

7. アプリケーションを実行します。

    ```bash
    dotnet run
    ```

    アプリケーションは、同時に実行されている 100 台のデバイスをシミュレートします。アプリケーションの実行を数分間許可してから、Enter キーを押してアプリケーションを停止します。

### タスク 2: 別のアプリをビルドして実行し、MongoDB データベースをクエリする

1. **migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery** フォルダーに移動します。

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

    このフォルダーには、各デバイスによってキャプチャされたデータを分析するために使用できる別のアプリケーションが含まれています。

2. **コード** エディターを使用して、**Program.cs** ファイルを調べます。   

    ```bash
    code Program.cs
    ```

    アプリケーションはデータベースに接続します (ファイルの下部にある **ConnectToDatabase** メソッドを使用して、デバイス番号の入力をユーザーに求めます。  アプリケーションは、.NET Framework の MongoDB ライブラリを使用して、指定されたデバイスの次の統計情報を計算する集計パイプラインを作成して実行します。

    - 読み取り値の数が記録されました。
    - 平均気温が記録されました。
    - 最低の読み取り値です。
    - 最高の読み取り値です。
    - 最新の読み取り値です。

3. コード エディターを閉じてから、**App.config** ファイルを開きます。

    ```bash
    code App.config
    ```

    前と同様に、**アドレス** キーの値を、前に記録した MongoDB サーバーの IP アドレスに設定してから、ファイルを保存してエディターを閉じます。 

4. アプリケーションをビルドして実行します。

    ```bash
    dotnet build
    dotnet run
    ```

5. **デバイス番号の入力** プロンプトで、0 から 99 の値を入力します。アプリケーションは、データベースをクエリし、統計情報を計算し、結果を表示します。**Q** キーを押してアプリケーションを終了します。 

## エクササイズ 3: MongoDB ワークロードを Cosmos DB に移行する

次の手順では、MongoDB データベースを取得し、Cosmos DB に転送します。

### タスク 1: Cosmos アカウントとデータベースの作成

1. Azure portal に戻ります。
2. 左側のウィンドウで、**+ リソースの作成** をクリックします。
3. **新規** ページの **マーケットプレースの検索** ボックスに　***Azure CosmosDB** と入力してから、Enter キーを押します。
4. **Azure Cosmos DB** ページで、**作成** をクリックします。
5. **Azure Cosmos DB アカウントの作成** ページで、次の設定を入力してから、**確認 + 作成** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | サブスクリプションを選択します |
    | リソース グループ | mongodbrg |
    | アカウント名 | mongodb*nnn*、 *nnn*はあなたが選択した乱数です。 |
    | API | Azure Cosmos DB for MongoDB API |
    | 場所 | MongoDB サーバーと仮想ネットワークに使用したのと同じ場所を指定します |
    | 地理的冗長性 | 無効にする |
    | 複数リージョンの書き込み | 無効にする |

6. 検証ページで、**作成**をクリックし、Cosmos DB アカウントがデプロイされるまで待ちます。
7. 左側のウィンドウで、 **Azure Cosmos DB** をクリックします。
8. **Azure Cosmos DB** ページで、Cosmos DB アカウント (**mongodb*nnn***) をクリックします。
9. **mongodb*nnn*** ページで、**データ エクスプローラー** をクリックします。
10. **データ エクスプローラー** ウィンドウで、**新しいコレクション** をクリックします。
11. **コレクションの追加** ウィンドウで、次の設定を指定してから、**OK** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | データベース ID | **新規作成**をクリックしてから、**DeviceData** と入力します。 |
    | データベース スループットのプロビジョニング | 選択済み |
    | スループット | 1000 |
    | コレクション id | 温度 |
    | シャード キー | deviceID |

### タスク 2: Database Migration Service を作成する

1. 左側のウィンドウで、**すべてのサービス** をクリックします。
2. **すべてのサービス** ページで、**サブスクリプション** をクリックします。
3. **サブスクリプション**ページで、サブスクリプションをクリックします。 
4. サブスクリプション ページの **設定**で、**リソース プロバイダー** をクリックします。   
5. **名前でフィルター** ボックスに **DataMigration** と入力してから、**Microsoft.DataMigration** をクリックします。
6. **登録** をクリックし、**状態** が **登録済み** に変わるのを待ちます。      **最新の情報に更新** をクリックして、変更する状態を確認する必要がある場合があります。
7. 左側のウィンドウで、**+ リソースの作成** をクリックします。
8. **新規** ページの **マーケットプレースの検索** ボックスに **Azure Database Migration Service** と入力してから、Enter キーを押します。
9. **Azure Database Migration Service** ページで、**作成** をクリックします。
10. **移行サービスの作成** ページで、次の設定を入力してから、**作成** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | サービス名 | MongoDBMigration |
    | サブスクリプション | サブスクリプションを選択する |
    | リソースグループを選択する | mongodbrg |
    | 場所 | 以前に使用したのと同じ場所を選択する |
    | 仮想ネットワーク | **仮想ネットワークの選択または作成** をクリックし、**databasevnet/default** を選択してから、**OK** をクリックします。 |
    | 価格レベル | Standard: 1 vCores |

11. 続行する前に、サービスがデプロイされるのを待ちます。この操作には数分かかります。

### タスク 3: 新しい移行プロジェクトの作成と実行

1. 左側のウィンドウで、**リソース グループ**をクリックします。
2. **Resource groups** ウィンドウで、**mongodbrg** をクリックします。
3. **mongodbrg** ウィンドウで、**MongoDBMigration** をクリックします。
4. **MongoDBMigration** ページで、**+ 新しい移行プロジェクト** をクリックします。   
5. **新しい移行プロジェクト** ページで、次の設定を入力してから、**アクティビティの作成と実行** をクリックします。   

    | プロパティ  | 値  |
    |---|---|
    | プロジェクト名 | MigrateTemperatureData |
    | ソース サーバー タイプ | MongoDB |
    | ターゲット サーバー タイプ | Cosmos DB (MongoDB API) |
    | アクティビティ タイプの選択 | オフライン データ移行 |

6. **移行ウィザード** が起動したら、**ソースの詳細** ページで次の詳細を入力してから、**保存** をクリックします。     

    | プロパティ  | 値  |
    |---|---|
    | モード | 標準モード |
    | ソース サーバー名 | 前に記録した **mongodbserver-ip** IP アドレスの値を指定します。 |
    | ユーザー名 | ルート |
    | パスワード | 前にメモした mongodbserver VM 上のルート ユーザーのパスワードを入力します (bitnami_credentials ファイルから) |
    | SSL が必要 | 空白のままにする |

7. **移行詳細** ページで、次の詳細を入力してから、**保存** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | モード | Cosmos DB ターゲットの選択 |
    | サブスクリプション | サブスクリプションを選択します |
    | Cosmos DB 名の選択 | mongodb*nnn* |
    | 接続文字列 | Cosmos DB アカウント用に生成された接続文字列を受け入れる |

8. **ターゲット データベースへマッピング** ページで、次の詳細を入力してから、**保存** をクリックします。

    | プロパティ  | 値  |
    |---|---|
    | ソース データベース | DeviceData |
    | ターゲット データベース | DeviceData |
    | スループット (RU/s) | 1000 |
    | コレクションのクリーンアップ | このチェック ボックスをクリアする |

9. **コレクションの設定** ページで、DeviceData データベースのドロップダウン矢印をクリックし、次の詳細を入力してから、**保存** をクリックします。   

    | プロパティ  | 値  |
    |---|---|
    | 名前 | 温度 |
    | ターゲット コレクション | 温度 |
    | スループット (RU/s) | 1000 |
    | シャード キー | deviceID |
    | 一意 | 空白のままにする |

10. **移行の概要** ページの **アクティビティ名** フィールド に **mongodb-migration** と入力し、**初期データ コピー時に RU をブーストする** を選択してから、**移行の実行** をクリックします。         
11. **mongodb-migration** ページで、移行が完了するまで 30 秒ごとに **最新の情報に更新** をクリックします。移行されたドキュメントの数に注意してください。

### タスク 4: データ移行が成功したことを確認する

1. 左側のウィンドウで、 **Azure Cosmos DB** をクリックします。
2. **Azure Cosmos DB** ページで、**mongodb*nnn*** をクリックします。
3. **mongodb*nnn** ページで、**データ エクスプローラー** をクリックします。
4. **データ エクスプローラー** ウィンドウで、**温度** データベースを展開し、**ドキュメント** をクリックします。     
5. **ドキュメント** ウィンドウで、ドキュメントの一覧をスクロールします。  各ドキュメントのドキュメント ID (**_id**) とシャード キー (**/deviceID**) が表示されます。
6. 任意のドキュメントをクリックします。ドキュメントの詳細が表示されます。一般的なドキュメントは次のようになります。

    ```JSON
    {
	    "_id" : ObjectId("5ce8104bf56e8a04a2d0929a"),
	    "deviceID" : "Device 83",
	    "temperature" : 19.65268837271849,
	    "time" : 636943091952553500
    }
    ```

7. **ドキュメント エクスプローラー** ウィンドウのツール バーで、**新しいシェル** をクリックします。   
8. **Shell 1** ウィンドウの **\>**プロンプトで、次のコマンドを入力してから、Enter キーを押します。   

    ```mongosh
    db.Temperatures.count()
    ```

    このコマンドは、Temperatures コレクション内のドキュメントの数を表示します。移行ウィザードによってレポートされる番号と一致する必要があります。

9. 次のコマンドを入力してから、Enter キーを押します。

    ```mongosh
    db.Temperatures.find({deviceID: "Device 99"})
    ```

    このコマンドは、デバイス 99 のドキュメントを取り込んで表示します。

## エクササイズ 4: Cosmos DB を使用するために既存のアプリケーションを再構成および実行する

最後の手順では、既存の MongoDB アプリケーションを再構成して Cosmos DB に接続し、それらが以前と同じように動作することを確認します。このプロセスでは、アプリケーションがデータベースに接続する方法を変更する必要がありますが、アプリケーションのロジックは変更されません。

### タスク 1: MongoDB 集計サポートを有効にする

1. **mongodb*nnn*** ウィンドウの **設定** で、**プレビュー機能** をクリックします。
2. **集約パイプライン** の横にある **有効にする** をクリックします。    集計パイプラインが有効になるまで待ちます。この機能は、MongoDB の集計および Cosmos DB データベースに対するグループ化操作を追加します。集計パイプラインが使用可能になるまでに数分かかる場合があります。ブラウザー ウィンドウを更新して、機能の最新の状態を確認します。

### タスク 2: DeviceDataQuery アプリケーションを再構成する

1. **mongodb*nnn*** ウィンドウの **設定** で、**接続文字列** をクリックします。
2. **mongodb*nnn* 接続文字列** ページで、次の設定を書き留めます。  

    - ホスト
    - ユーザー名
    - プライマリ パスワード
  
3. Cloud Shell ウィンドウに戻り (セッションがタイムアウトした場合は再接続 します)、**migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery** フォルダーに移動します。 

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

4. コード エディターで App.config ファイルを開きます。

    ```bash
    code App.config
    ```

5. ファイルの **MongoDB の設定** セクションで、既存の設定をコメント アウトします。 
6. **Cosmos DB Mongo API の設定** セクションの設定のコメントを解除 し、これらの設定の値を次のように設定します。 

    | 設定  | 値  |
    |---|---|
    | 住所 | **mongodb*nnn* 接続文字列** ページのホスト |
    | ユーザー名 | **mongodb*nnn* 接続文字列** ページのユーザー名 |
    | パスワード | **mongodb*nnn* 接続文字列** ページのプライマリ パスワード |

    完成したファイルは次のようになります。

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="Database" value="DeviceData" />
            <add key="Collection" value="Temperatures" />

            <!-- Settings for MongoDB -->
            <!--add key="Address" value="168.63.99.236" />
            <add key="Port" value="27017" />
            <add key="Username" value="deviceadmin" />
            <add key="Password" value="Pa55w.rd" /-->
            <!-- End of settings for MongoDB -->

            <!-- Settings for CosmosDB Mongo API -->
            <add key="Address" value="mongodbnnn.documents.azure.com"/>
            <add key="Port" value="10255"/>
            <add key="Username" value="mongodbnnn"/>
            <add key="Password" value="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=="/>
            <!-- End of settings for CosmosDB Mongo API -->
        </appSettings>
    </configuration>
    ```

7. ファイルを保存し、コード エディターを閉じます。

8. コード エディターを使用して Program.cs ファイルを開きます。

    ```bash
    code Program.cs
    ```

9. **ConnectToDatabase** メソッドまで下にスクロールします。
10. MongoDB に接続するための資格情報を設定する行をコメントアウトし、Cosmos DB に接続するための資格情報を指定するステートメントのコメントを解除します。コードは、以下のようになるはずです。

    ```C#
    // MongoDB データベースに接続する
    MongoClient client = new MongoClient(new MongoClientSettings
    {
        Server = new MongoServerAddress(address, port),
        ServerSelectionTimeout = TimeSpan.FromSeconds(10),

        //
        // MongoDB の資格情報の設定
        //

        // Credential = MongoCredential.CreateCredential(database, azureLogin.UserName, azureLogin.SecurePassword),

        //
        // CosmosDB Mongo API 向けの資格情報設定
        //

        UseSsl = true,
        SslSettings = new SslSettings
        {
            EnabledSslProtocols = SslProtocols.Tls12
        },
        Credential = new MongoCredential("SCRAM-SHA-1", new MongoInternalIdentity(database, azureLogin.UserName), new PasswordEvidence(azureLogin.SecurePassword))

        // Mongo API 設定の終了 
    });

    ```

    元の MongoDB データベースが SSL 接続を使用していなかったため、これらの変更が必要です。Cosmos DB は常に SSL を使用します。

11. ファイルを保存し、コード エディターを閉じます。
12. アプリケーションを再構築して実行します。

    ```bash
    dotnet build
    dotnet run
    ```

13. **デバイス番号の入力**プロンプトで、0 から 99 のデバイス番号を入力します。  アプリケーションは、今回のように Cosmos DB データベースに保持されているデータを使用する場合を除き、以前とまったく同じように実行する必要があります。
14. 他のデバイス番号を使用してアプリケーションをテストします。  **Q** と入力して終了します。 

MongoDB データベースを Cosmos DB に正常に移行し、既存の MongoDB アプリケーションを再構成して Cosmos DB データベースに接続しました。

## エクササイズ 5: クリーンアップ

1. Azure portal に戻ります。
2. 左側のウィンドウで、**リソース グループ**をクリックします。
3. **Resource groups** ウィンドウで、**mongodbrg** をクリックします。
4. **リソース グループの削除** をクリックします。
5. **「mongodbrg」 を削除しますか** ページで、**リソース グループ名を入力** ボックスに **mongodbrg** 入力してから、**削除** をクリックします。       

---
© 2019 Microsoft Corporation.All rights reserved.

このドキュメントのテキストは、 [クリエイティブ・コモンズ表示 3.0 ライセンス](https://creativecommons.org/licenses/by/3.0/legalcode)で入手できます。  このドキュメントに含まれるその他すべてのコンテンツ (商標、ロゴ、画像などを含むがこれに該当する) は、 クリエイティブ・コモンズ・ライセンス付与には**含まれません**。  このドキュメントは、マイクロソフト製品の知的財産権に関する法的権利をお客様に提供するものではありません。内部での参照目的でこのドキュメントをコピーして使用することができます。

このドキュメントは「現状のまま」提供されています。このドキュメントに記載されている情報および見解 (URL およびその他のインターネット Web サイトの参照を含む) は、予告なしに変更されることがあります。その使用にはリスクが伴います。一部の例は説明のみを目的としており、架空のものです。実際の団体とは一切関係ありません。マイクロソフトは、ここに記載されている情報に関して、明示または黙示を問わず、いかなる保証も行いません。
