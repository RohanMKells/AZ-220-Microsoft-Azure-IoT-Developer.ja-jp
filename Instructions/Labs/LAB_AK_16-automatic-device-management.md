---
lab:
    title: 'ラボ 16: Azure IoT Hub を使用した IoT デバイス管理の自動化'
    module: 'モジュール 8: デバイス管理'
---

# Azure IoT Hub を使用した IoT デバイス管理の自動化

IoT デバイスは、最適化されたオペレーティング システムを使用したり、シリコン上でコードを直接実行したりします (実際のオペレーティング システムは必要ありません)。このようなデバイスで実行されているソフトウェアを更新するために最も一般的な方法は、OSだけでなく実行されているアプリ (ファームウェアと呼ばれる) を含み、ソフトウェア パッケージ全体の新しいバージョンをフラッシュすることです。

各デバイスには特定の目的があるため、ファームウェアは非常に限定的で、デバイスの目的だけでなく、利用可能な制約されたリソースにも最適化されています。

ファームウェアの更新プロセスは、ハードウェアとハードウェアの製造元がボードを作成した方法に対して固有のプロセスにすることもできます。これは、ファームウェアの更新プロセスの一部が汎用的ではないことを意味し、ファームウェアの更新プロセスの詳細を取得するためにデバイスの製造元と協力する必要があります (ファームウェアの更新プロセスを知っている可能性が高い独自のハードウェアを開発している場合を除きます)。

ファームウェアの更新が個々のデバイスに手動で適用される場合、この方法では一般的な IoT ソリューションで使用されるデバイスの数を考慮することは意味がありません。現在、ファームウェアの更新は、クラウドからリモートで管理する新しいファームウェアの展開により、無線 (OTA) を介して実行することがより一般的になりました。

すべての無線を介した IoT デバイスのファームウェアの更新には、共通する一連の特徴があります。

1. ファームウェアのバージョンは、一意に識別されます
1. ファームウェアはバイナリ ファイル形式で提供されます。デバイスはこれをオンライン ソースから取得する必要があります
1. ローカルに保存されているファームウェアは、何らかの形の物理ストレージです (ROM メモリ、ハードドライブなど)
1. デバイスの製造元は、ファームウェアを更新するために必要なデバイスの操作の説明を提供します。

