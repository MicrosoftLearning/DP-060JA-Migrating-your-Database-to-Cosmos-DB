
# ラボ 2: MongoDB ワークロードを Cosmos DB に移行する
<!-- TOC -->

- [ラボ 2: MongoDB ワークロードを Cosmos DB に移行する](#lab-2-migrate-mongodb-workloads-to-cosmos-db)
    - [演習 1: 設定](#exercise-1-setup)
        - [タスク 1: リソース グループと Virtual Network を作成する](#task-1-create-a-resource-group-and-virtual-network)
        - [タスク 2: MongoDB データベース サーバーを作成する](#task-2-create-a-mongodb-database-server)
        - [タスク 3: MongoDB データベースを構成する](#task-3-configure-the-mongodb-database)
    - [演習 2: クエリと MongoDB データベースを設定する](#exercise-2-populate-and-query-the-mongodb-database)
        - [タスク 1: アプリをビルドして実行し、MongoDB データベースを設定する](#task-1-build-and-run-an-app-to-populate-the-mongodb-database)
        - [タスク 2: 別のアプリをビルドして実行し、MongoDB データベースにクエリを実行する](#task-2-build-and-run-another-app-to-query-the-mongodb-database)
    - [演習 3: MongoDB データベースを Cosmos DB に移行する](#exercise-3-migrate-the-mongodb-database-to-cosmos-db)
        - [タスク 1: Cosmos アカウントとデータベースを作成する](#task-1-create-a-cosmos-account-and-database)
        - [タスク 2: Database Migration Service を作成する](#task-2-create-the-database-migration-service)
        - [タスク 3: 新しい移行プロジェクトを作成し実行する](#task-3-create-and-run-a-new-migration-project)
        - [タスク 4: 正常に移行されたことを検証する](#task-4-verify-that-migration-was-successful)
    - [演習 4: Cosmos DB を使用するために既存のアプリケーションを再構成および実行する](#exercise-4-reconfigure-and-run-existing-applications-to-use-cosmos-db)
    - [演習 5: クリーンアップする](#exercise-5-clean-up)

<!-- /TOC -->
このラボでは、既存の MongoDB データベースを Cosmos DB に移行します。Azure Database Migration Service を使用します。代わりに、MongoDB データベースを使用して Cosmos DB データベースに接続する既存のアプリケーションを再構成する方法についても確認します。

このラボは、一連の IoT デバイスから温度データをキャプチャするシステム例に基づいています。温度は、タイムスタンプと共に MongoDB データベースに記録されます。各デバイスには一意の ID があります。これらのデバイスをシミュレートし、データベースにデータを保存する MongoDB アプリケーションを実行します。また、ユーザーが各デバイスに関する統計情報を照会できるようにする 2 つ目のアプリケーションも使用します。MongoDB から Cosmos DB にデータベースを移行した後、両方のアプリケーションを構成して Cosmos DB に接続し、それらが正しく機能することを確認します。

このラボは、Azure Cloud Shell と Azure portal を使用して実行されます。

## 演習 1: 設定

最初の演習では、あなたの会社が製造する温度デバイスからキャプチャしたデータを保持するための MongoDB データベースを作成します。

### タスク 1: リソース グループと Virtual Network を作成する

1. インターネット ブラウザで、https://portal.azure.com に移動してサインインします。
1. Azure portal で「**リソース グループ**」を選択し、「**+ 追加**」を選択します。
1. 「**リソース グループを作成する**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | mongodbrg |
    | リージョン | 最も近い場所を選択します |

1. 「**Review + create**」を選択し、「**作成**」を選択します。リソース グループが作成されるまで待ちます。
1. Azure portal のハンバーガー メニューで、「**+ リソースの作成**」をクリックします。
1. 「**新規**」ページの「**Marketplace を検索**」ボックスに、「**Virtual Network**」と入力し、Enter キーを押します。
1. 「**仮想ネットワーク**」ページで、「**作成**」を選択します。
1. 「**仮想マシンの作成**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | mongodbrg |
    | 名前 | databasevnet |
    | リージョン | リソース グループに指定したのと同じ場所を選択します |

1. 「**次へ:**」を選択します。**IP アドレス**
1. 「**IP アドレス**」ページの「**IPv4 アドレス空間**」の下で、「**10.0.0.0/16**」空間を選択し、右側の「**削除**」ボタンを選択します。
1. 「**IPv4 アドレス空間**」リストに、「**10.0.0.0/24**」と入力します。
1. 「**+ サブネットの追加**」を選択します。「**サブネットの追加**」ペインで、「**サブネット名**」を「**default**」に設定し、「**サブネット アドレス範囲**」を「**10.0.0.0/28**」に設定して、「**追加**」を選択します。
1. 「**IP アドレス**」ページで、「**次へ: セキュリティ**」を選択します。
1. 「**セキュリティ**」ページで、「**DDoS Protection Standard**」が「**無効**」に設定され、「**ファイアウォール**」が「**無効**」に設定されていることを確認します。「**Review + create**」を選択します。
1. 「**仮想ネットワークの作成**」ページで、「**作成**」を選択します。仮想ネットワークが作成されるのを待ってから、続行してください。

### タスク 2: MongoDB データベース サーバーを作成する

1. Azure portal のハンバーガー メニューで、「**+ リソースの作成**」をクリックします。
1. 「**Marketplace の検索**」ボックスに「**Ubuntu**」と入力し、Enter キーを押します。
1. 「**Marketplace**」ページで、「**UBUNTU Server 18.04 LTS**」を選択します。 
1. 「**Ubuntu Server 18.04 LTS**」ページで、「**作成**」を選択します。
1. 「**仮想マシンの作成**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | *\<your-subscription\>* |
    | リソース グループ | mongodbrg |
    | 仮想マシン名 | mongodbserver | 
    | リージョン | リソース グループに指定したのと同じ場所を選択します |
    | 可用性オプション | インフラストラクチャの冗長性は必要ありません |
    | 画像 | Ubuntu Server 18.04 LTS - Gen1 |
    | Azure Spot インスタンス | チェック解除 |
    | サイズ | Standard A1_v2 |
    | 認証の種類 | パスワード |
    | ユーザー名 | azureuser |
    | パスワード | Pa55w.rdPa55w.rd |
    | パスワードの確認 | Pa55w.rdPa55w.rd |
    | パブリック受信ポート | 選択したポートを許可する |
    | 受信ポートの選択 | SSH (22) |

1. 「**次へ: Disks \>**」を選択します。
1. 「**ディスク**」ページで、既定の設定をそのままにし、「**次へ: ネットワーク \>**」を選択します。
1. 「**ネットワーク**」ページで、以下の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | バーチャル ネットワーク | databasevnet |
    | サブネット | 既定値 (10.0.0.0/28) |
    | パブリック IP | (新規) mongodbserver-ip |
    | NIC ネットワーク セキュリティ グループ | 詳細 |
    | ネットワーク セキュリティ グループを構成する | (新規) mongodbserver-nsg |
    | 高速ネットワーク | チェック解除 |
    | 負荷分散 | チェック解除 |

1. 「**Review + create \>**」を選択します。
1. 検証ページで、「**作成**」を選択します。
1. 仮想マシンがデプロイされるまで待ってから続けます
1. Azure portal のハンバーガー メニューで、「**すべてのリソース**」を選択します。
1. 「**すべてのリソース**」ページで、「**mongodbserver-nsg**」を選択します。
1. 「**mongodbserver-nsg**」のページの「**設定**」で、「**受信セキュリティ規則**」を選択します。
1. 「**mongodbserver-nsg - 受信セキュリティ規則**」ページで、「**+ 追加**」を選択します。
1. 「**受信セキュリティ規則の追加**」ペインで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | ソース | Any |
    | 送信元ポートの範囲 | * |
    | 宛先 | Any |
    | 宛先ポート範囲 | 27017 |
    | プロトコル | Any |
    | アクション | 許可 |
    | 優先度 | 1030 |
    | 名前 | Mongodb-port |
    | 説明 | クライアントが MongoDB への接続に使用するポート |

1. 「**追加**」を選択します。

### タスク 3: MongoDB をインストールする

1. Azure portal のハンバーガー メニューで、「**すべてのリソース**」を選択します。
1. 「**すべてのリソース**」ページで、「**mongodbserver-ip**」を選択します。
1. 「**Mongodbserver-ip**」ページで、「**IP アドレス**」をメモしておきます。
1. Azure portal の上部にあるツール バーで、「**Cloud Shell**」を選択します。
1. 「**ストレージがマウントされていません**」というメッセージ ボックスが表示されたら、「**ストレージの作成**」を選択します。
1. Cloud Shell が起動したら、Cloud Shell ウィンドウの上にあるドロップダウン リストで、「**Bash**」を選択します。
1. Cloud Shell で、次のコマンドを入力して mongodbserver 仮想マシンに接続します。*\<ip address\>* を **mongodbserver-ip** IP アドレスの値に置き換えます。

    ```bash
    ssh azureuser@<ip address>
    ```

1. プロンプトで、「**yes**」と入力して接続を続行します。
1. パスワード「**Pa55w.rdPa55w.rd**」を入力します。
1. MongoDB パブリック GPG キーをインポートするために、次のコマンドを入力します。コマンドにより **OK** が返されるはずです。

    ```bash
    wget -qO - https://www.mongodb.org/static/pgp/server-4.0.asc | sudo apt-key add -
    ```

1. パッケージの一覧を作成するために、次のコマンドを入力します。

    ```bash
    echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
    ```

    コマンドにより `deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.0 multiverse` に類似するリターン テキストが返されるはずです。これは、パッケージ一覧が正常に作成されたことを示しています。

1. パッケージ データベースを再度読み込むには、次のコマンドを入力します。

    ```bash
    sudo apt-get update
    ```

1. MongoDB をインストールするには、次のコマンドを入力します。

    ```bash
    sudo apt-get install -y mongodb-org
    ```

    インストールは、パッケージのインストール、準備、および展開に関するメッセージと共に進める必要があります。インストールの完了には数分かかる場合があります。

### タスク 4: MongoDB データベースを構成する

既定では、Mongo DB インスタンスは、認証なしで実行するように構成されています。このタスクでは、ローカル ネットワーク インターフェイスにバインドするように MongoDB を構成して、他のコンピューターからの接続を受け入れるようにします。また、認証を有効にし、移行を実行するために必要なユーザー アカウントを作成します。最後に、テスト アプリケーションからデータベースに対してクエリを実行するために使用できるアカウントを追加します。

1. MongoDB 構成ファイルを開くには、次のコマンドを実行します。

    ```bash
    sudo nano /etc/mongod.conf
    ```

1. ファイルで **bindIp** 設定を見つけ、**0.0.0.0** に設定します。
1. 以下の設定を追加します。`セキュリティ` セクションが既に存在する場合は、このコードで置き換えます。それ以外の場合は、2 つの新しい行にコードを追加してください。

    ```bash
    security:
        authorization: 'enabled'
    ```

1. 構成ファイルを保存するには、<kbd>Esc</kbd> キーを押してから、<kbd>CTRL + X</kbd> キーを押します。変更したバッファーを保存するには、<kbd>y</kbd> キーを押してから E<kbd>Enter</kbd> キーを押します。
1. MongoDB サービスを再起動して変更を適用するには、次のコマンドを入力します。

    ```bash
    sudo service mongod restart
    ```

1. MongoDB サービスに接続するには、次のコマンドを入力します。

    ```bash
    mongo
    ```

1. **>** プロンプトで、**管理者**データベースに切り替えるには、次のコマンドを実行します。

    ```bash
    use admin;
    ```

1. **administrator** という名前の新しいユーザーを作成するには、次のコマンドを実行します。読みやすくするために、コマンドを 1 行または複数行にわたって入力できます。`mongo` プログラムがセミコロンに到達すると、コマンドが実行されます。

    ```bash
    db.createUser(
        {
            user: "administrator",
            pwd: "Pa55w.rd",
            roles: [
                { role: "userAdminAnyDatabase", db: "admin" },
                { role: "clusterMonitor", db:"admin" },
                "readWriteAnyDatabase"
            ]
        }
    );
    ```

1. `mongo` プログラムを終了するには、次のコマンドを入力します。

    ```bash
    exit;
    ```

1. 新しい管理者アカウントを使用して MongoDB に接続するには、次のコマンドを実行します。

    ```bash
    mongo -u "administrator" -p "Pa55w.rd"
    ```

1. **DeviceData** データベースに切り替えるには、次のコマンドを実行します。

    ```bash
    use DeviceData;    
    ```

1. アプリケーションでデータベースに接続するために使用される **deviceadmin** という名前のユーザーを作成するには、次のコマンドを実行します。

    ```bash
    db.createUser(
        {
            user: "deviceadmin",
            pwd: "Pa55w.rd",
            roles: [ { role: "readWrite", db: "DeviceData" } ]
        }
    );
    ```

1. `mongo` プログラムを終了するには、次のコマンドを入力します。

    ```bash
    exit;
    ```

1. 次のコマンドを実行し、mongodb サービスを再起動します。エラー メッセージなしでサービスが再起動されることを確認します。

    ```bash
    sudo service mongod restart
    ```

1. 次のコマンドを実行し、mongodb に deviceadmin ユーザーとしてログインできることを確認します。

    ```bash
    mongo -u "deviceadmin" -p "Pa55w.rd" --authenticationDatabase DeviceData
    ```

1. **>** プロンプトで、次のコマンドを実行して mongo シェルを終了します。

    ```bash
    exit;
    ```

1. bash プロンプトで、次のコマンドを実行して MongoDB サーバーから切断し、Cloud Shell に戻ります。

    ```bash
    exit
    ```

## 演習 2: クエリと MongoDB データベースを設定する

これで、MongoDB サーバーとデータベースが作成されました。次の手順では、このデータベースのデータを入力および照会できるサンプル アプリケーションを示します。

### タスク 1: アプリをビルドして実行し、MongoDB データベースを設定する

1. Azure Cloud Shell で、次のコマンドを実行して、このワークショップのサンプル コードをダウンロードします。

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

1. **migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceCapture** フォルダーに移動します。

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceDataCapture
    ```

1. **コード** エディターを使用して、**TemperatureDevice.cs** ファイルを調べます。

    ```bash
    code TemperatureDevice.cs
    ```

    このファイルのコードには、**TemperatureDevice** という名前のクラスが含まれています。このクラスは、データをキャプチャして MongoDB データベースに保存する温度デバイスをシミュレートします。.NET Framework 向けの MongoDB ライブラリを使用しています。**TemperatureDevice** コンストラクターは、アプリケーション構成ファイルに格納された設定を使用してデータベースに接続します。**RecordTemperatures** メソッドは、読み取り値を生成し、データベースに書き込みます。

1. コード エディターを閉じるために、<kbd>CTRL + Q</kbd> を押してから、**ThermometerReading.cs** ファイルを開きます。

   ```bash
   code ThermometerReading.cs
   ```

    このファイルには、アプリケーションによってデータベースに格納されるドキュメントの構造が表示されます。各ドキュメントには、次のフィールドが含まれています。

    - オブジェクト ID。これは、各ドキュメントを一意に識別するために MongoDB によって生成される "_id" フィールドです。
    - デバイス ID。各デバイスには、"Device" というプレフィックスの付いた番号が付いています。
    - デバイスによって記録された温度。
    - 温度が記録された日時。
  
1. コード エディターを閉じるために、<kbd>CTRL + Q</kbd> を押してから、**App.config** ファイルを開きます。

    ```bash
    code App.config
    ```

    このファイルには、MongoDB データベースに接続するための設定が含まれています。**アドレス** キーの値を、前に記録した MongoDB サーバーの IP アドレスに設定します。

1. ファイルを保存するために、<kbd>CTRL + S</kbd> を押し、コード エディターを閉じるために、<kbd>CTRL + Q</kbd> を押します
1. プロジェクト ファイルを開くには、次のコマンドを実行します。

    ```bash
    code MongoDeviceDataCapture.csproj
    ```

1. `<TargetFramework>` タグに値 `netcoreapp3.1` が含まれていることを確認します。必要に応じて、この値を置き換えてください。
1. ファイルを保存するために、<kbd>CTRL + S</kbd> を押し、コード エディターを閉じるために、<kbd>CTRL + Q</kbd> を押します
1. 次のコマンドを実行して、アプリケーションをリビルドします。

    ```bash
    dotnet build
    ```

1. アプリケーションを実行します。

    ```bash
    dotnet run
    ```

    アプリケーションでは、100 台のデバイスが同時に実行するようにシミュレートします。アプリケーションの実行を数分間許可してから、Enter キーを押してアプリケーションを停止します。

### タスク 2: 別のアプリをビルドして実行し、MongoDB データベースにクエリを実行する

1. **migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery** フォルダーに移動します。

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

    このフォルダーには、各デバイスによってキャプチャされたデータを分析するために使用できる別のアプリケーションが含まれています。

1. **コード** エディターを使用して、**Program.cs** ファイルを調べます。

    ```bash
    code Program.cs
    ```

    アプリケーションは (ファイルの一番下にある **ConnectToDatabase** メソッドを使用して) データベースに接続され、ユーザーはデバイス番号の入力が求められます。このアプリケーションでは、.NET Framework 向けの MongoDB ライブラリを使用して、指定されたデバイスの次の統計情報を計算する集計パイプラインを作成および実行します。

    - 記録された読み取り値の数。
    - 記録された平均平均気温。
    - 最低の読み取り値です。
    - 最高の読み取り値です。
    - 最新の読み取り値です。

1. コード エディターを閉じるために、<kbd>CTRL + Q</kbd> を押してから、**App.config** ファイルを開きます。

    ```bash
    code App.config
    ```

    以前と同様に**アドレス** キーの値を、前に記録した MongoDB サーバーの IP アドレスに設定します。

1. ファイルを保存するために、<kbd>CTRL + S</kbd> を押し、コード エディターを閉じるために、<kbd>CTRL + Q</kbd> を押します
1. プロジェクト ファイルを開くには、次のコマンドを実行します。

    ```bash
    code DeviceDataQuery.csproj
    ```

1. `<TargetFramework>` タグに値 `netcoreapp3.1` が含まれていることを確認します。必要に応じて、この値を置き換えてください。
1. ファイルを保存するために、<kbd>CTRL + S</kbd> を押し、コード エディターを閉じるために、<kbd>CTRL + Q</kbd> を押します

1. アプリケーションをビルドして実行します。

    ```bash
    dotnet build
    dotnet run
    ```

1. 「**デバイス番号の入力**」プロンプトで、0 から 99 の値を入力します。アプリケーションは、データベースをクエリし、統計情報を計算し、結果を表示します。<kbd>Q + Enter</kbd> を押して、アプリをを終了します。

## 演習 3: MongoDB データベースを Cosmos DB に移行する

次の手順では、MongoDB データベースを取得し、Cosmos DB に転送します。

### タスク 1: Cosmos アカウントとデータベースを作成する

1. Azure portal に戻ります。
1. ハンバーガー メニューで、「**+ リソースの作成**」 を選択します。
1. 「**新規**」ページの「**Marketplace を検索**」ボックスに「**Azure Cosmos DB**」と入力し、Enter キーを押します。
1. 「**Azure Cosmos DB**」ページで、「**作成**」を選択します。
1. 「**Azure Cosmos DB アカウントの作成**」ページで、次の設定を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | サブスクリプションを選択します |
    | リソース グループ | mongodbrg |
    | アカウント名 | mongodb*nnn*、*nnn* は選択された乱数 |
    | API | MongoDB 用の Azure Cosmos DB API |
    | ノートブック | オフ |
    | 場所 | MongoDB サーバーと仮想ネットワークに使用したのと同じ場所を指定します |
    | 容量モード | プロビジョニング スループット |
    | Free レベル割引の適用 | 適用 |
    | 勘定タイプ | 非運用 |
    | バージョン | 3.6 |
    | geo 冗長性 | Disable |
    | マルチ リージョン書き込み | Disable |
    | 可用性ゾーン | Disable |

1. 「**Review + create**」を選択する
1. 検証ページで、「**作成**」を選択し、Cosmos DB アカウントがデプロイされるのを待ちます。
1. Azure portal のハンバーガー メニューで、「**すべてのリソース**」を選択し、新しい Cosmos DB アカウント (**mongodb*nnn***) を選択します。
1. 「**mongodb*nnn***」ページで、「**データ エクスプローラー**」を選択します。
1. 「**データ エクスプローラー**」ペインで、「**新しいコレクション**」を選択します。
1. 「**コレクションの追加**」ペインで、次の設定を指定します。

    | プロパティ  | 値  |
    |---|---|
    | データベース ID | 「**新規作成**」を選択し、「**DeviceData**」と入力します |
    | データベース スループットをプロビジョニングする | オン |
    | スループット | 1000 |
    | コレクション ID | 温度 |
    | ストレージの容量 | 無制限 |
    | シャード キー | deviceID |
    | シャード キーが 100 バイトを超える | チェック解除 |
    | すべてのフィールドにワイルドカード インデックスを作成する | チェック解除 |
    | 分析ストア | オフ |

1. 「**OK**」を選択します。

### タスク 2: Database Migration Service を作成する

1. Azure portal のハンバーガー メニューで、「**すべてのサービス**」を選択します。
1. 「**すべてのサービス**」検索ボックスに、「**サブスクリプション**」と入力し、Enter キーを押します。
1. 「**サブスクリプション**」ページで、対象のサブスクリプションを選択します。
1. 対象の「サブスクリプション」ページの「**設定**」で、「**リソース プロバイダー**」を選択します。
1. 「**名前でフィルター**」ボックスに「**DataMigration**」と入力してから、「**Microsoft.DataMigration**」を選択します。
1. **ステータス**が**登録**されていない場合は、「**登録**」を選択し、「**ステータス**」が「**登録済み**」に変わるまで待ちます。状態の変化を確認するには、「**更新**」を選択する必要がある場合があります。
1. Azure portal のハンバーガー メニューで、「**+ リソースの作成**」をクリックします。
1. 「**新規**」ページの「**Marketplace の検索**」ボックスに「**Azure Database Migration Service**」と入力してから、Enter キーを押します。
1. 「**Azure Database Migration Service**」ページで、「**作成**」を選択します。
1. 「**移行サービスの作成**」ページで、次の設定を入力します。

    | プロパティ  | 値  |
    |---|---|
    | サブスクリプション | サブスクリプションを選択します |
    | リソース グループ | mongodbrg |
    | サービス名 | MongoDBMigration |
    | 場所 | 以前に使用したのと同じ場所を選択する |
    | サービス モード | Azure |
    | 価格レベル | Standard: 1 仮想コア |

1. 「**次へ: ネットワーク**」を選択します。
1. 「**ネットワーク**」ページで、「**databasevnet/default**」チェックボックスにチェックを入れてから、「**Review + create**」を選択します
1. 「**作成**」を選択し、サービスがデプロイされるまで待ってから続行します。この操作には数分かかります。

### タスク 3: 新しい移行プロジェクトを作成し実行する

1. Azure portal のハンバーガー メニューで、「**リソース グループ**」を選択します。
1. 「**リソース グループ**」ウィンドウで、「**mongodbrg**」を選択します。
1. 「**mongodbrg**」ウィンドウで、「**MongoDBMigration**」を選択します。
1. 「**MongoDBMigration**」ページで、「**+ 新しい移行プロジェクト**」を選択します。
1. 「**新しい移行プロジェクト**」ページで、次の設定を入力します。

    | プロパティ  | 値  |
    |---|---|
    | プロジェクト名 | MigrateTemperatureData |
    | ソース サーバーの種類 | MongoDB |
    | ターゲット サーバー名 | Cosmos DB (MongoDB API) |
    | アクティビティ タイプの選択 | オンライン データの移行 |

1. 「**アクティビティの作成と実行**」を選択します。
1. 「**移行ウィザード**」が起動したら、「**ソースの詳細**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | モード | 標準モード |
    | ソース サーバー名 | 前に記録した **mongodbserver-ip** IP アドレスの値を指定します |
    | サーバー ポート | 27017 |
    | ユーザー名 | administrator |
    | パスワード | Pa55w.rd |
    | SSL を要求 | チェック解除 |

1. 「**次へ: ターゲットの選択**」を選択します。
1. 「**ターゲットの選択**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | モード | Cosmos DB ターゲットを選択します |
    | サブスクリプション | サブスクリプションを選択します |
    | Cosmos DB 名を選択します | mongodb*nnn* |
    | 接続文字列 | Cosmos DB アカウント用に生成された接続文字列を受け入れます |

1. 「**次へ: データベースの設定**」を選択します。
1. 「**データベースの設定**」ページで、次の詳細を入力します。

    | プロパティ  | 値  |
    |---|---|
    | ソース データベース | DeviceData |
    | ターゲット データベース | DeviceData |
    | スループット (RU/s) | 1000 |
    | コレクションのクリーン アップ | チェック解除 |

1. 「**次へ: コレクション設定**」を選択します。
1. 「**コレクション設定**」ページで、DeviceData データベースのドロップダウンの矢印を選択し、次の情報を入力します。

    | プロパティ  | 値  |
    |---|---|
    | 名前 | 温度 |
    | ターゲット コレクション | 温度 |
    | スループット (RU/s) | 1000 |
    | シャード キー | deviceID |
    | 一意 | 空白のままにします |

1. 「**次へ: 移行の概要」を選択します。**
1. 「**移行の概要**」ページで、「**アクティビティ名**」フィールドに「**mongodb-migration**」と入力し、次に「**移行の開始**」を選択します。
1. 「**mongodb-migration**」ページで、移行が完了するまで 30 秒ごとに「**更新**」を選択します。処理されたドキュメントの数をメモします。

### タスク 4: 正常に移行されたことを検証する

1. Azure portal のハンバーガー メニューで、「**すべてのリソース**」を選択します。
1. **すべてのリソース** ページで、**mongodb*nnn*** を選択します。
1. 「**mongodb*nnn***」ページで、「**データ エクスプローラー**」を選択します。
1. 「**データ エクスプローラー**」ペインで、「**DeviceData**」データベースを展開し、「**Temperatures**」コレクションを展開して、「**ドキュメント**」を選択します。
1. **ドキュメント** ウィンドウで、ドキュメントの一覧をスクロールします。各ドキュメントのドキュメント ID (**_id**) とシャード キー (**/deviceID**) が表示されます。
1. 任意のドキュメントを選択します。表示されたドキュメントの詳細が表示されます。一般的なドキュメントは次のようになります。

    ```JSON
    {
	    "_id" : ObjectId("5ce8104bf56e8a04a2d0929a"),
	    "deviceID" : "Device 83",
	    "temperature" : 19.65268837271849,
	    "time" : 636943091952553500
    }
    ```

1. 「**ドキュメント エクスプローラー**」ペインのツールバーで、「**新しいシェル**」を選択します。
1. 「**Shell 1**」ペインの「**\>**」プロンプトで、次のコマンドを入力してから、Enter キーを押します。

    ```mongosh
    db.Temperatures.count()
    ```

    このコマンドは、Temperatures コレクション内のドキュメントの数を表示します。移行ウィザードによってレポートされる番号と一致する必要があります。

1. 次のコマンドを入力し、Enter キーを押します。

    ```mongosh
    db.Temperatures.find({deviceID: "Device 99"})
    ```

    このコマンドは、デバイス 99 のドキュメントを取り込んで表示します。

## 演習 4: Cosmos DB を使用するために既存のアプリケーションを再構成および実行する

最後の手順では、既存の MongoDB アプリケーションを再構成して Cosmos DB に接続し、それらが以前と同じように動作することを確認します。このプロセスでは、アプリケーションがデータベースに接続する方法を変更する必要がありますが、アプリケーションのロジックは変更しないでください。

1. 「**mongodb*nnn***」ペインの「**設定**」で、「**接続文字列**」を選択します。
1. 「**mongodb*nnn* 接続文字列**」ページで、次の設定をメモしておきます。

    - ホスト
    - ユーザー名
    - プライマリ パスワード
  
1. 「Cloud Shell」ウィンドウに戻り (セッションがタイムアウトした場合は再接続します)、**migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery** フォルダーに移動します。

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

1. コード エディターで App.config ファイルを開きます。

    ```bash
    code App.config
    ```

1. ファイルの「**MongoDB の設定**」セクションで、既存の設定をコメント アウトします。
1. 「**Cosmos DB Mongo API の設定**」セクションの設定のコメントを解除し、これらの設定の値を次のように設定します。

    | 設定  | 値  |
    |---|---|
    | 住所 | 「**mongodb*nnn* 接続文字列**」ページの「**ホスト**」 |
    | ユーザー名 | 「**mongodb*nnn* 接続文字列**」ページの「**ユーザー名**」 |
    | パスワード | 「**mongodb*nnn* 接続文字列**」ページの「**プライマリ パスワード**」 |

    完成したファイルは次のようになります。

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="Database" value="DeviceData" />
            <add key="Collection" value="Temperatures" />

            <!-- Settings for MongoDB -->
            <!--add key="Address" value="nn.nn.nn.nn" />
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

1. ファイルを保存して、コード エディターを閉じます。

1. コード エディターを使って、Program.cs ファイルを開きます。

    ```bash
    code Program.cs
    ```

1. 「**ConnectToDatabase**」メソッドまで下にスクロールします。
1. MongoDB に接続するための資格情報を設定する行をコメント アウトし、Cosmos DB に接続するための資格情報を指定するステートメントのコメントを解除します。コードは、以下のようになるはずです。

    ```C#
    // MongoDB データベースに接続する
    MongoClient client = new MongoClient(new MongoClientSettings
    {
        Server = new MongoServerAddress(address, port),
        ServerSelectionTimeout = TimeSpan.FromSeconds(10),

        //
        // MongoDB 向けの資格情報設定
        //

        // Credential = MongoCredential.CreateCredential(database, azureLogin.UserName, azureLogin.SecurePassword),

        //
        // CosmosDB Mongo API 向けの資格情報設定
        //

        UseTls = true,
        Credential = new MongoCredential("SCRAM-SHA-1", new MongoInternalIdentity(database, azureLogin.UserName), new PasswordEvidence(azureLogin.SecurePassword))

        // Mongo API 設定の終了 
    });

    ```

    これらの変更は、元の MongoDB データベースが TLS 接続を使用していなかったために必要です。Cosmos DB では常に TLS を使用します。

1. ファイルを保存して、コード エディターを閉じます。
1. アプリケーションを再構築して実行します。

    ```bash
    dotnet build
    dotnet run
    ```

1. **デバイス番号の入力**プロンプトで、0 から 99 のデバイス番号を入力します。アプリケーションは、今回のように Cosmos DB データベースに保持されているデータを使用する場合を除き、以前とまったく同じように実行する必要があります。
1. 他のデバイス番号を使用してアプリケーションをテストします。**Q** と入力して終了します。

MongoDB データベースを Cosmos DB に正常に移行し、既存の MongoDB アプリケーションを再構成して Cosmos DB データベースに接続しました。

## 演習 5: クリーンアップする

1. Azure portal に戻ります。
1. ハンバーガー メニューで、「**リソース グループ**」をクリックします。
1. 「**リソース グループ**」ウィンドウで、「**mongodbrg**」を選択します。
1. 「**リソース グループの削除**」を選択します。
1. 「**"mongodbrg" を削除しますか**」ページで、「**リソース グループ名を入力**」ボックスに「**mongodbrg**」と入力してから、「**削除**」をクリックします。

---
© 2020 Microsoft Corporation. All rights reserved.

このドキュメントのテキストは、「クリエイティブ・コモンズ表示 3.0 ライセンス」(https://creativecommons.org/licenses/by/3.0/legalcode)」で入手できます。このドキュメントに含まれるその他すべてのコンテンツ (商標、ロゴ、画像などを含むがこれに該当する) は、 クリエイティブ・コモンズ・ライセンス付与には**含まれません**。このドキュメントは、Microsoft 製品に含まれる知的財産に対するいかなる法的権利も提供するものではありません。お客様の社内での参照目的に限り、このドキュメントをコピーし使用することができます。

このドキュメントは "現状のまま" 提供されます。このドキュメントに記載された情報および見解は、URL やその他のインターネット Web サイトの参照も含め、予告なく変更する可能性があります。お客様は、その使用に関するリスクを負うものとします。一部の例は説明のみを目的としており、架空のものです。実際の団体とは一切関係ありません。Microsoft は、以下の情報に対して、明示または黙示を問わず、一切の保証を行わないものとします。
