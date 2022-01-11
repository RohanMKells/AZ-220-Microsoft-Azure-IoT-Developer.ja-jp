---
lab:
    title: 'ラボ 15: Azure IoT Hub を使用してデバイスをリモートで監視および制御する'
    module: 'モジュール 8: デバイス管理'
---

# Azure IoT Hub を使用してデバイスをリモートで監視および制御する

## ラボ シナリオ

Contoso は、受賞歴のあるチーズを誇りにしており、製造プロセス全体で完璧な温度と湿度を維持するように気を付けていますが、エージング プロセス中の状況には常に特別な注意を払ってきました。

近年、Contoso は環境センサーを使用して、ナチュラル チーズのエイジングのための洞窟の中の状態を記録し、そのデータを使用してほぼ完璧な環境を特定しています。最も成功した (賞に値する) 場所のデータによると、熟成チーズの理想的な温度はおよそ華氏 50 度 +/- 5 度 (摂氏 10 度 +/- 2.8 度) です。最大飽和度のパーセンテージで測定される理想的な湿度値は、約 85% +/- 10% です。

これらの理想的な温度と湿度の値は、ほとんどのタイプのチーズにも効果的です。ただし、特に硬いチーズや特に柔らかいチーズには、少々調整が必要です。また、チーズの外皮に望ましい条件など、特定の結果を得るためにはエージング プロセス中の重要な時期あるいは段階で環境を調整する必要があります。

Contoso は幸運にも、(特定の地域で) ほぼ一年中理想的な条件を自然に維持するチーズ洞窟を運営することができます。しかしこういった場所でも、エージングプロセス中の環境管理は重要です。また、自然の洞窟には多くの場合、異なる小部屋が多くあり、小部屋の環境はそれぞれに少しずつ違います。チーズの品種は、それぞれ特定の要件に合う小部屋 (ゾーン) に配置されます。環境条件を望ましい範囲内に保つために、温度と湿度の両方を制御する空気処理・調整システムが使われています。

現在、作業者は洞窟施設の各ゾーン内の環境条件を監視し、必要に応じて空気処理システムの設定を調整して、望ましい温度と湿度を維持しています。作業者は、4時間ごとに各ゾーンを訪問し、環境条件を確認することができます。昼は高温、夜は低温と温度が著しく変化する場所では、状況が望ましい制限の範囲外になる可能性があります。

Contoso から貯蔵庫の環境を制御制限の範囲内に保つ自動化システムの実装を任されました。

このラボでは、IoT デバイスを実装するチーズ貯蔵庫監視システムをプロトタイプとして作成します。各デバイスには温度および湿度センサーが装備されており、デバイスが配置されているゾーンの温度と湿度を制御する空気処理システムに接続されています。

### 簡略化されたラボ条件

テレメトリの出力頻度は、生産ソリューションにおいて重要な検討事項です。冷却装置の温度センサーは 1 分間に 1 回しか報告する必要がないのに対し、航空機の加速度センサーは毎秒 10 回報告する必要がある場合があります。テレメトリを送信する必要のある頻度は、現在の状況に依存する場合もあります。たとえば、チーズ貯蔵庫のシナリオにおいて夜に急速に温度が低下する傾向がある場合は、日没の 2 時間前からセンサーの読み取り頻度を高めると役立つことがあります。当然のことながら、テレメトリの頻度を変更する要件は予測可能なパターンの一部である必要はなく、IoT デバイス設定の変更を促すイベントは予測不能な可能性があります。

このラボをシンプルに進行するために、以下を前提とします。

* デバイスは数秒ごとに IoT Hub にテレメトリ (温度と湿度の値) を送信します。この頻度はチーズ貯蔵庫では非現実的ですが、15 分ごとではなく、もっと頻繁に変化を見る必要があるラボ環境には最適です。
* 空気処理システムは、次の 3 つの状態のいずれかになる送風機です：オン、オフ、エラー
  * 送風機はオフ状態に初期化されています。
  * IoTデバイス上でダイレクト メソッドによって、送風機への電力を制御 (オン/オフ) します。
  * デバイス ツインの必要なプロパティ値は、送風機の目的の状態を設定するために使用されます。必要なプロパティ値は、送風機/デバイスの規定の設定をオーバーライドします。
  * 温度は送風機をオン/オフにすることで制御できます (ファンをオンにすると温度が下がります)

このラボでのコーディングは、製品利用統計情報の送受信、ダイレクト メソッドの呼び出しと実行、デバイス ツイン プロパティの設定と読み取りの 3 つの部分に分かれています。

まず、製品利用統計情報を送信するデバイス用と、製品利用統計情報を受信する (クラウドで実行される) バックエンド サービス用の 2 つのアプリを作成します。

次のリソースが作成されます。

![ラボ 15 アーキテクチャ](media/LAB_AK_15-architecture.png)

## このラボでは

このラボでは、次のタスクを正常に達成します。

* ラボの前提条件が満たされていることを確認する (必要な Azure リソースがあること)

    * スクリプトは、必要に応じて IoT Hub を作成します。
    * スクリプトは、このラボに必要な新しいデバイス ID を作成します。

* シミュレートされたデバイス アプリを作成して、デバイス テレメトリを IoT Hub 送信する
* テレメトリをリッスンするバックエンド サービス アプリを作成する
* ダイレクト メソッドを実装し、IoT デバイスに設定を伝達する
* IoT デバイスのプロパティを管理するために、デバイス ツイン機能を実装する

## ラボの手順

### 演習 1: ラボの前提条件を確認する

このラボでは、次の Azure リソースが利用可能であることを前提としています。

| リソースの種類 | リソース名 |
| :-- | :-- |
| リソース グループ | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |
| IoT デバイス | sensor-th-0055 |

> **重要**: セットアップ スクリプトを実行して、必要なデバイスを作成します。

不足しているリソースと新しいデバイスを作成するには、演習 2 に進む前に、以下の手順に従って **lab15-setup.azcli** スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

**lab15-setup.azcli** スクリプトは、**Bash** シェル環境で実行するために記述されています。Azure Cloud Shell でこれを実行するのが、最も簡単な方法です。

> **注:** **sensor-th-0055** デバイスの接続文字列が必要です。このデバイスが Azure IoT Hub に登録されている場合は、Azure Cloud Shell で次のコマンドを実行して接続文字列を取得できます
>
> ```bash
> az iot hub device-identity connection-string show --hub-name iot-az220-training-{your-id} --device-id sensor-th-0055 -o tsv
> ```

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

    Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Cloud Shell が **Bash** を使用していることを確認します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash** であることを確認します。