Azure IoT Hub は、単一のデバイスとデバイスのコレクションにデバイス管理操作を実装するための高度なサポートを提供します。[自動デバイス管理](https://docs.microsoft.com/azure/iot-hub/iot-hub-auto-device-config)機能を使用すると、一連の操作の構成、その操作のトリガー、進行状況の監視を簡単に行えます。

## ラボ シナリオ

Contoso 社のチーズ熟成庫に実装した自動空気処理システムは、同社が既に高い品質水準をさらに上げるのに役立ちました。同社は、チーズでこれまで以上に多くの賞を受賞しています。

基本ソリューションは、センサーと気候制御システムと統合された IoT デバイスで構成されており、マルチチャンバー貯蔵庫システム内の温度と湿度をリアルタイムで制御します。また、ダイレクト メソッドとデバイス ツイン プロパティの両方を使用してデバイスを管理する機能を示す、シンプルなバックエンド アプリを開発しました。

Contoso 社は、初期ソリューションからシンプルなバックエンド アプリを拡張し、オペレーターが蔵環境の監視とリモート管理に使用できるオンライン ポータルを含めています。新しいポータルでは、オペレーターはチーズの種類に基づいて、またはチーズの熟成プロセス内の特定の段階に合わせて蔵内の温度と湿度をカスタマイズすることさえできます。蔵内の各チャンバーまたはゾーンは、別々に制御することができます。

IT 部署は、オペレータ向けに開発したバックエンド ポータルを保守しますが、管理者はソリューションのデバイス側を管理することに同意しています。 

この場合、これは次の 2 つのことを意味します。 

1. Contoso 社の運用チームは、改善方法を常に模索しています。これらの機能強化の結果、デバイス ソフトウェアの新機能のリクエストが寄せられることがよくあります。 

1. 洞窟の場所にデプロイされている IoT デバイスには、プライバシーを確保しハッカーがシステムを掌握するのを防ぐために、最新のセキュリティパッチが必要です。システムのセキュリティを維持するため、デバイスのファームウェアをリモートで更新して、デバイスを最新の状態に保つ必要があります。

自動デバイス管理とデバイス管理を大規模に行えるようにする IoT Hub の機能を実装する予定です。

次のリソースが作成されます。

![ラボ 16 アーキテクチャ](media/LAB_AK_16-architecture.png)

## このラボでは

このラボでは、次のタスクを正常に達成します。

* ラボの前提条件が満たされていることを確認する (必要な Azure リソースがあること)
* ファームウェアの更新を実装するシミュレートされたデバイスのコードを記述します
* Azure IoT Hub の自動デバイス管理を使用して、1 つのデバイスでファームウェアの更新プロセスをテストする

## ラボの手順

### 演習 1: ラボの前提条件を確認する

このラボでは、次の Azure リソースが利用可能であることを前提としています。

| リソースの種類 | リソース名 |
| :-- | :-- |
| リソース グループ | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |
| IoT デバイス | sensor-th-0155 |

> **重要**: セットアップ スクリプトを実行して、必要なデバイスを作成します。

不足しているリソースと新しいデバイスを作成するには、演習 2 に進む前に、以下の手順に従って **lab16-setup.azcli** スクリプトを実行する必要があります。スクリプト ファイルは、開発環境構成 (ラボ 3) の一部としてローカルに複製した GitHub リポジトリに含まれています。

> **注:** **sensor-th-0155** デバイスの接続文字列が必要です。このデバイスが Azure IoT Hub に登録されている場合は、Azure Cloud Shell で次のコマンドを実行して接続文字列を取得できます
>
> ```bash
> az iot hub device-identity connection-string show --hub-name iot-az220-training-{your-id} --device-id sensor-th-0050 -o tsv
> ```

**lab16-setup.azcli** スクリプトは、**Bash** シェル環境で実行するために記述されています。Azure Cloud Shell でこれを実行するのが、最も簡単な方法です。

1. ブラウザーを使用して [Azure Cloud Shell](https://shell.azure.com/) を開き、このコースで使用している Azure サブスクリプションでログインします。

    Cloud Shell のストレージの設定に関するメッセージが表示された場合は、デフォルトをそのまま使用します。

1. Cloud Shell が **Bash** を使用していることを確認します。

    「Azure Cloud Shell」 ページの左上隅にあるドロップダウンは、環境を選択するために使用されます。選択されたドロップダウンの値が **Bash** であることを確認します。

1. Cloud Shell ツール バーで、「**ファイルのアップロード/ダウンロード**」 をクリックします(右から 4番目のボタン)。

1. ドロップダウンで、「**アップロード**」 をクリックします。

1. ファイル選択ダイアログで、開発環境を構成したときにダウンロードした GitHub ラボ ファイルのフォルダーの場所に移動します。

    _ラボ 3: 開発環境の設定_: ZIP ファイルをダウンロードしてコンテンツをローカルに抽出することで、ラボ リソースを含む GitHub リポジトリを複製しました。抽出されたフォルダー構造には、次のフォルダー パスが含まれます。

    * すべてのファイル
      * ラボ
          * Azure IoT Hub を使用した 16 の IoT デバイス管理の自動化
            * 設定

    lab16-setup.azcli スクリプト ファイルは、ラボ 16 の設定フォルダー内にあります。

1. **lab16-setup.azcli** ファイルを選択し、「**開く**」 をクリックします。

    ファイルのアップロードが完了すると、通知が表示されます。

1. 正しいファイルが Azure Cloud Shell にアップロードされたことを確認するには、次のコマンドを入力します。

    ```bash
    ls
    ```

    `ls` コマンドを使用して、現在のディレクトリの内容を表示します。一覧にある lab16-setup.azcli ファイルを確認できるはずです。

1. セットアップ スクリプトを含むこのラボのディレクトリを作成し、そのディレクトリに移動するには、次の Bash コマンドを入力します。

    ```bash
    mkdir lab16
    mv lab16-setup.azcli lab16
    cd lab16
    ```

1. **lab16-setup.azcli** に実行権限があることを確認するには、次のコマンドを入力します。

    ```bash
    chmod +x lab16-setup.azcli
    ```

1. Cloud Shell ツールバーで、lab16-setup.azcli ファイルへのアクセスを有効にするには、「**エディタを開く**」 (右から 2 番目のボタン - **{ }**) をクリックします。

1. 「**ファイル**」 の一覧で、lab16 フォルダーを展開してスクリプト ファイルを開くには、「**lab16**」 をクリックし、「**lab16-setup.azcli**」 をクリックします。

    エディタは **lab16-setup.azcli** ファイルの内容を表示します。

1. エディターで、割り当て済みの値 `{your-id}` と `{your-location}` を更新します。

    以下の参考サンプルでは、このコースの開始時に作成した固有 id に **(cah191211)** などの `{your-id}` を設定し、`{your-location}` をリソースにとって意味のある場所に設定する必要があります。

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
    ./lab16-setup.azcli
    ```

    このスクリプトの実行には数分かかります。各ステップが完了すると、出力が表示されます。

    このスクリプトは、まず **rg-az220** という名前のリソース グループ と **iot-az220-training-{your-id}** という名前の IoT Hub を作成します。既に存在する場合は、対応するメッセージが表示されます。次にスクリプトは、**sensor-th-0155** の ID を持つデバイスを IoT Hub に追加し、デバイスの接続文字列を表示します。

1. スクリプトが完了すると、デバイスの接続文字列が表示されることに注意してください。

    接続文字列は「HostName=」で始まります。

1. 接続文字列をテキスト ドキュメントにコピーし、**sensor-th-0155** デバイス用であることに注意してください。

    接続文字列を簡単に見つけることができる場所に保存すると、ラボを続ける準備が整います。

### 演習 2: ファームウェアの更新を実装するシミュレートされたデバイスのコードを記述します

この演習では、デバイス ツインの必要なプロパティの変更を管理し、ファームウェアの更新をシミュレートするローカル プロセスをトリガーする簡単なシミュレートされたデバイスを作成します。ファームウェア更新を起動するために実装するプロセスは、実際のデバイスでファーム更新に使用されるプロセスと同様です。新しいファームウェア バージョンのダウンロード、ファームウェア更新のインストール、およびデバイスの再起動のプロセスがシミュレートされます。

Azure portal を使用して、デバイス　ツインのプロパティを使用してファームウェアの更新を構成および実行します。デバイス ツインのプロパティを使用して、構成変更要求をデバイスに転送し、進行状況を監視します。

#### タスク 1: デバイス シミュレーター アプリを作成する

このタスクでは、Visual Studio Code を使用して、新しいコンソール アプリを作成します。

1. Visual Studio Code を開きます。

    このコースの課題 3 を修了すると、開発環境に [.NET Core](https://dotnet.microsoft.com/download) と [C# 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)がインストールされているはずです。

1. 「**ターミナル**」 メニューで、「**新しいターミナル**」 をクリックします。

1. ターミナル コマンド プロンプトで、次のコマンドを入力します。

    ```cmd/sh
    mkdir fwupdatedevice
    cd fwupdatedevice
    ```

    最初のコマンドは、 **fwupdatedevice** と呼ばれるフォルダを作成します。2 番目のコマンドで、 **fwupdatedevice** フォルダーに移動します。

1. 新しいコンソール アプリを作成するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet new console
    ```

    > **注**: 新しい .NET コンソール アプリが作成されると、`dotnet restore` は作成後のプロセスとして実行されている必要があります。「ターミナル」 ペインに、この問題が発生したことを示すメッセージが表示されない場合、アプリは必要な .NET パッケージにアクセスできない場合があります。これに対処するためには、次のコマンドを入力します。`dotnet restore`

1. アプリに必要なライブラリをインストールするには、次のコマンドを入力します。

    ```cmd/sh
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Microsoft.Azure.Devices.Shared
    dotnet add package Newtonsoft.Json
    ```

    「ターミナル」 ウィンドウでメッセージを確認し、3 つのライブラリがすべてインストールされていることを確認します。

1. **「ファイル」** メニューで、**「フォルダを開く」** を選択します。

1. **「フォルダを開く」** ダイアログで、「ターミナル」 ウィンドウで指定したフォルダの場所に移動し、 **「fwupdatedevice」** をクリックしてから、**「フォルダの選択」** をクリックします。

    「EXPLORER」 ウィンドウが Visual Studio Code で開き、`Program.cs` と `fwupdatedevice.csproj` ファイルが一覧表示されます。

1. **EXPLORER** ペインで、**Program.cs** をクリックします。

1. 「コード エディター」 ペインで、Program.cs ファイルの内容を削除します。

#### タスク 2: アプリにコードを追加する

このタスクでは、IoT Hub で生成された要求に応答して、デバイスのファームウェア更新プログラムをシミュレートするためのコードを入力します。

1. **Program.cs** ファイルが Visual Studio Code で開かれていることを確認します。

    「コード エディター」 ペインには、空のコード ファイルが表示されます。

1. 次のコードをコピーして 「コード エディター」 ペインに貼り付けます。

    ```cs
    // Copyright (c) Microsoft. All rights reserved.
    // MIT ライセンスの下でライセンスされています。ライセンス情報の全容については、プロジェクト ルートのライセンス ファイルをご覧ください。

    using Microsoft.Azure.Devices.Shared;
    using Microsoft.Azure.Devices.Client;
    using System;
    using System.Threading.Tasks;

    namespace fwupdatedevice
    {
        class SimulatedDevice
        {
            // IoT Hub でデバイスを認証するためのデバイス接続文字列。
            static string s_deviceConnectionString = "";

            // デバイス ID 変数
            static string DeviceID="unknown";

            // ファームウェア バージョン変数
            static string DeviceFWVersion = "1.0.0";

            // シンプルなコンソール ログ関数
            static void LogToConsole(string text)
            {
                // デバイス ID にログを前に付けます
                Console.WriteLine(DeviceID + ": " + text);
            }

            // OS/HW からファームウェア バージョンを取得する機能
            static string GetFirmwareVersion()
            {
                // ここでは、ハードウェアから実際のファームウェアのバージョンを取得します。シミュレーションの目的で FWVersion 変数値を返送します
                return DeviceFWVersion;
            }

            // 現在のファームウェア (更新) の状態を報告するデバイス ツイン報告プロパティを更新する機能
            // IoT Hub のファームウェア更新の構成によって 「ファームウェア」更新プロパティで想定される値を次に示します
            //  currentFwVersion: デバイスで現在実行されているファームウェアのバージョン。
            //  pendingFwVersion: 更新後の次のバージョン、一致すべき項目
            //                    目的のプロパティで指定されています。空白の場合
            //                    保留中の更新はありません (fwUpdateStatus は '最新' です)。
            //  fwUpdateStatus:   更新の進行状況を定義して、
            //                    概要ビューから分類できるようにします。つぎのいずれかです。
            //         - 最新:     保留中のファームウェア更新はありません。currentFwVersion は
            //                    目的のプロパティの fwVersion と一致する必要があります。
            //         - ダウンロード中: ファームウェア更新イメージをダウンロード中です。
            //         - 検証:   イメージ ファイルのチェックサムおよびその他の検証を検証しています。
            //         - 適用:    新しいイメージ ファイルへの更新が進行中です。
            //         - 再起動:   デバイスは更新プロセスの一部として再起動中です。
            //         - エラー:       更新処理中にエラーが発生しました。追加情報
            //                     fwUpdateSubstatusで指定する必要があります。
            //        - ロールバック:  エラーのため、更新プログラムは以前のバージョンにロールバックされました。
            //  fwUpdateSubstatus: fwUpdateStatus の追加の詳細。含めることができます
            //                     エラーまたはロールバックの状態、またはダウンロードの % の理由。
            //
            // reported: {
            //       firmware: {
            //         currentFwVersion: '1.0.0',
            //         pendingFwVersion: '',
            //         fwUpdateStatus: 'current',
            //         fwUpdateSubstatus: '',
            //         lastFwUpdateStartTime: '',
            //         lastFwUpdateEndTime: ''
            //   }
            // }

            static async Task UpdateFWUpdateStatus(DeviceClient client, string currentFwVersion, string pendingFwVersion, string fwUpdateStatus, string fwUpdateSubstatus, string lastFwUpdateStartTime, string lastFwUpdateEndTime)
            {
                TwinCollection properties = new TwinCollection();
                if (currentFwVersion!=null)
                    properties["currentFwVersion"] = currentFwVersion;
                if (pendingFwVersion!=null)
                    properties["pendingFwVersion"] = pendingFwVersion;
                if (fwUpdateStatus!=null)
                    properties["fwUpdateStatus"] = fwUpdateStatus;
                if (fwUpdateSubstatus!=null)
                    properties["fwUpdateSubstatus"] = fwUpdateSubstatus;
                if (lastFwUpdateStartTime!=null)
                    properties["lastFwUpdateStartTime"] = lastFwUpdateStartTime;
                if (lastFwUpdateEndTime!=null)
                    properties["lastFwUpdateEndTime"] = lastFwUpdateEndTime;

                TwinCollection reportedProperties = new TwinCollection();
                reportedProperties["firmware"] = properties;

                await client.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
            }

            // デバイスでファームウェアの更新を実行します
            static async Task UpdateFirmware(DeviceClient client, string fwVersion, string fwPackageURI, string fwPackageCheckValue)
            {
                LogToConsole("A firmware update was requested from version " + GetFirmwareVersion() + " to version " + fwVersion);
                await UpdateFWUpdateStatus(client, null, fwVersion, null, null, DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ"), null);

                // 新しいファームウェア バイナリを取得します。ここでは、バイナリをダウンロードするか、お使いのデバイスの指示に従ってソースから取得し、ダウンロードしたバイナリの整合性をハッシュで再確認します
                LogToConsole("Downloading new firmware package from " + fwPackageURI);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "0", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "25", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "50", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "75", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "100", null, null);
                // バイナリがダウンロードされたことを報告する
                LogToConsole("The new firmware package has been successfully downloaded.");

                // バイナリの整合性を確認する
                LogToConsole("Verifying firmware package with checksum " + fwPackageCheckValue);
                await UpdateFWUpdateStatus(client, null, null, "verifying", null, null, null);
                await Task.Delay(5 * 1000);
                // バイナリがダウンロードされたことを報告する
                LogToConsole("The new firmware binary package has been successfully verified");

                // 新しいファームウェアを適用
                LogToConsole("Applying new firmware");
                await UpdateFWUpdateStatus(client, null, null, "applying", null, null, null);
                await Task.Delay(5 * 1000);

                // 実際のデバイスでは、プロセスの最後に再起動し、ブート時にデバイスは実際のファームウェアバージョンを報告します。成功した場合、新バージョンになります。
                // シミュレーションのために、しばらく待って新しいファームウェアのバージョンを報告します
                LogToConsole("Rebooting");
                await UpdateFWUpdateStatus(client, null, null, "rebooting", null, null, DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ"));
                await Task.Delay(5 * 1000);

                // 実際のデバイスでは、デバイスを再起動するコマンドを発行します。ここでは、単に init 関数を実行しています
                DeviceFWVersion = fwVersion;
                await InitDevice(client);

            }

            // 必要なプロパティの変更に応答するためのコールバック
            static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
            {
                LogToConsole("Desired property changed:");
                LogToConsole($"{desiredProperties.ToJson()}");

                // ファームウェアの更新を実行します
                if (desiredProperties.Contains("firmware") && (desiredProperties["firmware"]!=null))
                {
                    // 目的のプロパティでは、次の情報が表示されます。
                    // fwVersion: フラッシュする新しいファームウェアのバージョン番号
                    // fwPackageURI: 新しいファームウェア バイナリをダウンロードする場所の URI
                    // fwPackageCheckValue: ダウンロードされたバイナリの整合性を検証するためのハッシュ
                    // ファームウェアのバージョンは新しいものと仮定します
                    TwinCollection fwProperties = new TwinCollection(desiredProperties["firmware"].ToString());
                    await UpdateFirmware((DeviceClient)userContext, fwProperties["fwVersion"].ToString(), fwProperties["fwPackageURI"].ToString(), fwProperties["fwPackageCheckValue"].ToString());

                }
            }

            static async Task InitDevice(DeviceClient client)
            {
                LogToConsole("Device booted");
                LogToConsole("Current firmware version: " + GetFirmwareVersion());
                await UpdateFWUpdateStatus(client, GetFirmwareVersion(), "", "current", "", "", "");
            }

            static async Task Main(string[] args)
            {
                // コマンド ラインからデバイス接続文字列を取得します
                if (string.IsNullOrEmpty(s_deviceConnectionString) && args.Length > 0)
                {
                    s_deviceConnectionString = args[0];
                } else
                {
                    Console.WriteLine("Please enter the connection string as argument.");
                    return;
                }

                DeviceClient deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);

                if (deviceClient == null)
                {
                    Console.WriteLine("Failed to create DeviceClient!");
                    return;
                }

                // デバイス ID を取得します
                string[] elements = s_deviceConnectionString.Split('=',';');

                for(int i=0;i<elements.Length; i+=2)
                {
                    if (elements[i]=="DeviceId") DeviceID = elements[i+1];
                }

                // デバイスの初期化ルーチンを実行します
                await InitDevice(deviceClient);

                // 必要なプロパティの変更に対するコールバックのアタッチ
                await deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, deviceClient).ConfigureAwait(false);

                // アプリを終了するキーストロークを待つ
                // TODO
                while (true)
                {
                    Console.ReadLine();
                    return;
                }
            }
        }
    }
    ```

    > **注**: 
    > デバイスがデバイス ツインの変更に対してどのように反応し、必須プロパティの 「firmware」 で共有された構成に基づいてファームウェアの更新を実行するかに注意しながら、コードのコメントに目を通してください。また、デバイス ツインの報告されたプロパティを通じて、現在のファームウェアの更新状態を報告する関数も注目に値します。

1. 「**ファイル**」 メニューの 「**上書き保存**」 をクリックします。

これで、デバイス側のコードが完成しました。次に、このシミュレートされたデバイスでファームウェアの更新プロセスが期待どおりに動作することをテストします。

### 演習 3: 1 台のデバイスでのファームウェアの更新をテストする

この演習では、Azure portal を使用して新しいデバイス管理構成を作成し、それを単一のシミュレートされたデバイスに適用します。

#### タスク 1: デバイス シミュレーターを起動する

1. 必要に応じて **fwupdatedevice** プロジェクトを Visual Studio Code で開きます。

1. 「ターミナル」 ウィンドウが開いていることを確認します。

    コマンド プロンプトのフォルダーの場所は、`fwupdatedevice` フォルダーです。

1. `fwupdatedevice` アプリを実行するには、次のコマンドを入力します。

    ``` bash
    dotnet run "<device connection string>"
    ```

    > **注**: プレースホルダーを実際のデバイス接続文字列に置き換え、接続文字列の周囲に "" を必ず含めることに注意してください。 
    > 
    > 例: `"HostName=iot-az220-training-{your-id}.azure-devices.net;DeviceId=sensor-th-0155;SharedAccessKey={}="`

1. 「ターミナル」 ウインドウの内容を確認します。

    ターミナルに次の出力が表示されます (「mydevice」 は、デバイス ID を作成するときに使用したデバイス ID です)。

    ``` bash
        mydevice: Device booted
        mydevice: Current firmware version: 1.0.0
    ```

#### タスク 2: デバイス管理構成を作成する

1. 必要に応じて、Azure アカウントの資格情報を使用して [Azure Portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true) にログインします。

    複数の Azure アカウントをお持ちの場合は、このコースで使用するサブスクリプションに関連付けられているアカウントを使用してログインしていることを確認してください。

1. Azure portal ダッシュボードで、「**iot-az220-training-{your-id}**」 をクリックします。

    IoT Hub ブレードが表示されるようになりました。
 
1. 左側のナビゲーション メニューの **「自動デバイス管理」** で 、**「IoT デバイスの構成」** をクリックします。

1. **「IoT デバイスの構成」** ウィンドウで、**「+ デバイス構成の追加」** をクリックします。

1. **「デバイス ツイン構成の作成」** ブレードの **「名前」** で、**firmwareupdate** と入力します。

    **「ラベル」** の下ではなく、構成に必要な **「名前」** フィールドに 「firmwareupdate」と入力していることを確認 します。 

1. ブレードの最下部で、「**次へ:**」 をクリックします。**ツインズの設定 >**。

1. **「デバイス ツインの設定」** で、**「デバイス ツインプロパティ」** フィールドに **properties.desired.firmware** と入力します

1. **「デバイス ツイン プロパティのコンテンツ」** フィールドに、次のように入力します。

    ``` json
    {
        "fwVersion":"1.0.1",
        "fwPackageURI":"https://MyPackage.uri",
        "fwPackageCheckValue":"1234"
    }
    ```

1. ブレードの最下部で、「**次へ:**」 をクリックします。**メトリック >**。

    カスタム メトリックを使用して、ファームウェアの更新が有効であったかどうかを追跡します。 

1. **「メトリック」** タブの **「メトリック名」** で **fwupdated** と入力します

1. **「メトリック基準」** で、次の項目を入力します。

    ``` SQL
    SELECT deviceId FROM devices
        WHERE properties.reported.firmware.currentFwVersion='1.0.1'
    ```

1. ブレードの最下部で、「**次へ:**」 をクリックします。**ターゲット デバイス >**。

1. **「ターゲット デバイス」** タブの **「優先度」** で、**「優先度(高い値...)」** フィールドに **「10」** と入力します。

1. **「ターゲット条件」** の **「ターゲット条件」** フィールドに、次のクエリを入力します。

    ``` SQL
    deviceId='<your device id>'
    ```

    > **注**: `'<your device id>'` をデバイスの作成に使用したデバイス ID に置き換えてください。例: `'sensor-th-0155'`

1. ブレードの最下部で、「**次へ:**」 をクリックします。**Review + create >**

    **「Review + create」** タブが開くと、新しい構成の 「検証に成功しました」というメッセージが表示されます。 

1. **「Review + create」** タブで、「検証が成功しました」というメッセージが表示されたら、**「作成」** をクリックします。

    「検証が成功しました」というメッセージが表示された場合は、設定を作成する前に、作業を確認する必要があります。

1. 「**IoT デバイスの構成**」 ペインの 「**構成名**」 に、新しい **firmwareupdat** の構成が一覧表示されていることを確認します。  

    新しい構成が作成されると、IoT ハブは構成のターゲット デバイスの条件に一致するデバイスを探し、ファームウェアの更新構成を自動的に適用します。

1. 「Visual Studio Code」 ウィンドウに切り替え、ターミナル ペインの内容を確認します。

    「ターミナル」 ペインには、トリガーされたファームウェア更新プロセスの進行状況を一覧表示するアプリによって生成された新しい出力が含まれている必要があります。

1. シミュレートされたアプリを停止し、Visual Studio Code を閉じます。

    ターミナルの「Enter」キーを押すだけで、デバイス シミュレーターを停止できます。