1. Cloud Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします(右から 4番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    _ラボ 3: 開発環境の設定_:ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * すべてのファイル
      * ラボ
          * 15-Azure IoT Hub を使用してリモートによるデバイスを監視および制御する
            * 設定

    lab15-setup.azcli スクリプト ファイルは、ラボ 15 の設定フォルダー内にあります。

1. **lab15-setup.azcli** ファイルを選択し、「**開く**」 をクリックします。

    ファイルのアップロードが完了すると、通知が表示されます。

1. 正しいファイルが Azure Cloud Shell にアップロードされたことを確認するには、次のコマンドを入力します。

    ```bash
    ls
    ```

    `ls` コマンドを使用して、現在のディレクトリの内容を表示します。一覧にある lab15-setup.azcli ファイルを確認できるはずです。

1. セットアップ スクリプトを含むこのラボのディレクトリを作成し、そのディレクトリに移動するには、次の Bash コマンドを入力します。

    ```bash
    mkdir lab15
    mv lab15-setup.azcli lab15
    cd lab15
    ```

1. **lab15-setup.azcli** に実行権限があることを確認するには、次のコマンドを入力します。

    ```bash
    chmod +x lab15-setup.azcli
    ```

1. Cloud Shell ツールバーで、lab15-setup.azcli ファイルへのアクセスを有効にするには、「**エディターを開く**」 (右から 2 番目のボタン - **{ }**) をクリックします。

1. 「**ファイル**」 の一覧で、lab15 フォルダーを展開してスクリプト ファイルを開くには、「**lab15**」 をクリックし、「**lab15-setup.azcli**」 をクリックします。

    エディタは **lab15-setup.azcli** ファイルの内容を表示します。

1. エディターで、割り当て済みの値 `{your-id}` と `{your-location}` を更新します。

    サンプル例として、このコースの最初に作成した一意の id 、つまり **cah191211** に `{your-id}` を設定し、リソースにとって意味のある場所に `{your-location}` を設定する必要があります。

    ```bash
    #!/bin/bash

    # これらの値を変更してください!
    YourID="{your-id}"
    Location="{your-location}"
    ```

    > **注**:  `{your-location}` 変数は、すべてのリソースをデプロイするリージョンの短い名前に設定する必要があります。次のコマンドを入力すると、使用可能な場所と短い名前 (「**名前**」 の列) の一覧を表示できます。

    ```bash
    az account list-locations -o Table

    DisplayName           Latitude    Longitude    Name
    --------------------  ----------  -----------  ------------------
    East Asia             22.267      114.188      eastasia
    Southeast Asia        1.283       103.833      southeastasia
    Central US            41.5908     -93.6208     centralus
    East US               37.3719     -79.8164     eastus
    East US 2             36.6681     -78.3889     eastus2
    ```

1. エディター画面の右上で、ファイルに加えた変更を保存してエディターを閉じるには、**...** をクリックし、「**エディターを閉じる**」 をクリックします。

    保存を求められたら、「**保存**」 をクリックすると、エディタが閉じます。

    > **注**: **CTRL+S** を使っていつでも保存でき、**CTRL+Q** を押してエディターを閉じます。

1. このラボに必要なリソースを作成するには、次のコマンドを入力します。

    ```bash
    ./lab15-setup.azcli
    ```

    このスクリプトの実行には数分かかります。各ステップが完了すると、出力が表示されます。

    このスクリプトは、まず **rg-az220** という名前のリソース グループ と **iot-az220-training-{your-id}** という名前の IoT Hub を作成します。既に存在する場合は、対応するメッセージが表示されます。次にスクリプトは、**sensor-th-0055** の ID を持つデバイスを IoT Hub に追加し、デバイスの接続文字列を表示します。

1. スクリプトが完了すると、IoT Hub とデバイスに関する情報が表示されます。

    スクリプトは、以下のような情報を表示します。

    ```text
    Configuration Data:
    ------------------------------------------------
    iot-az220-training-{your-id} Service connectionstring:
    HostName=iot-az220-training-{your-id}.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=nV9WdF3Xk0jYY2Da/pz2i63/3lSeu9tkW831J4aKV2o=

    sensor-th-0055 device connection string:
    HostName=iot-az220-training-{your-id}.azure-devices.net;DeviceId=sensor-th-0055;SharedAccessKey=TzAzgTYbEkLW4nWo51jtgvlKK7CUaAV+YBrc0qj9rD8=

    iot-az220-training-{your-id} eventhub endpoint:
    sb://iothub-ns-iot-az220-training-2610348-5a463f1b56.servicebus.windows.net/

    iot-az220-training-{your-id} eventhub path:
    iot-az220-training-{your-id}

    iot-az220-training-{your-id} eventhub SaS primarykey:
    tGEwDqI+kWoZroH6lKuIFOI7XqyetQHf7xmoSf1t+zQ=
    ```

1. スクリプトが表示する出力をテキスト ドキュメントにコピーして、このラボで後ほど使用します。

    情報を簡単に見つけることができる場所に保存したら、ラボを続ける準備が整います。

### 演習 2: テレメトリを送受信するコードを記述する

この演習では、IoT Hub にテレメトリを送信するシミュレートされたデバイス アプリ (sensor-th-0055 デバイス用) を作成します。

#### タスク 1: テレメトリを生成するシミュレートされたデバイスを開く

1. **Visual Studio Code** を開きます。

1. **「ファイル」** メニューで、**「フォルダを開く」** を選択します。

1. 「フォルダーを開く」 ダイアログで、ラボ 15 のスターター フォルダーに移動します。

    _ラボ 3: 開発環境の設定_: ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * すべてのファイル
        * ラボ
            * 15-Azure IoT Hub を使用してリモートによるデバイスを監視および制御する
                * スターター
                    * cheesecavedevice
                    * CheeseCaveOperator

1. 「**cheesecavedevice**」 をクリックし、「**フォルダーの選択**」 をクリックします。

    Visual Studio Code のエクスプローラー ウィンドウに次のファイルが一覧表示されます。

    * cheesecavedevice.csproj
    * Program.cs

1. コード ファイルを開くには、「**Program.cs**」 をクリックします。

    ざっと見ると、このアプリケーションは、前のラボで作業したシミュレートされたデバイス アプリケーションと非常に似ていることがわかります。このバージョンは、対称キー認証を使用し、テレメトリとログ メッセージの両方を IoT Hub に送信し、より複雑なセンサー実装を備えています。

1. 「**ターミナル**」 メニューで、「**新しいターミナル**」 をクリックします。

    コマンド プロンプトの一部として表示されたディレクトリ パスに注目してください。以前のラボ プロジェクトのフォルダー構造内で、このプロジェクトのビルドを開始したくありません。

1. ターミナル コマンド プロンプトで、アプリケーションのビルドを確認するには、次のコマンドを入力します。

    ```bash
    dotnet build
    ```

    次のように出力されます。

    ```text
    > dotnet build
    Microsoft (R) Build Engine version 16.5.0+d4cbfca49 for .NET Core
    Copyright (C) Microsoft Corporation. All rights reserved.

    Restore completed in 39.27 ms for D:\Az220-Code\AllFiles\Labs\15-Remotely monitor and control devices with Azure IoT Hub\Starter\CheeseCaveDevice\CheeseCaveDevice.csproj.
    CheeseCaveDevice -> D:\Az220-Code\AllFiles\Labs\15-Remotely monitor and control devices with Azure IoT Hub\Starter\CheeseCaveDevice\bin\Debug\netcoreapp3.1\CheeseCaveDevice.dll

    Build succeeded.
        0 Warning(s)
        0 Error(s)

    Time Elapsed 00:00:01.16
    ```

次のタスクでは、接続文字列を構成し、アプリケーションを確認します。

#### タスク 2: 接続とレビュー コードを構成する

このタスクで構築するシミュレートされたデバイス アプリは、温度と湿度を監視する IoT デバイスをシミュレートします。アプリは、センサーの読み取り値をシミュレートし、2 秒ごとにセンサー データを通信します。

1. **Visual Studio Code** で、Program.cs ファイル開かれていることを確認します。

1. コード エディターに、次のコード行を見つけます。

    ```csharp
    private readonly static string deviceConnectionString = "<your device connection string>";
    ```

1. **\<your device connection string\>** を以前に保存したデバイス接続文字列に置き換えます。

    これは、テレメトリを IoT Hub に送信する前に実装する必要がある唯一の変更です。

1. 「**ファイル**」 メニューの 「**上書き保存**」 をクリックします。

1. アプリケーションの構造を確認してください。

    アプリケーションの構造は、以前のラボで使用されていたものと似ていることに注意してください。

    * ステートメントの使用
    * 名前空間の定義
      * プログラム クラス - Azure IoT への接続とテレメトリの送信を担当
      * CheeseCaveSimulator クラス - (EnvironmentSensor を置き換えます) テレメトリを生成するだけでなく、このクラスは、冷却ファンの動作によって影響を受ける実行中のチーズ セラー環境もシミュレートします。
      * ConsoleHelper - コンソールへの異なる色のテキストの書き込みをカプセル化するクラス

1. **Main** メソッドを確認する:

    ```csharp
    private static void Main(string[] args)
    {
        ConsoleHelper.WriteColorMessage("Cheese Cave device app.\n", ConsoleColor.Yellow);

        // MQTT プロトコルを使用して IoT Hub に接続します。
        deviceClient = DeviceClient.CreateFromConnectionString(deviceConnectionString, TransportType.Mqtt);

        // チーズ セラー シミュレーターのインスタンスを作成します
        cheeseCave = new CheeseCaveSimulator();

        // 以下にレジスタ直接メソッド コードを挿入します

        // 以下に希望のレジスタ プロパティ変更ハンドラ コードを挿入します

        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();
    }
    ```

    以前のラボと同様に、**Main** メソッドを使用して IoT Hub への接続を確立します。デバイス ツインのプロパティの変更を統合するために使用されることに気付いたかもしれません。この場合、ダイレクト メソッドも統合します。

1. **SendDeviceToCloudMessagesAsync** メソッドを簡単に確認してください。

    以前のラボで作成した以前のバージョンと非常に似ていることに注意してください。

1. **CheeseCaveSimulator** クラスを簡単に確認してください。

   これは、以前のラボで使用されていた **EnvironmentSensor** クラスを進化させたものです。主な違いは、ファンの導入です。ファンが**オン**の場合、温度と湿度は徐々に目的の値に向かって移動しますが、ファンが**オフ** (または**故障**) の場合、温度と湿度の値は周囲の値に向かって移動します。興味深いのは、温度が読み取られたときにファンが**故障**状態に設定される可能性が 1% あるという事実です。

#### タスク 3: テレメトリを送信するコードをテストする

1. Visual Studio Code で、ターミナルが開いたままであることを確認します。

1. ターミナル コマンド プロンプトで、シミュレートされたデバイス アプリを実行するには、次のコマンドを入力します。

    ```bash
    dotnet run
    ```

   このコマンドは、 現在のフォルダー内の **Program.cs** ファイルを実行します。

1. 出力がターミナルに送信されていることに注意してください。

    すぐに次のようなコンソール出力が表示されます。

    ![コンソール出力](media/LAB_AK_15-cheesecave-telemetry.png)

    > **注**:  緑のテキストは、物事が正常に機能していることを示すために使用されます。赤いテキストは、問題が発生したことを示すために使用されます。上の画像のような画面が表示されない場合は、まず、デバイスの接続文字列を確認してください。

1. このアプリを実行したままにします。

    このラボの後半で、IoT Hub にテレメトリを送信する必要があります。

### 演習 3: テレメトリを受信する 2 つ目のアプリを作成する

(シミュレートされた) チーズ セラー デバイスが IoT Hub にテレメトリを送信するようになったので、IoT Hub に接続してそのテレメトリを 「リッスン」 できるバックエンド アプリを作成する必要があります。最終的には、このバックエンド アプリは、チーズ ケーブの温度の制御を自動化するために使用されます。

#### タスク 1: テレメトリを受信するアプリを作成する

このタスクでは、IoT Hub イベント ハブのエンドポイントからテレメトリを受信するために使用するバックエンド アプリの作業を追加します。

1. Visual Studio Code の新しいインスタンスを開きます。

    シミュレートされたデバイス アプリは、既に開いている Visual Studio Code ウィンドウで実行されているため、バックエンド アプリの Visual Studio Code の新しいインスタンスが必要です。

1. **「ファイル」** メニューで、**「フォルダを開く」** を選択します。

1. 「**フォルダーを開く**」 ダイアログで、ラボ 15 のスターター フォルダーに移動します。

1. 「**CheeseCaveOperator**」 をクリックし、「**フォルダーの選択**」 をクリックします。

    用意されている CheeseCaveOperator アプリケーションは、いくつかの NuGet パッケージ ライブラリと使用されるいくつかのコメントを含む単純なコンソール アプリケーションであり、コードを構築するプロセスをガイドします。アプリケーションを実行する前に、プロジェクトにコードブ ロックを追加する必要があります。

1. **「エクスプローラー」** ペインで、アプリケーション プロジェクト ファイルを開くには、「**CheeseCaveOperator.csproj**」 をクリックします。

    これで、**CheeseCaveOperator.cs** ファイルがコード エディター ペインで開かれるはずです。

1. 時間を割いて **CaveDevice.csproj** ファイルの内容を確認します。

    ファイルの内容は次のようになります。

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>netcoreapp3.1</TargetFramework>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.Azure.Devices" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.EventHubs" Version="4.*" />
    </ItemGroup>

    </Project>
    ```

    > **注**: ファイル内のパッケージのバージョン番号は、上記の番号よりも遅い場合は、問題ありません。

    プロジェクト ファイル (.csproj) は、作業中のプロジェクトのタイプを指定する XML ドキュメントです。この場合、プロジェクトは **Sdk** スタイルのプロジェクトです。

    ご覧のとおり、プロジェクト定義には、**PropertyGroup** と **ItemGroup** の 2 つのセクションが含まれています。

    **PropertyGroup** は、このプロジェクトを構築することで生成される出力のタイプを定義します。この場合、.NET Core3.1 を対象とする実行可能ファイルを作成します。

    **ItemGroup** は、アプリケーションに必要な外部ライブラリを指定します。これらの特定の参照は NuGet パッケージ用であり、各パッケージ参照はパッケージ名とバージョンを指定します。

    > **注**: コマンド プロンプト (Visual Studio Code Terminal コマンド プロンプトなど) でコマンド `dotnet add package` を入力することにより、NuGet ライブラリ (上記の ItemGroup にリストされているものなど) をプロジェクト ファイルに手動で追加できます。`dotnet restore` コマンドを入力すると、すべての依存関係が確実にダウンロードされます。たとえば、上記のライブラリをロードし、それらがコード プロジェクトで使用可能であることを確認するには、次のコマンドを入力できます。
    >
    >   dotnet add package Microsoft.Azure.EventHubs
    >   dotnet add package Microsoft.Azure.Devices
    >   dotnet restore
    >
    > **情報**: NuGet について詳しくは、[こちら](https://docs.microsoft.com/ja-jp/nuget/what-is-nuget)をご覧ください。

#### タスク 3: テレメトリ レシーバー コードを追加する

1. **「エクスプローラー」** ペインで、**「Program.cs」** をクリックします。

    **Program.cs** ファイルは次のようになります。

    ```csharp
    // Copyright (c) Microsoft. All rights reserved.
    // MITライセンスの下でライセンスされています。ライセンス情報の全容については、プロジェクト ルートのライセンス ファイルをご覧ください。

    // 以下のステートメントを使用して INSERT を実行します

    namespace CheeseCaveOperator
    {
        class Program
        {
            // ここに変数を挿入します

            // 以下に Main メソッドを挿入します

            // 以下に ReceiveMessagesFromDeviceAsync メソッドを挿入します

            // 以下に InvokeMethod メソッドを挿入します

            // 以下にデバイス ツイン セクションを挿入します
        }

        internal static class ConsoleHelper
        {
            internal static void WriteColorMessage(string text, ConsoleColor clr)
            {
                Console.ForegroundColor = clr;
                Console.WriteLine(text);
                Console.ResetColor();
            }
            internal static void WriteGreenMessage(string text)
            {
                WriteColorMessage(text, ConsoleColor.Green);
            }

            internal static void WriteRedMessage(string text)
            {
                WriteColorMessage(text, ConsoleColor.Red);
            }
        }
    }
    ```

    このコードは、オペレーター アプリの構造の概要を示しています。

1. `// INSERT using statements below here` コメントを見つけます。

1. アプリケーション コードが使用する名前空間を指定するには、次のコードを入力します。

    ```csharp
    using System;
    using System.Threading.Tasks;
    using System.Text;
    using System.Collections.Generic;
    using System.Linq;

    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.Devices;
    using Newtonsoft.Json;
    ```

    **System** を指定するだけでなく、文字列をエンコードするための **System.Text**、非同期タスクのための **System.Threading.Tasks**、前に追加した 2 つのパッケージの名前空間など、コードが使用する他の名前空間も宣言していることに注意してください。

    > **ヒント**: コードを挿入する場合、コード レイアウトが理想的でない場合があります。コード エディター ペインを右クリックし、**「ドキュメントのフォーマット」** をクリックすると、Visual Studio Code でドキュメントをフォーマットできます。**「タスク」** ペインを開いて (**F1** を押す)、**「ドキュメントのフォーマット」** と入力してから **Enter** キーを押すと、同じ結果を得ることができます。また、Windows では、このタスクのショートカットは **SHIFT+ALT+F** です。

1. `// INSERT variables below here` コメントを見つけます。

1. プログラムが使用している変数を指定するには、次のコードを入力します。

    ```csharp
    // グローバル変数。
    // イベント ハブ互換エンドポイント。
    private readonly static string eventHubsCompatibleEndpoint = "<your event hub endpoint>";

    // イベント ハブと互換性のある名前。
    private readonly static string eventHubsCompatiblePath = "<your event hub path>";
    private readonly static string iotHubSasKey = "<your event hub SaS key>";
    private readonly static string iotHubSasKeyName = "service";
    private static EventHubClient eventHubClient;

    // 以下にサービス クライアント変数を挿入します

    // 以下にレジストリ マネージャー変数を挿入します

    // IoT Hub の接続文字列。
    private readonly static string serviceConnectionString = "<your service connection string>";

    private readonly static string deviceId = "sensor-th-0055";
    ```

1. 入力したコード (およびコード コメント) を確認してください。

    **eventHubsCompatibleEndpoint** 変数は、イベント ハブと互換性のある IoT Hub 組み込みサービス向けエンド ポイント (メッセージ/イベント) の URI を格納するために使用されます

    **eventHubsCompatiblePath** 変数には、Even tHub エンティティへのパスが含まれます。

    **iotHubSasKey** 変数には、名前空間またはエンティティに対応する共有アクセス ポリシー ルールのキー名が含まれます。

    **iotHubSasKeyName** 変数には、名前空間またはエンティティに対応する共有アクセス ポリシー ルールのキーが含まれます。

    **eventHubClient** 変数には、IoT Hub からメッセージを受信するために使用されるイベント ハブ クライアント インスタンスが含まれます。

    **serviceClient** 変数には、アプリから IoT Hub に (そしてそこからターゲット デバイスなどに) メッセージを送信するために使用されるサービス クライアント インスタンスが含まれます。

    **serviceConnectionString** 変数には、オペレーター アプリが IoT Hub に接続できるようにする接続文字列が含まれます。

    **deviceId** 変数には、**CheeseCaveDevice** アプリケーションで使用されるデバイス ID が含まれます。

1. サービス接続文字列の割り当てに使用するコード行を見つける

    ```csharp
    private readonly static string serviceConnectionString = "<your service connection string>";
    ```

1. **\<your service connection string\>** を、このラボで以前に保存した IoT Hub サービス接続文字列に置き換えます。

    演習 1 で実行したラボ 5- setup.azcli セットアップ スクリプトによって生成された iothubowner 共有アクセス ポリシーのプライマリ接続文字列を保存しておく必要があります。

    > **注**: **サービス**共有ポリシーではなく、**iothubowner** 共有ポリシーがなぜ使用されているのか、不思議に思われるかもしれません。答えは、各ポリシーに割り当てられた IoT Hub のアクセス許可に関連しています。**サービス**ポリシーには **ServiceConnect** アクセス許可があり、通常はバックエンド クラウド サービスによって使用されます。これは、次の権利を付与します。
    >
    > * クラウド サービス向けの通信エンドポイントと監視エンドポイントへのアクセスを許可します。
    > * デバイスからクラウドへのメッセージの受信、クラウドからデバイスへのメッセージの送信、対応する配信確認メッセージの取得のアクセス許可を付与します。
    > * ファイル アップロードの配信確認メッセージの取得のアクセス許可を付与します。
    > * タグおよび必要なプロパティを更新するためのツインへのアクセス、報告されるプロパティの取得、クエリの実行のアクセス許可を付与します。
    >
    > ラボの最初の部分では、**serviceoperator** アプリケーションがファンの状態を切り替えるダイレクト メソッドを呼び出しており、 **サービス**ポリシーには十分な権限があります。ただし、ラボの後半では、デバイス レジストリが照会されます。これは **RegistryManager** クラスを通じて行われます。**RegistryManager** クラスを使用してデバイス レジストリをクエリするには、IoT Hub への接続に使用される共有アクセス ポリシーに **レジストリ読み取り**アクセス許可が必要です。これは、次の権限を付与します。
    >
    > * ID レジストリへの読み取りアクセスを許可します。
    >
    > **iothubowner** ポリシーには**レジストリの書き込み**アクセス許可が与えられているため、**レジストリの読み取り**権限が継承されるため、ニーズに適しています。
    >
    > 運用シナリオでは、 **サービス接続**と**レジストリ読み取り**アクセス許可のみを持つ新しい共有アクセス ポリシーを追加することを検討してください。

1. **\<your event hub endpoint\>**、**\<your event hub path\>**、および **\<your event hub SaS key\>** を、このラボで前に保存した値に置き換えます。

1. `// INSERT Main method below here` コメントを見つけます。

1. **Main** メソッドを実装するには、次のコードを入力します。

    ```csharp
    public static void Main(string[] args)
    {
        ConsoleHelper.WriteColorMessage("Cheese Cave Operator\n", ConsoleColor.Yellow);

        // IoT Hub イベント ハブと互換性のあるエンドポイントに接続する EventHubClient インスタンスを作成します。
        var connectionString = new EventHubsConnectionStringBuilder(new Uri(eventHubsCompatibleEndpoint), eventHubsCompatiblePath, iotHubSasKeyName, iotHubSasKey);
        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString.ToString());

        // ハブの各パーティションに対して PartitionReceiver を作成します。
        var runtimeInfo = eventHubClient.GetRuntimeInformationAsync().GetAwaiter().GetResult();
        var d2cPartitions = runtimeInfo.PartitionIds;

        // 以下に希望のレジスタ プロパティ変更ハンドラ コードを挿入します

        // 以下にサービス クライアント インスタンス変数を挿入します

        // メッセージをリッスンする受信者を作成します。
        var tasks = new List<Task>();
        foreach (string partition in d2cPartitions)
        {
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition));
        }

        // すべての PartitionReceivers が完了するのを待ちます。
        Task.WaitAll(tasks.ToArray());
    }
    ```

1. 入力したコード (およびコード コメント) を確認してください。

    **EventHubsConnectionStringBuilder** クラスを使用して **EventHubClient** 接続文字列を構築していることに注意してください。これは事実上、さまざまな値を正しい形式に連結するヘルパークラスです。次に、これを使用してイベント ハブ エンド ポイントに接続し、**eventHubClient** 変数にデータを入力します。

    次に、**eventHubClient** を使用して、イベントハブの実行時情報を取得します。この情報には次のものが含まれます。

    * **CreatedAt** - イベントハブが作成された日時
    * **PartitionCount** - パーティションの数 (ほとんどの IoT Hub は 4 つのパーティションで構成されています） 
    * **PartitionIds** - パーティション ID を含む文字列配列
    * **Path** - イベント ハブ エンティティ パス

    パーティション ID の配列は、**d2cPartitions** 変数に格納されます。この変数は、各パーティションからメッセージを受信するタスクのリストを作成するためにまもなく使用されます。

    > **情報**: 共パーティションの目的の詳細については、[こちら](https://docs.microsoft.com/ja-jp/azure/iot-hub/iot-hub-scaling#partitions)を参照してください。

    デバイスから IoT Hub に送信されるメッセージは任意のパーティションで処理される可能性があるため、アプリは各パーティションからメッセージを取得する必要があります。コードの次のセクションでは、非同期タスクのリストを作成します。各タスクは特定のパーティションからメッセージを受信します。最後の行は、すべてのタスクが完了するのを待ちます。各タスクは無限ループになるため、この行はアプリケーションが終了するのを防ぎます。

1. `INSERT ReceiveMessagesFromDeviceAsync method below here` コメントを見つけます。

1. **ReceiveMessagesFromDeviceAsync** メソッドを実装するには、次のコードを入力します。

    ```csharp
    // パーティションの PartitionReceiver を非同期的に作成し、シミュレートされたクライアントから送信されたメッセージの読み取りを開始します。
    private static async Task ReceiveMessagesFromDeviceAsync(string partition)
    {
        // 既定のコンシューマー グループを使用して受信側を作成します。
        var eventHubReceiver = eventHubClient.CreateReceiver("$Default", partition, EventPosition.FromEnqueuedTime(DateTime.Now));
        Console.WriteLine("Created receiver on partition: " + partition);

        while (true)
        {
            // EventData を確認する - このメソッドは、取得するものがない場合にタイムアウトします。
            var events = await eventHubReceiver.ReceiveAsync(100);

            // バッチにデータがある場合は、処理します。
            if (events == null) continue;

            foreach (EventData eventData in events)
            {
                string data = Encoding.UTF8.GetString(eventData.Body.Array);

                ConsoleHelper.WriteGreenMessage("Telemetry received: " + data);

                foreach (var prop in eventData.Properties)
                {
                    if (prop.Value.ToString() == "true")
                    {
                        ConsoleHelper.WriteRedMessage(prop.Key);
                    }
                }
                Console.WriteLine();
            }
        }
    }
    ```

    ご覧のとおり、このメソッドには、ターゲット パーティションを定義する引数が付属しています。4 つのパーティションが指定されている既定の構成の場合、このメソッドは 4 回呼び出され、それぞれが非同期で並列に実行され、パーティションごとに 1 つ実行されることを思い出してください。

    このメソッドの最初の部分は、イベント ハブ レシーバーを作成します。このコードは、$Default コンシューマー グループ (カスタム コンシューマー グループを作成するのが一般的です) が、パーティション、そして最後にイベント パーティションのデータのどの位置から受信を開始するかを指定します。この場合、受信者は現在の時刻以降にキューに入れられたメッセージのみに関心があります。データ ストリームの開始、データ ストリームの終了、または特定のオフセットを提供できるようにする他のオプションがあります。

    > **情報**: コンシューマー グループの詳細については、[こちら](https://docs.microsoft.com/ja-jp/azure/event-hubs/event-hubs-features#consumer-groups)をご覧ください。

    レシーバーが作成されると、アプリは無限ループに入り、イベントの受信を待ちます。

    > **注**: `eventHubReceiver.ReceiveAsync(100)` コードは、一度に受信できるイベントの最大数を指定しますが、その数を待つことはありません。少なくとも 1 つが利用可能になるとすぐに戻ります。(タイムアウトのために) イベントが返されない場合、ループは続行され、コードはさらにイベントを待機します。

    1 つ以上のイベントが受信されると、各イベント データ本体がバイト配列から文字列に変換され、コンソールに書き込まれます。次に、イベントデータのプロパティが繰り返され、この場合、値が true であるかどうかが確認されます。現在のシナリオでは、これはアラートを表します。アラートが見つかると、コンソールに書き込まれます。

1. 「**ファイル**」 メニューで 変更を Program.cs ファイルに保存するには、「**保存**」 をクリックします。

#### タスク 3: テレメトリを受信するコードをテストする

このテストは重要であり、シミュレートされたデバイスによって送信されたテレメトリがバックエンド アプリによって取得されているかどうかを確認します。デバイス アプリはまだ実行されており、テレメトリが送信されていることを思い出してください。

1. ターミナルで **CheeseCaveOperator** バックエンド アプリを実行するには、ターミナル ウィンドウを開き、次のコマンドを入力します。

    ```bash
    dotnet run
    ```

   このコマンドは、 現在のフォルダー内の **Program.cs** ファイルを実行します。

   > **注**:  未使用の変数 **serviceConnectionString** - に関する警告は無視してかまいません。まもなく、その変数を使用するためのコードを追加します。

1. ターミナルへの出力をしばらく観察してください。

    コンソールの出力がすぐに表示されるはずで、IoT Hub にHub に正常に接続された場合、アプリがテレメトリ メッセージ データをほぼ即座に表示します。

    それ以外の場合は、IoT Hub サービスの接続文字列を慎重に確認し、文字列がサービス接続文字列であり、他の接続文字列ではないことを確認します。

    ![コンソール出力](media/LAB_AK_15-cheesecave-telemetry-received.png)

    > **注**:  緑色のテキストは、物事が本来のように機能していることを示すために使用され、赤いテキストは悪いことが起こっているときに表示されます。この画像のような画面が表示されない場合は、まず、デバイスの接続文字列を確認してください。

1. このアプリをしばらく実行したままにします。

1. 両方のアプリを実行した状態で、Operator アプリによって表示されるテレメトリが Device アプリによって送信されるテレメトリと同期していることを確認します。

    送信されているテレメトリと受信されているテレメトリを視覚的に比較します。

    * 完全に一致するデータはありますか?
    * データが送信された時点から受信されるまでの遅延は多いですか。

1. テレメトリ データを確認したら、実行中のアプリを停止し、VS Code の両方のインスタンスでターミナル ペインを閉じますが、Visual Studio Code ウィンドウは閉じないでください。

これで、デバイスからテレメトリを送信するアプリと、データの受信を確認するバックエンド アプリが作成されました。次の演習では、制御側を処理するステップ (データに問題が発生した場合の対処方法) の作業を開始します。

### 演習 4: ダイレクト メソッドを呼び出すコードを記述する

ダイレクト メソッドを呼び出すバックエンド アプリからの呼び出しには、ペイロードの一部として複数のパラメーターを含めることができます。ダイレクト メソッドは、通常、デバイスの機能をオンまたはオフにしたり、デバイスの設定を指定したりするために使用されます。

Contoso のシナリオでは、チーズ セラーにあるファンの動作を制御するデバイスに直接法を実装します (ファンをオンまたはオフにすることで温度と湿度の制御をシミュレートします)。Operator アプリケーションは、IoT Hub と通信して、デバイスでダイレクト メソッドを呼び出します。

チーズ セラー デバイスが、ダイレクト メソッドを実行するための指示を受け取ったときにチェックする必要があるいくつかのエラー状態があります。これらのチェックの 1 つは簡単で、ファンが障害状態の場合はエラーで応答します。報告するもう 1 つのエラー状況は、無効なパラメーターを受け取ったときです。デバイスは遠隔地にある可能性があるので、明確なエラー報告が重要です。

ダイレクト メソッドを使用するには、バックエンド アプリでパラメーターを準備してから、メソッドを呼び出す 1 つのデバイスを指定して呼び出しを行う必要があります。バックエンド アプリは応答を待機し、それを報告します。

デバイス アプリには、ダイレクト メソッドの機能コードが含まれます。関数名は、デバイスに対する IoT クライアントに登録されます。このプロセスにより、クライアントで、IoT Hub から呼び出しが行われたときに、実行する関数が確実に認識されます (多くのダイレクト メソッドが存在する可能性があります)。

この演習では、チーズ ケーブでファンをオンにするシミュレーションを行うダイレクト メソッドのコードを追加して、デバイス アプリを更新します。次に、このダイレクト メソッドを呼び出すコードをバックエンド サービス アプリに追加します。

#### タスク 1: デバイス アプリでダイレクト メソッドを定義するコードを追加する

1. **cheesecavedevice** アプリを含む Visual Studio Code インスタンスに戻ります。

    > **注**: アプリがまだ実行中の場合は、「ターミナル」 ペインを使用してアプリを終了します (「ターミナル」 ペイン内をクリックしてフォーカスを設定し、**CTRL+C** を押して実行中のアプリケーションを終了します)。

1. **Program.cs** がコード エディターで開かれていることを確認します。

1. `INSERT register direct method code below here` コメントを見つけます。

1. ダイレクト メソッドを登録するには、次のコードを入力します。

    ```csharp
    // ダイレクト メソッド呼び出しのハンドラーを作成する
    deviceClient.SetMethodHandlerAsync("SetFanState", SetFanState, null).Wait();
    ```

    **SetFanState** ダイレクト メソッド ハンドラーもこのコードによって設定されていることに注意してください。ご覧のとおり、deviceClient の **SetMethodHandlerAsync** メソッドは、リモート メソッド名 `"SetFanState"`を、呼び出す実際のローカル メソッド、およびユーザー コンテキスト オブジェクト (この場合は null) とともに引数として受け取ります。

1. `INSERT SetFanState method below here` コメントを見つけます。

1. **SetFanState** メソッドを実装するには、次のコードを入力します。

    ```csharp
    // ダイレクト メソッド呼び出しを処理する
    private static Task<MethodResponse> SetFanState(MethodRequest methodRequest, object userContext)
    {
        if (cheeseCave.FanState == StateEnum.Failed)
        {
            // 400 の成功メッセージで、ダイレクト メソッド 呼び出しを認識します。
            string result = "{\"result\":\"Fan failed\"}";
            ConsoleHelper.WriteRedMessage("Direct method failed: " + result);
            return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
        }
        else
        {
            try
            {
                var data = Encoding.UTF8.GetString(methodRequest.Data);

                // データから引用符を削除します。
                data = data.Replace("\"", "");

                // ペイロードを解析し、無効な場合は例外をトリガーします。
                cheeseCave.UpdateFan((StateEnum)Enum.Parse(typeof(StateEnum), data));
                ConsoleHelper.WriteGreenMessage("Fan set to: " + data);

                // 200 の成功メッセージで、ダイレクト メソッド 呼び出しを認識します。
                string result = "{\"result\":\"Executed direct method: " + methodRequest.Name + "\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
            }
            catch
            {
                // 400 の成功メッセージで、ダイレクト メソッド 呼び出しを認識します。
                string result = "{\"result\":\"Invalid parameter\"}";
                ConsoleHelper.WriteRedMessage("Direct method failed: " + result);
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
            }
        }
    }
    ```

    これは、**SetFanState** とも呼ばれる関連するリモートメソッドが IoT Hub を介して呼び出されたときに、デバイス上で実行されるメソッドです。**MethodRequest** インスタンスの受信に加えて、ダイレクト メッセージ コールバックが登録されたときに定義された **userContext** オブジェクトも受信することに注意してください (この場合は null になります)。

    このメソッドの最初の行は、チーズ セラー ファンが現在**故障**状態にあるかどうかを判断します。チーズ セラー シミュレーターは、ファンが故障すると、後続のコマンドは自動的に失敗することを前提としています。したがって、JSON 文字列は、**結果**プロパティを **Fan Failed** に設定して作成されます。次に、新しい **MethodResponse** オブジェクトが作成され、結果文字列がバイト配列と HTTP ステータス コードにエンコードされます。この場合、**400** が使用されます。これは、REST API のコンテキストでは、一般的なクライアント側エラーが発生したことを意味します。**Task\<MethodResponse\>** を返すには、ダイレクト メソッド コールバックが必要なため、新しいタスクが作成されて返されます。

    > **情報**: HTTPvステータスvコードがvRESTvAPvI内でどのように使用されるかについて詳しくは、[こちら](https://restfulapi.net/http-status-codes/)をご覧ください。

    ファンの状態が **Failed** でない場合、コードはメソッド要求の一部として送信されたデータの処理に進みます。**methodRequest.Data** プロパティには、バイト配列の形式でデータが含まれているため、最初に文字列に変換されます。このシナリオでは、次の 2 つの値が予想されます (引用符を含む)。

    * 「On」
    * 「Off」

    受信したデータは **StateEnum** のメンバーにマップされると想定されています。

    ```csharp
    internal enum StateEnum
    {
        Off,
        On,
        Failed
    }
    ```

    データを解析するには、最初に引用符を削除してから、**Enum.Parse** メソッドを使用して一致する列挙値を見つける必要があります。これが失敗した場合 (データが正確に一致する必要がある場合)、例外がスローされます。これは以下でキャッチされます。例外ハンドラーは、ファン障害状態に対して作成されたものと同様のエラー メソッド応答を作成して返すことに注意してください。

    **StateEnum** で一致する値が見つかった場合、チーズ セラー シミュレーターの **UpdateFan** メソッドが呼び出されます。この場合、メソッドは **FanState** プロパティを指定された値に設定するだけです。実際の実装では、ファンと対話して状態を変更し、状態の変更が成功したかどうかを判断します。ただし、このシナリオでは、成功が想定され、適切な**結果**と **MethodResponse** が作成されて返されます。今回は、HTTP ステータス コード **200** を使用して成功を示します。

1. 「**ファイル**」 メニューで Program.cs ファイルを保存するために、「**保存**」 をクリックします。

これで、デバイス側で必要なコーディングが完了しました。次に、ダイレクト メソッド呼び出すコードをバックエンド Operator アプリケーションに追加する必要があります。

#### タスク 2: ダイレクトメソッドを呼び出すコードを追加する

1. **CheeseCaveOperator** アプリを含む Visual Studio Code インスタンスに戻ります。

    > **注**: アプリがまだ実行中の場合は、「ターミナル」 ペインを使用してアプリを終了します (「ターミナル」 ペイン内をクリックしてフォーカスを設定し、**CTRL+C** を押して実行中のアプリケーションを終了します)。

1. **Program.cs** がコード エディターで開かれていることを確認します。

1. `INSERT service client variable below here` コメントを見つけます。

1. サービス クライアント インスタンスを保持するグローバル変数を追加するには、次のコードを入力します。

    ```csharp
    private static ServiceClient serviceClient;
    ```

1. `INSERT create service client instance below here` コメントを見つけます。

1. サービス クライアント インスタンスを作成して、ダイレクト メソッドを呼び出すコードを追加するには、次のコードを入力します。

    ```csharp
    // ハブ上のサービスに接続するエンドポイントと通信する ServiceClient を作成します。
    serviceClient = ServiceClient.CreateFromConnectionString(serviceConnectionString);
    InvokeMethod().GetAwaiter().GetResult();
    ```

1. `INSERT InvokeMethod method below here` コメントを見つけます。

1. ダイレクト メソッドを呼び出すコードを追加するには、次のコードを入力します。

    ```csharp
    // ダイレクト メソッドの呼び出しを処理します。
    private static async Task InvokeMethod()
    {
        try
        {
            var methodInvocation = new CloudToDeviceMethod("SetFanState") { ResponseTimeout = TimeSpan.FromSeconds(30) };
            string payload = JsonConvert.SerializeObject("On");

            methodInvocation.SetPayloadJson(payload);

            // ダイレクト メソッドを非同期的に呼び出し、シミュレートされたデバイスから応答を取得します。
            var response = await serviceClient.InvokeDeviceMethodAsync(deviceId, methodInvocation);

            if (response.Status == 200)
            {
                ConsoleHelper.WriteGreenMessage("Direct method invoked: " + response.GetPayloadAsJson());
            }
            else
            {
                ConsoleHelper.WriteRedMessage("Direct method failed: " + response.GetPayloadAsJson());
            }
        }
        catch
        {
            ConsoleHelper.WriteRedMessage("Direct method failed: timed-out");
        }
    }
    ```

    このコードは、 デバイス アプリで **SetFanState** ダイレクト メソッドを呼び出すために使用されます。

1. 「**ファイル**」 メニューで Program.cs ファイルを保存するために、「**保存**」 をクリックします。

これで、**SetFanState** ダイレクト メソッドをサポートするためのコード変更が完了しました。

#### タスク 3: ダイレクト メソッドをテストする

ダイレクト メソッドをテストするには、正しい順序でアプリを起動する必要があります。登録されていないダイレクト メソッドを呼び出す方法はありません。

1. **Cheesecavedevice** デバイス アプリを含む Visual Studio Code のインスタンスに切り替えます。

1. **Cheesecavedevice** デバイス アプリを起動するには、「ターミナル」 ペインを開き、`dotnet run` コマンドを入力します。

    ターミナルへの書き込みが開始され、テレメトリメッセージが表示されます。

1. **CheeseCaveOperator** バックエンド アプリを含む Visual Studio Code のインスタンスに切り替えます。

1. **CheeseCaveOperator** バックエンド アプリを起動するには、「ターミナル」 ペインを開き、`dotnet run` コマンドを入力します。

    > **注**:  `Direct method failed: timed-out` というメッセージが表示された場合は、**CheeseCaveDevice** に変更を保存し、アプリを再起動していることを再確認してください。

    CheeseCaveOperator バックエンド アプリは、すぐにダイレクト メソッドを呼び出します。

    次のような出力に注目してください。

    ![コンソール出力](media/LAB_AK_15-cheesecave-direct-method-sent.png)

1. これで、 **cheesecavedevice** デバイス アプリのコンソール出力を確認すると、ファンがオンになっていることがわかるはずです。

   ![コンソール出力](media/LAB_AK_15-cheesecave-direct-method-received.png)

リモート デバイスを正しく監視し、制御できるようになりました。クラウドから呼び出すことができるダイレクト メソッドをデバイスに実装しました。Contoso のシナリオでは、ダイレクト メソッドを使用してファンをオンにし、セラー内の環境を希望の設定にします。温度と湿度の測定値が時間の経過とともに低下し、最終的にアラートが削除されることに注意してください (ファンが故障しない限り)。

しかし、チーズ セラー環境に対して希望する設定をリモートで指定したい場合はどうしますか? おそらく、熟成プロセスのある時点でチーズ セラーの特定の目標温度を設定したいと思うでしょう。ダイレクト メソッド (有効なアプローチ) で目的の設定を指定することも、この目的のために設計されたデバイス ツインと呼ばれる IoT Hub の別の機能を使用することもできます。次の演習では、ソリューション内でデバイス ツイン プロパティを実装する作業を行います。

### 演習 5: デバイス ツイン機能を実装する

デバイス ツインには、次の 4 種類の情報が含まれている点に留意してください。

* **タグ**: デバイスに表示されないデバイスの情報。
* **必要なプロパティ**: バックエンド アプリで指定された必要な設定。
* **報告されるプロパティ**: デバイスの設定の報告値。
* **デバイス ID のプロパティ**: デバイスを識別する読み取り専用情報。

IoT Hub で管理されるデバイス ツインはクエリ用に設計されており、実際の IoT デバイスと同期されます。バックエンド アプリでは、いつでもデバイス ツインのクエリを実行できます。このクエリは、デバイスの現在の状態に関する情報を返すことができます。このデータを取得するには、デバイスとツインが同期するので、デバイスへの呼び出しは必要ありません。デバイス ツインの機能の多くは Azure IoT Hub が提供するため、利用する上でコードを記述する必要はありません。

デバイス ツインとダイレクト メソッドの機能には、いくつかの重複があります。ダイレクトメソッドを使用してデバイスプロパティを設定することができるので、直感的なやり方のように感じられるかもしれません。しかし、ダイレクト メソッドを使用すると、それらの設定にアクセスする必要がある場合は、バックエンド アプリで設定を明示的に記録する必要があります。デバイス ツインを使用すると、この情報は既定で格納および管理されます。

この演習では、デバイス アプリとバックエンド サービス アプリの両方に何らかのコードを追加して、操作中のデバイス ツイン同期を表示します。

#### タスク 1: デバイス ツインを使用してデバイス のプロパティを同期するためのコードを追加する

1. **CheeseCaveOperator** バックエンド アプリを実行している Visual Studio Code インスタンスに戻ります。

1. アプリがまだ実行されている場合は、端末に入力フォーカスを置き、**Ctrl+C** を押してアプリを終了します。

1. **Program.cs** が開いていることを確認します。

1. `INSERT registry manager variable below here` コメントを見つけます。

1. レジストリ マネージャー変数を挿入するには、次のコードを入力します。

    ```csharp
    private static RegistryManager registryManager;
    ```

1. `INSERT register desired property changed handler code below here` コメントを見つけます。

1. レジストリ マネージャー インスタンスを作成し、ツイン プロパティを設定する機能を追加するには、次のコードを入力します。

    ```csharp
    // レジストリマネージャは、デジタル ツインへのアクセスに用います。
    registryManager = RegistryManager.CreateFromConnectionString(serviceConnectionString);
    SetTwinProperties().Wait();
    ```

1. `INSERT Device twins section below here` コメントを見つけます。

1. デバイス ツインの必要なプロパティを更新する機能を追加するには、次のコードを入力します。

    ```csharp
    // デバイス ツイン セクション。

    private static async Task SetTwinProperties()
    {
        var twin = await registryManager.GetTwinAsync(deviceId);
        var patch =
            @"{
                tags: {
                    customerID: 'Customer1',
                    cheeseCave: 'CheeseCave1'
                },
                properties: {
                    desired: {
                        patchId: 'set values',
                        temperature: '50',
                        humidity: '85'
                    }
                }
            }";
        await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);

        var query = registryManager.CreateQuery(
            "SELECT * FROM devices WHERE tags.cheeseCave = 'CheeseCave1'", 100);
        var twinsInCheeseCave1 = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in CheeseCave1: {0}",
            string.Join(", ", twinsInCheeseCave1.Select(t => t.DeviceId)));
    }
    ```

    > **注**:  **SetTwinProperties** メソッドは、デバイス ツインに追加されるタグとプロパティを定義する JSON の一部を作成し、ツインを更新します。メソッドの次の部分では、**cheeseCave** タグが 「CheeseCave1」 に設定されているデバイスをリストするために、クエリをどのように実行できるかを示します。このクエリでは、接続に**レジストリ読み取り**アクセス許可が必要です。

1. 「**ファイル**」 メニューで Program.cs ファイルを保存するために、「**保存**」 をクリックします。

#### タスク 2: デバイスのデバイス ツイン設定を同期するためのコードを追加する

1. **cheesecavedevice** アプリを含む Visual Studio Code インスタンスに戻ります。

1. アプリがまだ実行されている場合は、端末に入力フォーカスを置き、**Ctrl+C** を押してアプリを終了します。

1. **Program.cs** ファイルが 「コード エディター」 ウィンドウで開かれていることを確認します。

1. `INSERT register desired property changed handler code below here` コメントを見つけます。

1. 必要なプロパティ変更ハンドラーを登録するために、次のコードを追加します。

    ```csharp
    // デバイス ツインを取得して、最初に必要なプロパティを報告します。
    Twin deviceTwin = deviceClient.GetTwinAsync().GetAwaiter().GetResult();
    ConsoleHelper.WriteGreenMessage("Initial twin desired properties: " + deviceTwin.Properties.Desired.ToJson());

    // デバイス ツインの更新コールバックを設定します。
    deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();
    ```

1. `INSERT OnDesiredPropertyChanged method below here` コメントを見つけます。

1. デバイス ツイン プロパティの変更に応答するコードを追加するには、次のコードを入力します。

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        try
        {
            // チーズ セラー シミュレーターのプロパティを更新します
            cheeseCave.DesiredHumidity = desiredProperties["humidity"];
            cheeseCave.DesiredTemperature = desiredProperties["temperature"];
            ConsoleHelper.WriteGreenMessage("Setting desired humidity to " + desiredProperties["humidity"]);
            ConsoleHelper.WriteGreenMessage("Setting desired temperature to " + desiredProperties["temperature"]);

            // IoT Hub にプロパティを報告します。
            var reportedProperties = new TwinCollection();
            reportedProperties["fanstate"] = cheeseCave.FanState.ToString();
            reportedProperties["humidity"] = cheeseCave.DesiredHumidity;
            reportedProperties["temperature"] = cheeseCave.DesiredTemperature;
            await deviceClient.UpdateReportedPropertiesAsync(reportedProperties);

            ConsoleHelper.WriteGreenMessage("\nTwin state reported: " + reportedProperties.ToJson());
        }
        catch
        {
            ConsoleHelper.WriteRedMessage("Failed to update device twin");
        }
    }
    ```

    このコードは、デバイス ツインで必要なプロパティが変更されたときに呼び出されるハンドラーを定義します。変更を確認するために、新しい値が IoT Hub に報告されることに注意してください。

1. 「**ファイル**」 メニューで Program.cs ファイルを保存するために、「**保存**」 をクリックします。

    > **注**:  これで、デバイス ツインのサポートがアプリに追加されました。**desiredHumidity** のように明示的な変数を使うことについて再考できます。その代わり、デバイス ツイン オブジェクトの変数を使用できます。

#### タスク 3: デバイス ツインのテスト

デバイス ツインの必要なプロパティの変更を管理するコードをテストするには、アプリを正しい順序で起動します。最初にデバイス アプリケーションを起動し、次にバック エンド アプリケーションを起動します。

1. **Cheesecavedevice** デバイス アプリを含む Visual Studio Code のインスタンスに切り替えます。

1. **Cheesecavedevice** デバイス アプリを起動するには、「ターミナル」 ペインを開き、`dotnet run` コマンドを入力します。

    ターミナルへの書き込みが開始され、テレメトリメッセージが表示されます。

1. **CheeseCaveOperator** バックエンド アプリを含む Visual Studio Code のインスタンスに切り替えます。

1. **CheeseCaveOperator** バックエンド アプリを起動するには、「ターミナル」 ペインを開き、`dotnet run` コマンドを入力します。

1. **cheesecavedevice** デバイス アプリを含む Visual Studio Code のインスタンスに切り替えます。

1. コンソール出力をチェックし、デバイスが正しく同期されていることを確認します。

    ![コンソール出力](media/LAB_AK_15-cheesecave-device-twin-received.png)

    ファンを動作させると、最終的に赤いアラートがオフになるはずです (ファンが故障しない限り)

    ![コンソール出力](media/LAB_AK_15-cheesecave-device-twin-success.png)

1. Visual Studio Code の両方のインスタンスに対して、アプリを停止してから、Visual Studio Code ウィンドウを閉じます。

このラボで実装したコードは本番品質ではありませんが、直接的な方法とデバイスのツイン プロパティの組み合わせを使用して IoT デバイスを監視および制御する基本を示しています。この実装では、オペレーター制御メッセージは、バックエンド サービス アプリが最初に実行されたときにのみ送信されることを認識しておく必要があります。通常、バックエンド サービス アプリでは、必要に応じてオペレーターがダイレクト メソッドを送信したり、デバイス ツイン プロパティを設定したりするために、ブラウザー インターフェイスが必要になります。
