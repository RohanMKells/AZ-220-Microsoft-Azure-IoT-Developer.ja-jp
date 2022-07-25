---
lab:
  title: ラボ 20:IoT Central を使用して構築する
  module: 'Module 12: Build with IoT Central'
ms.openlocfilehash: dada6104c6bf5214db4685d4121354da830d2f9c
ms.sourcegitcommit: 690bdd47c6edc5151e872f3477aa2dbc02a703e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/24/2022
ms.locfileid: "145883699"
---
# <a name="create-your-first-azure-iot-central-app"></a>最初の Azure IoT Central アプリを作成する

Azure IoT サービスとテクノロジは優れた機能を備えており、チームにメンバーがいる場合は管理が簡単ですが、完全な IoT ソリューション アーキテクチャがあれば、小規模で専門性の低いチームでもソリューションを実装およびサポートすることができます。 Azure IoT Central は、Azure IoT Hub、Azure Device Provisioning System (DPS)、Azure Maps、Azure Time Series Insights、Azure IoT Edge など、基盤となる IoT テクノロジの幅広い範囲を網羅する IoT のサービスとしてのアプリケーション プラットフォーム (aPaaS) です。 IoT Central では、これらのテクノロジを直接実装するときに得られるレベルの細分性は提供されませんが、小規模なチームは一連のリモート デバイスを簡単に管理および監視できるようになります。

特に、このラボは、IoT Central が特定のシナリオをサポートする適切なツールである場合を判断するのに役立ちます。

## <a name="lab-scenario"></a>課題シナリオ

Contoso チーズ会社は、都市とその周辺地域でチーズの配送に使用する冷凍トラックのフリートを運営しています。 この地域には多数の顧客がおり、フリートの運用のベースとして市内の 1 つの場所が使われています。 毎日、トラックには製品が積み込まれ、ドライバーはディスパッチャーから配送ルートを与えられます。 システムはうまく機能しており、めったに問題は起きません。 しかし、トラックの冷却システムに障害が発生した場合、ドライバーとディスパッチャーは最善の配達方法を話し合う必要があります。 ディスパッチャーは、製品を倉庫に戻して検査するか、車両の現在地に近い顧客の場所に配送します。 トラックに残っている未出荷の製品の量と、冷蔵エリアの温度は、どちらも決定の要因です。

情報に基づいて決定するために、ドライバーとディスパッチャーは、トラックと運搬している製品に関する最新の情報を必要とします。 ドライバーとディスパッチャーは、地図上の各トラックの位置、トラックの冷却システムの状態、およびトラックの貨物の状態を知る必要があります。

IoT Central には、このシナリオを処理するために必要なあらゆるものが用意されています。

> **注**:このラボを行うには、Microsoft アカウントを使用できます。

次のリソースが作成されます。

![ラボ 20 アーキテクチャ](media/LAB_AK_20-architecture.png)

## <a name="in-this-lab"></a>このラボでは

このラボでは、次のタスクを正常に達成します。

* IoT Central ポータルを使用して、Azure IoT Central カスタム アプリを作成する
* IoT Central ポータルを使用して、カスタム デバイス用にデバイス テンプレートを作成する
* Visual Studio Code または Visual Studio を使用して、Azure Maps によって選択されたルートで冷凍ラックをシミュレートするプログラミング プロジェクトを作成する
* IoT Central ダッシュボードから、シミュレートされたデバイスの監視および指揮を行う

## <a name="lab-instructions"></a>ラボの手順

### <a name="exercise-1-create-and-configure-azure-iot-central"></a>演習 1:Azure IoT Central を作成して構成する

#### <a name="task-1-create-the-initial-iot-central-app"></a>タスク 1:最初の IoT Central アプリを作成する

1. [Azure IoT Central](https://apps.azureiotcentral.com/?azure-portal=true) に移動します。

    この URL は、すべての IoT Central アプリのホーム ページとしてブックマークしておくことをお勧めします。

1. 少し下にスクロールして、このホームページの内容を読んでください。

1. 左側のナビゲーション メニューで、 **[ビルド]** をクリックします。

    特定のシナリオに、より高度な開始点を提供するいくつかのオプションがあることに注意してください。

1. **[おすすめ]** で、カスタム アプリを作成するには、 **[アプリの作成]** をクリックします。

    求められたら、自分の Microsoft アカウントでサインインします。 

1. **[新しいアプリケーション]** ページの **[アプリケーション名]** に、「**Refrigerated-Trucks-{your-id}** 」と入力します。

    > **注**:このラボの間に、グローバルに一意のリソースの名前または値を入力する必要がある場合は常に、推奨されるリソース名の一部として {your-id} が表示されます。 推奨されるリソース名の {your-id} の部分はプレースホルダーです。 プレースホルダーの ({} を含む) 文字列全体を、一意の ID 置き換えます。"YourInitialsYYMMDD" というパターンを使って、イニシャル (小文字) と現在の数値日付を組み合わせることで、一意の ID 値を作成できます。 例: ccj220515

    入力したアプリケーション名がアプリケーション URL のルートとして使用されていることに注意してください (小文字に変換されます)。

    アプリケーション名は任意のフレンドリ名にできますが、**URL** は一意である "_必要があります_"。 2 つを完全に一致させる必要はありませんが、一致させると混乱が少なくなります。

    アプリケーション名に **{your-id}** を追加すると、URL を一意にするのに役立ちます。

1. **[アプリケーション テンプレート]** は、デフォルトの **[カスタム アプリケーション]** の値のままにします。

1. 少し時間を取って、 **[料金プラン]** と **[課金情報]** セクションのフィールドを確認します。

    **[ディレクトリ]** フィールドは、Azure Active Directory テナントを指定するために使用されます。 組織で AAD テナントを使用する場合は、ここで指定します。 このラボでは、既定値のままにします。

    コストを含む価格設定オプションを選択する場合は、Azure サブスクリプションを指定する必要があります。

1. **[価格プラン]** の **[Free]** をクリックします。

    無料オプションでは 7 日間の試用版が提供され、5 つの無料デバイスが含まれていることに注意してください。 

    > **注**: **[Free]** 価格プランを選んだら、それ以上 **[課金情報]** を入力する必要はありません。

    > **注**:プランに関するコミットメントまたは解約料はありません。 IoT Central の価格について詳しく知りたい場合は、 **[ビルド]** ページに [[価格の詳細を取得する](https://aka.ms/iotcentral-pricing)] へのリンクが含まれています。

1. ページの下部で、 **[作成]** をクリックします。

    アプリ リソースが構築されるまで数秒待つと、デフォルトのリンクがいくつかある **ダッシュボード** が表示されます。

1. Azure IoT Central ブラウザーのタブを閉じます。

    次回、Azure IoT Central ホーム ページを開いたときに、左側のナビゲーション メニューから **[マイ アプリ]** を選択すると、**Refrigerated-Trucks-{your-id}** アプリが一覧表示されます。

1. ブラウザーを使用して [Azure IoT Central](https://apps.azureiotcentral.com/?azure-portal=true) を開きます。

1. 左側のナビゲーション メニューで、 **[マイ アプリ]** をクリックし、 **[Refrigerated-Trucks-{your-id}]** をクリックします。

    次の手順では、_デバイス テンプレート_ を指定します。

#### <a name="task-2-create-a-device-template"></a>タスク 2:デバイス テンプレートを作成する

リモート デバイスと IoT Central の間で通信されるデータは、_デバイス テンプレート_ で指定されます。 デバイス テンプレートでは、データに関するすべての詳細がカプセル化されます。これにより、デバイスと IoT Central の両方で通信を解釈するために必要なすべての情報が得られます。

1. Azure IoT Central アプリの **[ダッシュボード]** ページの、左側のナビゲーション メニューの **[接続]** で、 **[デバイス テンプレート]** をクリックします。

1. **[デバイス テンプレート]** で、 **[デバイス テンプレートの作成]** をクリックします。

    カスタム デバイス テンプレートおよび構成済みのデバイス テンプレートのオプションの範囲が表示されます。

    > **助言**：構成済みのオプションをメモします。 関連するハードウェアがある場合は、これらの構成済みのデバイス テンプレートのいずれかを将来のプロジェクトに使用できます。

1. **[カスタム デバイス テンプレートの作成]** で、 **[IoT デバイス]** をクリックします。

1. ページの下部で、 **[Next: Customize] \(次へ: カスタマイズ\)** をクリックします

1. **[デバイス テンプレート名]** ボックスに、「**RefrigeratedTruck**」と入力します。

    > **注**: **[This is a gateway device]\(これはゲートウェイ デバイスです\)** は選択しないでください。

1. ページの下部で、 **[次へ: レビュー]** をクリックします。

    表示されている **[基本情報]** が正しいことを確認します。

1. **[レビュー]** ページの下部で、 **[作成]** をクリックします。

    テンプレートが作成されると、**RefrigeratedTruck** ページが表示されます。

1. "**RefrigeratedTruck**" のページの **[モデルの作成]** で、 **[カスタム モデル]** をクリックします。

    これで、デバイス テンプレートの詳細を追加する準備ができました。

#### <a name="task-3-add-sensor-telemetry"></a>タスク 3:センサー テレメトリの追加

テレメトリは、センサーによって送信されるデータ値です。 冷凍トラックの最も重要なセンサーでは、荷物の温度が監視されます。

1. "**RefrigeratedTruck**" デバイス テンプレートのページで、必要に応じて省略記号 (3 つのドット) をクリックし、 **[+ 継承されたインターフェイスの追加]** をクリックしてから、 **[カスタム]** をクリックします

    インターフェイスにより、"_機能_" のセットが定義されます。 冷凍トラックの機能を定義するには、かなりの数のインターフェイスを追加します。

    カスタム インターフェイスを使用すると、空のインターフェイスから構築を開始できます。

1. **[継承インターフェイス]** で、 **[+ 機能の追加]** をクリックします

1. 少し時間を取って、 **[機能の種類]** と **[セマンティックの種類]** のオプションを確認してください。

1. トラックの温度センサーの機能を定義するには、次のフィールド値を入力します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | 荷物の温度 |
    | 名前 | ContentsTemperature |
    | 機能の種類 | テレメトリ |
    | セマンティックの種類 | 気温 |
    | スキーマ | 倍精度浮動小数点 |
    | 単位 | 摂氏 |

    > **注**:[スキーマ] と [単位] のドロップダウン リストを入力するには、下向きのキャレットを選んで機能フィールドを展開します。

1. 入力した情報を再確認してください。

    > **重要**:このラボの後半で追加するコードでは、上記の名前を使用します。したがって、インターフェイスに入力する名前は、表示のように _正確に_ 入力する必要があります。

#### <a name="task-4-add-state-and-event-telemetry"></a>タスク 4:状態とイベントのテレメトリを追加する

状態は、それによってオペレーターは何が起こっているかを知ることができるため、重要です。 IoT Central の状態は、さまざまな値に関連付けられた名前です。 このラボの後半では、各状態値に関連付ける色を選択し、識別しやすくします。

1. **[継承インターフェイス]** で、 **[+ 機能の追加]** をクリックします

1. トラックの貨物状態の機能を定義するには、次のフィールド値を入力します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | 荷物の状態 |
    | 名前 | ContentsState |
    | 機能の種類 | テレメトリ |
    | セマンティックの種類 | 状態 |
    | 値スキーマ | 文字列 |

1. **[値スキーマ]** の下で、 **[複合型を定義する必要があります]** というメッセージが表示されます。

    ラボのシナリオを簡略化するために、トラックの貨物状態を _空_、_満杯_、または _溶融_ のいずれかとして定義します。

1. "**複合型を定義する必要があります**" というメッセージの下の、 **[+ 追加]** をクリックします。

1. **[表示名]** の下に、**空の値** を入力します

    **[名前]** フィールドには、自動的に **空の値** が入力されます。

1. **[値]** に「**空**」と入力します

    3 つのフィールドはすべて **空** になります。

1. 入力したフィールドのすぐ下にある **[+ 追加]** をクリックします。

1. 上記のようなやり方で、もう 2 つの状態値 **満杯** と **融解** を追加します。

    ここでも、追加した各状態値のオプションについて、 **[表示名]** 、 **[名前]** 、 **[値]** の各フィールドに同じテキストが表示されます。

1. 各機能を慎重に確認してから、次に進んでください。

    ここで、シミュレーションに不確実性を加えるために、トラックの冷却システムの故障状態を追加してみましょう。 冷却システムに障害が発生した場合、このラボで後述するとおり、搭載物が "融解" する可能性が大幅に高まります。 トラックの冷却システムに対して _オン_ 、_オフ_ 、_故障_ のエントリを追加します。

1. **RefrigeratedTruck** デバイス テンプレートのページで、 **[継承インターフェイス]** の **[+ 機能の追加]** をクリックします。

1. トラックの冷却システムの状態を扱う機能を定義するために、次のフィールド値を入力します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | 冷却システムの状態 |
    | 名前 | CoolingSystemState |
    | 機能の種類 | テレメトリ |
    | セマンティックの種類 | 状態 |
    | 値スキーマ | 文字列 |

1. "**複合型を定義する必要があります**" というメッセージの下の **[** ]+ をクリックして、次の状態値のオプションを上記と同じ方法で追加します。

    * on
    * オフ
    * 失敗

    **[表示名]** 、 **[名前]** 、 **[値]** の 3 つすべてのフィールドで、3 つの状態値オプション (オン、オフ、失敗) が繰り返されていることを確認します。

    トラック自体には、さらに複雑な状態を定義する必要があります。 すべてが順調な場合、トラックの通常のルート指定は、_ready_、_enroute_、_delivering_、_returning_、_loading_ となって _ready_ に戻ります。 いっぽう、溶けた積み荷を倉庫に持ち帰って検査する (そして廃棄するかもしれない) 場合に対応するために、 _荷下ろし中_ の状態も含めるでしょう。

1. 前述した状態の機能を定義したのと同じやり方で、新しい機能を次のように作成します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | トラックの状態 |
    | 名前 | TruckState |
    | 機能の種類 | テレメトリ |
    | セマンティックの種類 | 状態 |
    | 値スキーマ | 文字列 |

    状態値オプションを定義するには、複合型メッセージの下に次の値を追加します。

    * ready
    * 配送中
    * 配信
    * 結果を返して
    * 読み込み
    * ダンプ

    指定する必要がある次の機能タイプは、イベントです。 デバイスによってイベントがトリガーされて、IoT Central アプリに通知されます。 イベントは、次の 3 種類のいずれかにできます。_エラー_、_警告_、または _情報_。

1. イベント機能を作成するために、 **[+ 機能の追加]** をクリックして新しい機能を次のように設定します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | イベント |
    | 名前 | Event |
    | 機能の種類 | テレメトリ |
    | セマンティックの種類 | イベント |
    | スキーマ | 文字列 |

    デバイスがイベントをトリガーする可能性がある場合の 1 つの例は、競合するコマンドです。 たとえば、トラックが顧客から空で帰ってきた後、その荷物を別の顧客に配送する指令を受け取る場合のシナリオを考えます。 この種の競合が発生した場合は、デバイスでイベントをトリガーして、IoT Central アプリのオペレーターに警告するのがよい方法です。

    イベントのもう 1 つの用途は、情報の確認です。 たとえば、トラックの配送先の顧客 ID を確認して記録するだめに、イベントが使われる可能性があります。

#### <a name="task-5-add-location-telemetry"></a>タスク 5:位置情報テレメトリの追加

トラックの現在の場所は、配送追跡シナリオにとって非常に重要ですが、デバイス テンプレートに追加するテレメトリ測定としては最も簡単なものの 1 つです。 内部的には、デバイスの緯度、経度、およびオプションの高度で構成されます。

1. 位置情報の機能を作成するために、 **[+ 機能の追加]** をクリックして新しい機能を次のように設定します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | 位置情報 |
    | 名前 | Location |
    | 機能の種類 | テレメトリ |
    | セマンティックの種類 | 位置情報 |
    | スキーマ | Geopoint |

#### <a name="task-6-add-properties"></a>タスク 6:プロパティの追加

デバイスのプロパティは、通常は定数値で、通信が最初に開始されたときに IoT Central アプリに伝達されます。 冷蔵トラックのシナリオでは、トラックのナンバー プレートのように、トラックに固有の ID がプロパティの良い例です。

プロパティは、デバイスの構成データにも使用できます。 トラックの積み荷の _最適温度_ をプロパティとして定義することもできるでしょう。 この適温は、さまざまな種類の荷物、さまざまな気象条件、または他の妥当な条件によって変化する可能性があります。 設定には初期の既定値が設定されていて、変更する必要はありませんが、必要に応じて、簡単かつ迅速に変更することができます。 この種類のプロパティは、"_書き込み可能なプロパティ_" と呼ばれます。

プロパティは単一の値です。 より複雑なデータ セットをデバイスに送信する必要がある場合は、コマンド (下記参照) の方がそれを処理する方法として適しています。

1. トラック ID のプロパティ機能を作成するには、 **[+ 機能の追加]** をクリックして新しい機能を次のように設定します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | トラック ID |
    | 名前 | TruckID |
    | 機能の種類 | プロパティ |
    | セマンティックの種類 | なし |
    | スキーマ | 文字列 |
    | 書き込み可能 | オフ |
    | 単位 | なし |

1. 最適温度のプロパティ機能を作成するには、 **[+ 機能の追加]** をクリックして新しい機能を次のように設定します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | 適温 |
    | 名前 | OptimalTemperature |
    | 機能の種類 | プロパティ |
    | セマンティックの種類 | なし |
    | スキーマ | 倍精度浮動小数点 |
    | 書き込み可能 | オン |
    | 単位 | 摂氏 |

#### <a name="task-7-add-commands"></a>タスク 7:コマンドの追加

コマンドは、IoT Central アプリのオペレーターによってリモート デバイスに送信されます。 コマンドは書き込み可能なプロパティに似ていますが、1 つのコマンドには任意の数の入力フィールドを含めることができるのに対し、書き込み可能なプロパティは 1 つの値に制限されています。

冷凍トラックの場合、次の 2 つのコマンドを追加する必要があります。荷物を顧客に配送するコマンドとトラックをベースに呼び戻すコマンドです。

1. 荷物を顧客に配送するコマンド機能を作成するには、 **[+ 機能の追加]** をクリックし、新しい機能を次のように作成します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | 顧客への配送 |
    | 名前 | GoToCustomer |
    | 機能の種類 | コマンド |

1. コマンド機能を展開して追加のフィールド オプションを表示し、 **[要求]** をオンに設定します。

    **[要求]** オプションをオンにすると、コマンドの詳細を入力できるようになります。

1. コマンド機能の **[要求]** 部分を完了するには、次のようにフィールド値を入力します。

    | フィールド | 値 |
    | --- | --- |
    | オフラインの場合にキューに入れる | オフ |
    | 要求 | オン |
    | 表示名 | 顧客 ID |
    | 名前 | CustomerID |
    | スキーマ | 整数 |

1. トラックを呼び戻すコマンド機能を作成するには、 **[+ 機能の追加]** をクリックし、新しい機能を次のように作成します。

    | フィールド | 値 |
    | --- | --- |
    | 表示名 | 呼び戻し |
    | 名前 | Recall |
    | 機能の種類 | コマンド |

    このコマンドには追加のパラメーターがないため、 **[要求]** をオフのままにします。

1. ページの先頭までスクロールして、 **[保存]** をクリックします。

    先に進む前に、さらに慎重にインターフェイスを確認してください。 インターフェイスを発行した後では、編集オプションが非常に限られてしまいます。 発行前に間違いがないようにしておくことが重要です。

    **[ビュー]** オプションで終了するメニューでデバイス テンプレートの名前をクリックすると、機能の概要が表示されます。

#### <a name="task-8-publish-the-template"></a>タスク 8:テンプレートを発行する

1. 前回保存してから変更を行った場合は、 **[保存]** をクリックします。

1. **RefrigeratedTruck** デバイス テンプレートの左上隅にある **[発行]** をクリックします。

    > **注**:確認を求めるポップアップ ダイアログが表示されたら、 **[発行]** をクリックします。

    注釈が **[ドラフト]** から、**[発行済み]** に変わります。

デバイス テンプレートの準備には、少しの注意と少しの時間が必要です。

次の演習では、デバイス テンプレートの機能を使って、コントローラーとオペレーターのビューを準備します。 デバイス テンプレート用のビューの作成は、デバイス テンプレートを発行する前または後に行うことができます。

### <a name="exercise-3-monitor-a-simulated-device"></a>演習 3:シミュレートされたデバイスを監視する

この演習を始めるには、デバイス テンプレートのすべての機能を示すビューを作成します。 その後、デバイス テンプレートを使用してデバイスを作成し、リモート デバイス アプリに必要な接続設定を記録します。

#### <a name="task-1-create-a-rich-view"></a>タスク 1:充実した内容のビューを作成する

1. **RefrigeratedTruck** デバイス テンプレートの左側のメニューで、 **[ビュー]** をクリックし、 **[デバイスの視覚化]** をクリックします。

1. **[ビューの作成]** ペインの **[タイルの追加]** で、 **[デバイスから始める]** を選びます。

1. 少し時間を取って、使用可能な **[テレメトリ]** 、 **[プロパティ]** 、 **[コマンド]** の一覧をそれぞれのドロップダウン リストで確認してください。

    これらは作成した機能であり、それぞれに選択チェックボックスがあります。

1. **[テレメトリの選択]** ドロップダウン メニューで、 **[場所]** をクリックして、 **[タイルの追加]** をクリックします。

    ビューはタイルを使って構成され、選んだタイルを配置したりサイズを変更したりできます。 [場所] タイルは世界地図上のトラックの場所を示し、これを最初に作成することで、地図のサイズを変更するための十分なスペースができます。

1. タイルの右下隅にマウス ポインターを置き、タイルの高さと幅が既定サイズの約 2 倍になるように、コーナーをドラッグします。

1. **[ビューの作成]** ペインの上部にある **[ビュー名]** テキスト ボックスに、「**トラック ビュー**」と入力します。

1. **[テレメトリの選択]** ドロップダウン メニューで、"**積載物の状態**" をクリックして、 **[タイルの追加]** をクリックします。

    **[タイルの追加]** ボタンは、 **[ビューの作成]** ペインの下部にあります。
 
1. 残りのテレメトリ機能ごとに前の手順を繰り返し、上から順に作業します。

    [場所] タイルが既に追加されていることを思い出してください。

1. 同じトップダウン プロセスを使用して、プロパティ機能を追加します。

    ラボで後ほど、ビューにタイルを配置する機会があります。 ここでは、リモート デバイスから送信されるすべてのテレメトリを確認するビューだけが必要です。

    ビューにコマンドを追加するオプションがありますが、追加する必要はありません。

1. 少し時間を取って、ビューのレイアウトを確認します。

    あちこちスクロールして、ビューのタイルを確認します。 タイルの内容を調べ、その情報をどのように使用するかを検討します。

1. タイルの内容の分析に基づいて、タイルの位置をすばやく調整します。

    今、これにあまり時間を費やさず、タイルをドラッグすることができ、ポータルでタイルがきれいに並べ替えられることに注意してください。

1. **[保存]** 、 **[戻る]** の順にクリックしてから、 **[発行]** をクリックします。

1. [発行] ダイアログ ボックスで、 **[ビュー]** エントリの横にある **[はい]** エントリを記録しておき、 **[発行]** をクリックします。

ビューは必要な数だけ作成でき、それぞれにフレンドリ名を付けることができます。

次のタスクでは、デバイス テンプレートからデバイスを作成します。

#### <a name="task-2-create-a-real-device"></a>タスク 2:実際のデバイスを作成する

IoT Central は、実際のセンサーを使用して物理デバイスに接続したり、アルゴリズムに基づいてデータを生成するシミュレートされたデバイスに接続したりできます。 どちらの場合も、IoT Central はリモート アプリがテレメトリ データを生成していることを理解し、どちらの方法でも、接続されたデバイスを "実際の" デバイスとして扱います。

1. 左側のナビゲーション メニューの **[接続]** で、 **[デバイス]** をクリックします。

1. **[デバイス]** ペインの **[すべてのデバイス]** で、**RefrigeratedTruck** をクリックします

    画面が更新され、選択したデバイス テンプレートが太字で表示されることに注意してください。 デバイス テンプレートが多数ある場合は、正しいデバイス テンプレートを使用していることを確認するのに役立ちます。

1. トップ メニューで **[+ 新規]** をクリックします。

1. **[新しいデバイスの作成]** ダイアログ ボックスの **[デバイス名]** で、**RefrigeratedTruck** がプレフィックスとして一覧に表示されていることを確認します。

    これは、適切なデバイス テンプレートを選択したことを確認するもう 1 つの機会です。

1. **[デバイス ID]** に、「**RefrigeratedTruck1**」と入力します

1. **[デバイス名]** に「**RefrigeratedTruck - 1**」と入力します

1. **[このデバイスをシミュレートしますか?]** セクションで、 **[いいえ]** が選ばれていることを確認します。

    IoT Central では、物理デバイスとシミュレートされたデバイスへの接続が同じ方法で処理されることを思い出してください。 どちらもリモート アプリで、どちらも本物です。 あなたはここで本物のトラックを構築します。 シミュレートされる "_実際の_" トラックです。

    **[このデバイスをシミュレートしますか?]** の値を **[はい]** に設定すると、テレメトリとしてランダムな値を出力するよう IoT Central に指示されます。 これらのランダム値は、デバイス テンプレートを検証する際に役立ちますが、このラボでは、シミュレートされたデバイス (トラック) を使用してテレメトリをシミュレートします。

1. **[新しいデバイスの作成]** ダイアログ ボックスで、 **[作成]** をクリックします。

    数秒待つと、デバイス リストに 1 つのエントリが表示されます。

    **[デバイスの状態]** が **登録済み** に設定されていることを確認します。 IoT Central アプリは、**デバイスの状態** が **プロビジョニング済み** の場合にのみ、デバイスへの接続を受け入れます。 このラボの後半では、デバイスをプロビジョニングする方法を示すコーディング タスクがあります。

1. **[デバイス名]** の **[冷蔵トラック - 1]** をクリックします。

    デバイスのライブ ビューが表示されるはずです ("**データが見つかりませんでした**" というメッセージが多数含まれます)。

1. デバイス ダッシュボードの **[RefrigeratedTruck - 1]** というタイトルの下にある **[コマンド]** をクリックします

    入力した 2 つのコマンドがすでにリストアップされていて、実行できる状態にあることを確認できます。

次のステップでは、リモート デバイスが IoT Central アプリと通信できるようにする SAS キーを作成します。

#### <a name="task-3-record-the-connection-keys"></a>タスク 3:接続キーを記録する

1. 右上のメニューで **[接続]** をクリックします。

1. **[認証の種類]** ドロップダウン リストで、 **[Shared Access Signature (SAS)]** が選ばれていることを確認します。

1. **[デバイス接続のグループ]** ダイアログ ボックスを使って、 **[ID スコープ]** 、 **[デバイス ID]** 、 **[主キー]** の値をコピーし、"**トラック接続.txt**" という名前のテキスト ファイルに保存します。

    メモ帳 (または別のテキスト エディター) を使用してこれらの値をテキスト ファイルに保存し、トラック接続.txt などのわかりやすい名前を付けます。

1. ダイアログ ボックスの下部で、 **[閉じる]** をクリックします。

ブラウザーで IoT ポータルを開いたままで待ちます。

### <a name="exercise-4-create-a-free-azure-maps-account"></a>演習 4:無料の Azure Maps アカウントを作成する

Azure Maps アカウントをまだ持っていない場合は、作成する必要があります。

1. 新しいブラウザー タブを開き、[Azure Maps](https://azure.microsoft.com/services/azure-maps/?azure-portal=true) に移動します。

1. 無料アカウントを作成するには、右上隅の **[無料アカウント]** をクリックして、表示される指示に従います。

    > **注**:または、このラボで使っている Azure サブスクリプションを使うこともできます。

1. 同じブラウザー タブで [Azure portal](https://portal.azure.com) に移動し、 **[リソース、サービス、ドキュメントの検索]** を使って **[Azure Maps アカウント]** ページに移動します。

1. **[Azure Maps アカウント]** ページで、 **[+ 作成]** を選びます。

1. **[Azure Maps アカウント リソースの作成]** ページの **[基本]** タブで、次の設定を指定します。

    | フィールド | 値 |
    | --- | --- |
    | Resource group | 新しいリソース グループの名前 **az-220-l20-RG** |
    | 名前 | AZ-220-MAPS |
    | Region | ラボ環境の近くにある Azure リージョンの名前 |
    | Pricing tier | S1 |

1. ライセンスとプライバシーに関する声明に同意することを確認するオプションを選びます。

1. **[Review + create]\(レビュー + 作成\)** をクリックします。 

1. **[確認と作成]** タブで、**[作成]** をクリックします。

    デプロイには 1、2 分かかる場合があります。

1. Azure Maps アカウントが作成されたら、 **[リソースに移動]** をクリックします。

1. **AZ-220-MAPS** ページの左側のナビゲーション メニューの **[設定]** で、 **[認証]** をクリックします。

1. **[認証]** ページで、 **[主キー]** の値をコピーし、先ほど作成した "トラック接続.txt" テキスト ファイルに貼り付けます。

    > **注**:(Azure Maps の) 主キーが正しくて動作することを確認する場合は、次の HTML コードを .html ファイルとして保存し、`<your Azure Maps subscription key>` プレースホルダーを Azure portal からコピーした主キーの値に置き換えてから、Web ブラウザーにファイルを読み込みます。 設定が正しい場合は、ブラウザー ページに世界地図が表示されます。

    ```html
    <!DOCTYPE html>
    <html>

    <head>
        <title>Map</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

        <!-- Add a reference to the Azure Maps Services lab JavaScript file. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas-service.min.js"></script>

        <script>
            function GetMap() {
                //Instantiate a map object
                var map = new atlas.Map("myMap", {
                    //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                    authOptions: {
                        authType: 'subscriptionKey',
                        subscriptionKey: '<your Azure Maps subscription key>'
                    }
                });
            }
        </script>
        <style>
            html,
            body {
                width: 100%;
                height: 100%;
                padding: 0;
                margin: 0;
            }

            #myMap {
                width: 100%;
                height: 100%;
            }
        </style>
    </head>

    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>

    </html>
    ```

これで、最初の IoT Central アプリを実際のデバイスに接続するための準備手順が完了しました。 お疲れさまでした。

次の手順では、デバイス アプリを作成します。

### <a name="exercise-5-create-a-programming-project-for-a-real-device"></a>エクササイズ 5:実際のデバイスのプログラミング プロジェクトを作成する

このタスクでは、冷蔵トラックでセンサー デバイスをシミュレートするプログラミング プロジェクトを作成します。 このシミュレーションにより、物理デバイスを必要とするかなり前にコードをテストできます。

デバイス アプリと IoT Central アプリの間の通信コードは物理デバイスとトラックの間の通信コードと同じであるため、IoT Central はこのシミュレートされたデバイスを "リアル" として扱います。 つまり、冷凍トラックの会社を経営している場合、このタスクのコードと同様のシミュレートされたコードから始めることになります。 コードが正常に動作することを確認した後、シミュレーション固有のコードは、センサー データを受信するコードに置き換えられます。 この制限された更新により、次のコードを書くことが貴重な経験となります。

#### <a name="task-1-create-the-device-app"></a>タスク 1:デバイス アプリを作成する

Visual Studio Code を使用して、デバイス センサー アプリをビルドします。

1. Visual Studio Code の新しいインスタンスを開きます。

1. **[ファイル]** メニューで、 **[フォルダーを開く]** をクリックします。

1. **[フォルダーを開く]** ダイアログ ボックスの上部にある **[新しいフォルダー]** をクリックし、「**RefrigeratedTruck**」と入力して **Enter** キーを押します。

    RefrigeratedTruck フォルダーは任意の場所に作成できます。

1. **[RefrigeratedTruck]** をクリックし、 **[フォルダの選択]** をクリックします。

    Visual Studio Code の **[エクスプローラー]** ペインが開くはずです。

1. **[表示]** メニューで、統合ターミナルを開くには、 **[ターミナル]** をクリックします。

    ターミナル コマンド プロンプトに、RefrigeratedTruck フォルダーへのパスが表示されるはずです。 次のコマンドは現在のフォルダーで実行されるため、これは重要です。

1. ターミナル コマンド プロンプトで、新しいコンソール アプリを作成するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet new console
    ```

    このコマンドにより、Program.cs ファイルがプロジェクト ファイルと共にフォルダー内に作成されます。

1. ターミナル コマンド プロンプトで、プリが、必要な .NET パッケージにアクセスできることを確認するには、次のコマンドを入力します。

    ```cmd/sh
    dotnet restore
    ```

1. ターミナル コマンド プロンプトで、必要なライブラリをインストールするには、次のコマンドを入力します。

    ```CLI
    dotnet add package AzureMapsRestToolkit -v 3.0.0
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Microsoft.Azure.Devices.Provisioning.Client
    dotnet add package Microsoft.Azure.Devices.Provisioning.Transport.Mqtt
    dotnet add package System.Text.Json
    ```

1. **EXPLORER** ペインで、**Program.cs** をクリックします。

1. [コード エディター] ペインで、Program.cs ファイルの内容を削除します。

これで、次のコードを追加する準備が整いました。

#### <a name="task-2-write-the-device-app"></a>タスク 2:デバイス アプリを作成する

このタスクでは、冷凍トラック用のシミュレートされたデバイス アプリを一度に 1 セクションずつ構築します。 各セクションについて簡単に説明します。

このプロセスをできるだけ簡単にするには、ここにリストされている順序で、コードの各セクションをファイルの末尾に追加する必要があります。

1. [コード エディター] ペインで、必要な `using` ステートメントを追加するには、次のコードを入力します。

   ```cs
    using System;
    using System.Text.Json;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Microsoft.Azure.Devices.Provisioning.Client;
    using Microsoft.Azure.Devices.Provisioning.Client.Transport;
    using AzureMapsToolkit;
    using AzureMapsToolkit.Common;
    ```

    これらの `using` ステートメントで、Azure IoT Central や Azure Maps などの、コードが使用するリソースに簡単にアクセスできるようになります。

1. コード エディター ペインで、名前空間、クラス、およびグローバル変数を追加するには、次のコードを入力します。

    ```cs
    namespace refrigerated_truck
    {
        class Program
        {
            enum StateEnum
            {
                ready,
                enroute,
                delivering,
                returning,
                loading,
                dumping
            };
            enum ContentsEnum
            {
                full,
                melting,
                empty
            }
            enum FanEnum
            {
                on,
                off,
                failed
            }

            // Azure maps service globals.
            static AzureMapsServices azureMapsServices;

            // Telemetry globals.
            const int intervalInMilliseconds = 5000;        // Time interval required by wait function.

            // Refrigerated truck globals.
            static int truckNum = 1;
            static string truckIdentification = "Truck number " + truckNum;

            const double deliverTime = 600;                 // Time to complete delivery, in seconds.
            const double loadingTime = 800;                 // Time to load contents.
            const double dumpingTime = 400;                 // Time to dump melted contents.
            const double tooWarmThreshold = 2;              // Degrees C that is too warm for contents.
            const double tooWarmtooLong = 60;               // Time in seconds for contents to start melting if temps are above threshold.


            static double timeOnCurrentTask = 0;            // Time on current task in seconds.
            static double interval = 60;                    // Simulated time interval in seconds.
            static double tooWarmPeriod = 0;                // Time that contents are too warm in seconds.
            static double tempContents = -2;                // Current temp of contents in degrees C.
            static double baseLat = 47.644702;              // Base position latitude.
            static double baseLon = -122.130137;            // Base position longitude.
            static double currentLat;                       // Current position latitude.
            static double currentLon;                       // Current position longitude.
            static double destinationLat;                   // Destination position latitude.
            static double destinationLon;                   // Destination position longitude.

            static FanEnum fan = FanEnum.on;                // Cooling fan state.
            static ContentsEnum contents = ContentsEnum.full;    // Truck contents state.
            static StateEnum state = StateEnum.ready;       // Truck is full and ready to go!
            static double optimalTemperature = -5;         // Setting - can be changed by the operator from IoT Central.

            const string noEvent = "none";
            static string eventText = noEvent;              // Event text sent to IoT Central.

            static double[,] customer = new double[,]
            {
                // Lat/lon position of customers.
                // Gasworks Park
                {47.645892, -122.336954},

                // Golden Gardens Park
                {47.688741, -122.402965},

                // Seward Park
                {47.551093, -122.249266},

                // Lake Sammamish Park
                {47.555698, -122.065996},

                // Marymoor Park
                {47.663747, -122.120879},

                // Meadowdale Beach Park
                {47.857295, -122.316355},

                // Lincoln Park
                {47.530250, -122.393055},

                // Gene Coulon Park
                {47.503266, -122.200194},

                // Luther Bank Park
                {47.591094, -122.226833},

                // Pioneer Park
                {47.544120, -122.221673 }
            };

            static double[,] path;                          // Lat/lon steps for the route.
            static double[] timeOnPath;                     // Time in seconds for each section of the route.
            static int truckOnSection;                      // The current path section the truck is on.
            static double truckSectionsCompletedTime;       // The time the truck has spent on previous completed sections.
            static Random rand;

            // IoT Central global variables.
            static DeviceClient s_deviceClient;
            static CancellationTokenSource cts;
            static string GlobalDeviceEndpoint = "global.azure-devices-provisioning.net";
            static TwinCollection reportedProperties = new TwinCollection();

            // User IDs.
            static string ScopeID = "<your Scope ID>";
            static string DeviceID = "<your Device ID>";
            static string PrimaryKey = "<your device Primary Key>";
            static string AzureMapsKey = "<your Azure Maps Subscription Key>";
    ```

    追加するコードは他にもありますが、入力したプレースホルダー値を置き換えるタイミングです。 これらはすべて、ラボ中に追加したテキスト ファイルで使用できる必要があります。

1. 以前に保存した冷蔵トラック1 および Azure Maps アカウント情報を含むテキスト ファイルを開きます。

1. [コード エディター] ペインで、プレースホルダーの値をテキスト ファイルからの対応する値に置き換えます。

    これらの値をコードで更新すると、アプリのビルドに戻ることができます。

1. [コード エディター] ペインで、Azure Maps 経由でルートを取得するために使用するメソッドを追加するには、次のコードを入力します。

    ```cs
    static double Degrees2Radians(double deg)
    {
        return deg * Math.PI / 180;
    }

    // Returns the distance in meters between two locations on Earth.
    static double DistanceInMeters(double lat1, double lon1, double lat2, double lon2)
    {
        var dlon = Degrees2Radians(lon2 - lon1);
        var dlat = Degrees2Radians(lat2 - lat1);

        var a = (Math.Sin(dlat / 2) * Math.Sin(dlat / 2)) + Math.Cos(Degrees2Radians(lat1)) * Math.Cos(Degrees2Radians(lat2)) * (Math.Sin(dlon / 2) * Math.Sin(dlon / 2));
        var angle = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        var meters = angle * 6371000;
        return meters;
    }

    static bool Arrived()
    {
        // If the truck is within 10 meters of the destination, call it good.
        if (DistanceInMeters(currentLat, currentLon, destinationLat, destinationLon) < 10)
            return true;
        return false;
    }

    static void UpdatePosition()
    {
        while ((truckSectionsCompletedTime + timeOnPath[truckOnSection] < timeOnCurrentTask) && (truckOnSection < timeOnPath.Length - 1))
        {
            // Truck has moved onto the next section.
            truckSectionsCompletedTime += timeOnPath[truckOnSection];
            ++truckOnSection;
        }

        // Ensure remainder is 0 to 1, as interval may take count over what is needed.
        var remainderFraction = Math.Min(1, (timeOnCurrentTask - truckSectionsCompletedTime) / timeOnPath[truckOnSection]);

        // The path should be one entry longer than the timeOnPath array.
        // Find how far along the section the truck has moved.
        currentLat = path[truckOnSection, 0] + remainderFraction * (path[truckOnSection + 1, 0] - path[truckOnSection, 0]);
        currentLon = path[truckOnSection, 1] + remainderFraction * (path[truckOnSection + 1, 1] - path[truckOnSection, 1]);
    }

    static void GetRoute(StateEnum newState)
    {
        // Set the state to ready, until the new route arrives.
        state = StateEnum.ready;

        var req = new RouteRequestDirections
        {
            Query = FormattableString.Invariant($"{currentLat},{currentLon}:{destinationLat},{destinationLon}")
        };
        var directions = azureMapsServices.GetRouteDirections(req).Result;

        if (directions.Error != null || directions.Result == null)
        {
            // Handle any error.
            redMessage("Failed to find map route");
        }
        else
        {
            int nPoints = directions.Result.Routes[0].Legs[0].Points.Length;
            greenMessage($"Route found. Number of points = {nPoints}");

            // Clear the path. Add two points for the start point and destination.
            path = new double[nPoints + 2, 2];
            int c = 0;

            // Start with the current location.
            path[c, 0] = currentLat;
            path[c, 1] = currentLon;
            ++c;

            // Retrieve the route and push the points onto the array.
            for (var n = 0; n < nPoints; n++)
            {
                var x = directions.Result.Routes[0].Legs[0].Points[n].Latitude;
                var y = directions.Result.Routes[0].Legs[0].Points[n].Longitude;
                path[c, 0] = x;
                path[c, 1] = y;
                ++c;
            }

            // Finish with the destination.
            path[c, 0] = destinationLat;
            path[c, 1] = destinationLon;

            // Store the path length and time taken, to calculate the average speed.
            var meters = directions.Result.Routes[0].Summary.LengthInMeters;
            var seconds = directions.Result.Routes[0].Summary.TravelTimeInSeconds;
            var pathSpeed = meters / seconds;

            double distanceApartInMeters;
            double timeForOneSection;

            // Clear the time on path array. The path array is 1 less than the points array.
            timeOnPath = new double[nPoints + 1];

            // Calculate how much time is required for each section of the path.
            for (var t = 0; t < nPoints + 1; t++)
            {
                // Calculate distance between the two path points, in meters.
                distanceApartInMeters = DistanceInMeters(path[t, 0], path[t, 1], path[t + 1, 0], path[t + 1, 1]);

                // Calculate the time for each section of the path.
                timeForOneSection = distanceApartInMeters / pathSpeed;
                timeOnPath[t] = timeForOneSection;
            }
            truckOnSection = 0;
            truckSectionsCompletedTime = 0;
            timeOnCurrentTask = 0;

            // Update the state now the route has arrived. One of: enroute or returning.
            state = newState;
        }
    }
    ```

    > **注**:上記のコード内の重要な呼び出しは `var directions = azureMapsServices.GetRouteDirections(req).Result;` です。 `directions` 構造体は複雑です。 このメソッドにブレークポイントを設定し、`directions` の内容を調べることを検討してください。

1. [コード エディター] ペインで、顧客に配信するダイレクト メソッドを追加するには、次のコードを入力します。

    ```cs
    static Task<MethodResponse> CmdGoToCustomer(MethodRequest methodRequest, object userContext)
    {
        try
        {
            // Pick up variables from the request payload, with the name specified in IoT Central.
            var payloadString = Encoding.UTF8.GetString(methodRequest.Data);
            int customerNumber = Int32.Parse(payloadString);

            // Check for a valid key and customer ID.
            if (customerNumber >= 0 && customerNumber < customer.Length)
            {
                switch (state)
                {
                    case StateEnum.dumping:
                    case StateEnum.loading:
                    case StateEnum.delivering:
                        eventText = "Unable to act - " + state;
                        break;

                    case StateEnum.ready:
                    case StateEnum.enroute:
                    case StateEnum.returning:
                        if (contents == ContentsEnum.empty)
                        {
                            eventText = "Unable to act - empty";
                        }
                        else
                        {
                            // Set event only when all is good.
                            eventText = "New customer: " + customerNumber.ToString();

                            destinationLat = customer[customerNumber, 0];
                            destinationLon = customer[customerNumber, 1];

                            // Find route from current position to destination, storing route.
                            GetRoute(StateEnum.enroute);
                        }
                        break;
                }

                // Acknowledge the direct method call with a 200 success message.
                string result = "{\"result\":\"Executed direct method: " + methodRequest.Name + "\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
            }
            else
            {
                eventText = $"Invalid customer: {customerNumber}";

                // Acknowledge the direct method call with a 400 error message.
                string result = "{\"result\":\"Invalid customer\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
            }
        }
        catch
        {
            // Acknowledge the direct method call with a 400 error message.
            string result = "{\"result\":\"Invalid call\"}";
            return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
        }
    }
    ```

    > **注**:デバイスが適切な状態ではない場合、デバイスは競合で応答します。 コマンド自体は、メソッドの終了時に確認されます。 次の手順に引き継がれる呼び戻しコマンドも、同様の方法で処理を行います。

1. コード エディター ペインで、呼び戻しダイレクト メソッドを追加するには、次のコードを入力します。

    ```cs
    static void ReturnToBase()
    {
        destinationLat = baseLat;
        destinationLon = baseLon;

        // Find route from current position to base, storing route.
        GetRoute(StateEnum.returning);
    }
    static Task<MethodResponse> CmdRecall(MethodRequest methodRequest, object userContext)
    {
        switch (state)
        {
            case StateEnum.ready:
            case StateEnum.loading:
            case StateEnum.dumping:
                eventText = "Already at base";
                break;

            case StateEnum.returning:
                eventText = "Already returning";
                break;

            case StateEnum.delivering:
                eventText = "Unable to recall - " + state;
                break;

            case StateEnum.enroute:
                ReturnToBase();
                break;
        }

        // Acknowledge the command.
        if (eventText == noEvent)
        {
            // Acknowledge the direct method call with a 200 success message.
            string result = "{\"result\":\"Executed direct method: " + methodRequest.Name + "\"}";
            return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
        }
        else
        {
            // Acknowledge the direct method call with a 400 error message.
            string result = "{\"result\":\"Invalid call\"}";
            return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
        }
    }
    ```

1. トラックのシミュレーションを各時間サイクルで更新するメソッドを追加するために、[コード エディタ] ペインで次のコードを入力します。

    ```cs
    static double DieRoll(double max)
    {
        return rand.NextDouble() * max;
    }

    static void UpdateTruck()
    {
        if (contents == ContentsEnum.empty)
        {
            // Turn the cooling system off, if possible, when the contents are empty.
            if (fan == FanEnum.on)
            {
                fan = FanEnum.off;
            }
            tempContents += -2.9 + DieRoll(6);
        }
        else
        {
            // Contents are full or melting.
            if (fan != FanEnum.failed)
            {
                if (tempContents < optimalTemperature - 5)
                {
                    // Turn the cooling system off, as contents are getting too cold.
                    fan = FanEnum.off;
                }
                else
                {
                    if (tempContents > optimalTemperature)
                    {
                        // Temp getting higher, turn cooling system back on.
                        fan = FanEnum.on;
                    }
                }

                // Randomly fail the cooling system.
                if (DieRoll(100) < 1)
                {
                    fan = FanEnum.failed;
                }
            }

            // Set the contents temperature. Maintaining a cooler temperature if the cooling system is on.
            if (fan == FanEnum.on)
            {
                tempContents += -3 + DieRoll(5);
            }
            else
            {
                tempContents += -2.9 + DieRoll(6);
            }

            // If the temperature is above a threshold, count the seconds this is occurring, and melt the contents if it goes on too long.
            if (tempContents >= tooWarmThreshold)
            {
                // Contents are warming.
                tooWarmPeriod += interval;

                if (tooWarmPeriod >= tooWarmtooLong)
                {
                    // Contents are melting.
                    contents = ContentsEnum.melting;
                }
            }
            else
            {
                // Contents are cooling.
                tooWarmPeriod = Math.Max(0, tooWarmPeriod - interval);
            }
        }

        timeOnCurrentTask += interval;

        switch (state)
        {
            case StateEnum.loading:
                if (timeOnCurrentTask >= loadingTime)
                {
                    // Finished loading.
                    state = StateEnum.ready;
                    contents = ContentsEnum.full;
                    timeOnCurrentTask = 0;

                    // Turn on the cooling fan.
                    // If the fan is in a failed state, assume it has been fixed, as it is at the base.
                    fan = FanEnum.on;
                    tempContents = -2;
                }
                break;

            case StateEnum.ready:
                timeOnCurrentTask = 0;
                break;

            case StateEnum.delivering:
                if (timeOnCurrentTask >= deliverTime)
                {
                    // Finished delivering.
                    contents = ContentsEnum.empty;
                    ReturnToBase();
                }
                break;

            case StateEnum.returning:

                // Update the truck position.
                UpdatePosition();

                // Check to see if the truck has arrived back at base.
                if (Arrived())
                {
                    switch (contents)
                    {
                        case ContentsEnum.empty:
                            state = StateEnum.loading;
                            break;

                        case ContentsEnum.full:
                            state = StateEnum.ready;
                            break;

                        case ContentsEnum.melting:
                            state = StateEnum.dumping;
                            break;
                    }
                    timeOnCurrentTask = 0;
                }
                break;

            case StateEnum.enroute:

                // Move the truck.
                UpdatePosition();

                // Check to see if the truck has arrived at the customer.
                if (Arrived())
                {
                    state = StateEnum.delivering;
                    timeOnCurrentTask = 0;
                }
                break;

            case StateEnum.dumping:
                if (timeOnCurrentTask >= dumpingTime)
                {
                    // Finished dumping.
                    state = StateEnum.loading;
                    contents = ContentsEnum.empty;
                    timeOnCurrentTask = 0;
                }
                break;
        }
    }
    ```

    > **注**:この関数は、期間ごとに呼び出されます。 実際の期間は、5 秒に設定されます。ただし、"_シミュレートされた時間_" (この関数が呼び出されるたびに経過させるよう指定したシミュレートされた秒数) はグローバルの `static double interval = 60` によって設定されます。 この値を 60 に設定すると、シミュレーションは、60 を 5 で割った速度、つまりリアルタイムの 12 倍の速度で実行されることを意味します。 シミュレートされる時間を下げるには、`interval` を、たとえば、30 に下げます (リアルタイムの 6 倍の速度で実行するシミュレーションの場合)。 `interval` を 5 に設定すると、リアルタイムでシミュレーションが実行されます。 この速度は現実的ではありますが、配送先の顧客への実際の運転時間を考慮すると、少し遅いかもしれません。

1. トラックのテレメトリを送信する (もし発生していたらイベントも送信する) メソッドを追加するために、[コード エディター] ペインで次のコードを入力します。

    ```cs
    static void colorMessage(string text, ConsoleColor clr)
    {
        Console.ForegroundColor = clr;
        Console.WriteLine(text);
        Console.ResetColor();
    }
    static void greenMessage(string text)
    {
        colorMessage(text, ConsoleColor.Green);
    }

    static void redMessage(string text)
    {
        colorMessage(text, ConsoleColor.Red);
    }

    static async void SendTruckTelemetryAsync(Random rand, CancellationToken token)
    {
        while (true)
        {
            UpdateTruck();

            // Create the telemetry JSON message.
            var telemetryDataPoint = new
            {
                ContentsTemperature = Math.Round(tempContents, 2),
                TruckState = state.ToString(),
                CoolingSystemState = fan.ToString(),
                ContentsState = contents.ToString(),
                Location = new { lon = currentLon, lat = currentLat },
                Event = eventText,
            };
            var telemetryMessageString = JsonSerializer.Serialize(telemetryDataPoint);
            var telemetryMessage = new Message(Encoding.ASCII.GetBytes(telemetryMessageString));

            // Clear the events, as the message has been sent.
            eventText = noEvent;

            Console.WriteLine($"\nTelemetry data: {telemetryMessageString}");

            // Bail if requested.
            token.ThrowIfCancellationRequested();

            // Send the telemetry message.
            await s_deviceClient.SendEventAsync(telemetryMessage);
            greenMessage($"Telemetry sent {DateTime.Now.ToShortTimeString()}");

            await Task.Delay(intervalInMilliseconds);
        }
    }
    ```

    > **注**:`SendTruckTelemetryAsync` は、テレメトリ、状態、およびイベントの IoT Central への送信を処理する重要な関数です。 JSON 文字列を使用してデータを送信することに注意してください。

1. 設定とプロパティを処理するコードを追加するために、[コード エディター] ペインで次のコードを入力します。

    ```cs
    static async Task SendDevicePropertiesAsync()
    {
        reportedProperties["TruckID"] = truckIdentification;
        await s_deviceClient.UpdateReportedPropertiesAsync(reportedProperties);
        greenMessage($"Sent device properties: {JsonSerializer.Serialize(reportedProperties)}");
    }
    static async Task HandleSettingChanged(TwinCollection desiredProperties, object userContext)
    {
        string setting = "OptimalTemperature";
        if (desiredProperties.Contains(setting))
        {
            BuildAcknowledgement(desiredProperties, setting);
            optimalTemperature = (int) desiredProperties[setting];
            greenMessage($"Optimal temperature updated: {optimalTemperature}");
        }
        await s_deviceClient.UpdateReportedPropertiesAsync(reportedProperties);
    }

    static void BuildAcknowledgement(TwinCollection desiredProperties, string setting)
    {
        reportedProperties[setting] = new
        {
            value = desiredProperties[setting],
            status = "completed",
            desiredVersion = desiredProperties["$version"],
            message = "Processed"
        };
    }
    ```

    アプリに追加できる設定とプロパティはそれぞれ 1 つだけです。 さらに必要な場合は、簡単に追加できます。

    > **注**:コードのこのセクションは、IoT Central と通信するほとんどの C# アプリに一般的なものです。 追加のプロパティまたは設定を追加するには、`reportedProperties` に追加するか、新しい設定文字列を作成して、`desiredProperties` をそれぞれ確認します。 通常は、他のコードの変更は必要ありません。

1. `Main` 関数を追加するには、[コード エディター] ペインで次のコードを入力します。

    ```cs
            static void Main(string[] args)
            {

                rand = new Random();
                colorMessage($"Starting {truckIdentification}", ConsoleColor.Yellow);
                currentLat = baseLat;
                currentLon = baseLon;

                // Connect to Azure Maps.
                azureMapsServices = new AzureMapsServices(AzureMapsKey);

                try
                {
                    using (var security = new SecurityProviderSymmetricKey(DeviceID, PrimaryKey, null))
                    {
                        DeviceRegistrationResult result = RegisterDeviceAsync(security).GetAwaiter().GetResult();
                        if (result.Status != ProvisioningRegistrationStatusType.Assigned)
                        {
                            Console.WriteLine("Failed to register device");
                            return;
                        }
                        IAuthenticationMethod auth = new DeviceAuthenticationWithRegistrySymmetricKey(result.DeviceId, (security as SecurityProviderSymmetricKey).GetPrimaryKey());
                        s_deviceClient = DeviceClient.Create(result.AssignedHub, auth, TransportType.Mqtt);
                    }
                    greenMessage("Device successfully connected to Azure IoT Central");

                    SendDevicePropertiesAsync().GetAwaiter().GetResult();

                    Console.Write("Register settings changed handler...");
                    s_deviceClient.SetDesiredPropertyUpdateCallbackAsync(HandleSettingChanged, null).GetAwaiter().GetResult();
                    Console.WriteLine("Done");

                    cts = new CancellationTokenSource();

                    // Create a handler for the direct method calls.
                    s_deviceClient.SetMethodHandlerAsync("GoToCustomer", CmdGoToCustomer, null).Wait();
                    s_deviceClient.SetMethodHandlerAsync("Recall", CmdRecall, null).Wait();

                    SendTruckTelemetryAsync(rand, cts.Token);

                    Console.WriteLine("Press any key to exit...");
                    Console.ReadKey();
                    cts.Cancel();
                }
                catch (Exception ex)
                {
                    Console.WriteLine();
                    Console.WriteLine(ex.Message);
                }
            }


            public static async Task<DeviceRegistrationResult> RegisterDeviceAsync(SecurityProviderSymmetricKey security)
            {
                Console.WriteLine("Register device...");

                using (var transport = new ProvisioningTransportHandlerMqtt(TransportFallbackType.TcpOnly))
                {
                    ProvisioningDeviceClient provClient =
                              ProvisioningDeviceClient.Create(GlobalDeviceEndpoint, ScopeID, security, transport);

                    Console.WriteLine($"RegistrationID = {security.GetRegistrationID()}");

                    Console.Write("ProvisioningClient RegisterAsync...");
                    DeviceRegistrationResult result = await provClient.RegisterAsync();

                    Console.WriteLine($"{result.Status}");

                    return result;
                }
            }
        }
    }
    ```

    > **注**:ダイレクト メソッドは、`s_deviceClient.SetMethodHandlerAsync("cmdGoTo", CmdGoToCustomer, null).Wait();` などのステートメントを使用して、クライアントで設定されます。

1. **[ファイル]** メニューの **[保存]** をクリックします。

    シミュレートされたデバイス アプリが完成したので、次は作成したコードをテストすることを考えます。

### <a name="exercise-6-test-your-iot-central-device"></a>演習 6: IoT Central デバイスをテストする

この演習では、作成したすべての可動部品が意図したとおりに連携するかどうかを検査します。

冷凍トラック デバイスを完全にテストするには、テストを次のようにいくつもの細かいチェックに分割すると便利です。

* デバイス アプリが Azure IoT Central に接続されている。

* テレメトリ関数が指定された間隔でデータを送信している。

* IoT Central によってデータが正しく取得されている。

* 指定された顧客にトラックを送るコマンドが、想定どおりに動作している。

* トラックを再呼び出しするコマンドは、想定どおりに動作します。

* カスタマー イベントと競合イベントが正しく送信されていることを確認する。

* トラックのプロパティを確認し、適温を変更する。

この一覧に加えて、調査することのできるエッジケースもあります。 このようなケースの 1 つに、トラックの荷物が解凍しはじめたときにどうなるのかということがあります。 このシミュレーションでは、前のタスクで乱数をコードに使用することで、そのような状態の発生を偶然に任せています。

#### <a name="task-1-prepare-iot-central-and-your-simulated-device"></a>タスク 1:IoT Central とシミュレートされたデバイスを準備する

1. Azure IoT Central アプリがブラウザーで開いていることを確認します。

    IoT Central とデバイス間の接続テストを始める前に、Azure IoT Central アプリがブラウザーで開いていることを確認します。 RefrigeratedTruck - 1 ビューの [コマンド] タブで、アプリを開いたままにしていました。 必要に応じて、ブラウザーで [Azure IoT Central](https://apps.azureiotcentral.com/?azure-portal=true) をもう一度開きます。

1. Visual Studio Code のターミナル コマンド プロンプトで、次のコマンドを実行してプログラムを実行します。

    ```cmd/sh
    dotnet run
    ```

1. 出力が [ターミナル] ペインに送信されることを確認します。

    ターミナル コンソールに、次のテキストで出力が表示されるはずです。**Starting Truck number 1** (トラック番号 1 を開始しています)。

1. テキストが以下であることを確認する:**Starting Truck number 1** が表示されることを確認します。

    > **注**:すべてが期待どおりに動作すれば、定義したいくつかのテスト ケースを迅速に検査できるでしょう。

    この後のタスクも、引き続き [ターミナル] ペインを確認します。

#### <a name="task-2-confirm-the-device-app-connects-to-azure-iot-central"></a>タスク 2:デバイス アプリが Azure IoT Central に接続されていることを確認する

1. [ターミナル] ペインで **デバイスが正常に Azure IoT Central に接続されました** と表示されることを確認します。

    コンソールの次の行の 1 つが "**Device successfully connected to Azure IoT Central**" (デバイスが Azure IoT Central に正常に接続されています) であれば、接続が確立されています。 このメッセージが表示されない場合は、通常、IoT Central アプリが実行されていないか、接続キー文字列が正しくないことを示しています。

1. "接続された" というメッセージの後に、設定とプロパティが正常に送信されたことを確認するテキストが続いていることを確認します。

    すべてが正常に行われた場合、2 番目のテスト (タスク 3) に進みます。

#### <a name="task-3-confirm-the-telemetry-functions-send-data-on-the-specified-interval"></a>タスク 3:テレメトリ関数で、指定された間隔でデータが送信されていることを確認する

1. テレメトリ データが送信されていることを確認します。

    コンソール メッセージが、荷物の温度と一緒に、5 秒ごとに表示されます。

1. しばらくの間テレメトリを見て、このラボのメイン テストの心の準備をしてください!

#### <a name="task-4-confirm-the-data-is-picked-up-correctly-by-iot-central"></a>タスク 4:IoT Central によりデータが正しく取得されていることを確認する

1. Azure IoT Central アプリが含まれているブラウザーの画面に切り替えます。

1. **RefrigeratedTruck - 1** ビューで、 **[トラック ビュー]** をクリックします。

    ご使用の RefrigeratedTruck デバイスが IoT Central で選択されていない場合は、次の手順を実行します。

    * 左側のナビゲーション メニューで、 **[デバイス]** をクリックします。
    * デバイスの一覧で、**RefrigeratedTruck - 1** をクリックします。
    * ビューで、 **[トラック ビュー]** が選ばれていることを確認します。

1. データが **RefrigeratedTruck - 1** ダッシュボードに表示されていることを確認します。

    たとえば、トラック ID タイルには "トラック番号 1" が表示され、トラック状態タイルには "準備完了" と時刻の値が表示されます。

1. ダッシュボードで、 **[荷物の温度]** タイルを見つけます。

    > **注**:一般的に許容可能な温度 (ゼロ℃近く) の期間が過ぎると、数値は上昇し始めます。

1. デバイス アプリから送信される温度が、IoT Central アプリのテレメトリ ビューに表示されているデータと一致していることを確認します。

    Visual Studio Code のターミナル ウィンドウの最新の値を、[コンテンツ温度] グラフに表示された最新の値と比較します。

1. トラックとその積載物が予期した状態であることを確認するには、状態タイルを確認します。**トラックの状態**、 **冷却システムの状態**、および **積載物の状態**。

1. デバイスの **[場所]** マップ ビューをチェックします。

    暗い円とリング (米国シアトルの近く) は、ベースの場所にいるトラックを示します。 少し縮小する必要があり、地図情報が正しく読み込まれて表示されるには時間がかかる場合があります。

    トラックは、そのベースに正しい状態で配置し、コマンドを待っている必要があります。

    次のタスクでは、アプリのテストを完了します。

#### <a name="task-5-confirm-the-command-to-send-the-truck-to-a-specified-customer-works-as-expected"></a>タスク 5: 指定された顧客にトラックを送るコマンドが想定どおりに動作することを確認する

1. **[冷蔵トラック - 1]** ダッシュボードで、ダッシュボードのタイトルのすぐ下にある **[コマンド]** をクリックします。

1. **[顧客 ID]** に「**1**」と入力します

    "0" から "9" までの値はすべて、有効な顧客 ID です

1. コマンドを発行するには、 **[実行]** をクリックします。

1. **トラックビュー** に戻ります。

    デバイス アプリのコンソールに、**New customer** イベントが表示されます。

   > **注**:"**Access denied due to invalid subscription key**" (サブスクリプション キーが無効なため、アクセスが拒否されました) というテキストを含むメッセージが表示された場合は、Azure Maps へのサブスクリプション キーを確認します。

1. ダッシュボードの **[場所]** タイルで、トラックが進行中であることを確認します。

    2 つのアプリが同期されるまで、しばらく待つ必要があります。

1. イベント タイルでイベント テキストが更新されていることを確認します。

1. 少し時間をとって、マップの更新とトラックの荷物の配送状態を確認します。

#### <a name="task-6-confirm-the-command-to-recall-the-truck-works-as-expected"></a>タスク 6: トラックを呼び戻すコマンドが想定どおり動作することを確認する

1. トラックがベースに戻り、荷物を再度積むと、状態が **準備完了** になります。

    別の配送コマンドを発行してみます。 別の顧客 ID を選択してください。

1. トラックが顧客に到達する前に呼び戻しコマンドを発行します。

1. トラックがこのコマンドに応答することを確認します。

#### <a name="task-7-check-customer-and-conflict-events-are-transmitted-correctly"></a>タスク 7: 顧客および競合イベントが正しく送信されていることを確認する

競合イベントをテストするために、意味がないと思われるコマンドを送信することができます。

1. トラックがベースにいる状態で、呼び戻しコマンドを発行します。

1. トラックが、"既にベースにある" イベントで応答することを確認します。

#### <a name="task-8-check-the-truck-properties-and-change-the-optimal-temperature"></a>タスク 8: トラックのプロパティを確認し、適温を変更する

1. **[トラック ID]** タイルに、 **[トラック番号 1]** が表示されていることを確認します。

    このプロパティは、テストするのが最も簡単なものの 1 つです。

    書き込み可能なプロパティのテストはより複雑です。**最適温度** プロパティは書き込み可能なプロパティなので、これをテストします。

1. 左側のナビゲーション メニューの **[管理]** で、 **[ジョブ]** をクリックします。

1. **[ジョブ]** ページの左上隅で、 **[+ 新規]** をクリックします。

1. **[ジョブの構成]** ページの **[名前]** テキスト ボックスに「**最適温度を -10 に設定する**」と入力します

1. **[デバイス グループ]** ドロップダウン リストで、 **[RefrigeratedTruck - すべてのデバイス]** をクリックします。

1. **[ジョブの種類]** ドロップダウン リストで、 **[プロパティ]** をクリックします。

1. **[名前]** ドロップダウン リストで、"**適温**" をクリックします。

1. **[値]** テキストボックスに、 **-10** と入力します

    このジョブを実行すると、デバイス グループ内のすべてのトラックに適温が設定されます。この場合は 1 つだけです。

1. ウィンドウの下部で、 **[次へ]** をクリックし、もう一度 **[次へ]** をクリックします。

1. **[配信オプション]** と **[スケジュール]** の既定の設定をそのまま使うには、 **[次へ]** をクリックし、もう一度 **[次へ]** をクリックします。

1. **[レビュー]** ページで、 **[実行]** を選びます。

1. しばらくすると、ジョブの **[状態]** が **[保留中]** から **[完了]** に変わります。

    わずか数秒で変化します。

1. **[デバイス]** を使用して、ダッシュボードに戻ります。

1. ダッシュボードの **[最適温度]** タイルで、**最適温度** が -10 に設定されていることを確認します。

1 台のトラックのテストが完了したら、IoT Central システムの拡張を検討します。

### <a name="exercise-7-create-multiple-devices"></a>演習 7: 複数のデバイスを作成する

この演習では、所有車両に複数のトラックを追加するために必要な手順を完了します。

#### <a name="task-1-add-multiple-devices-to-the-iot-central-app"></a>タスク 1:IoT Central アプリに複数のデバイスを追加する

1. IoT Central アプリが開いていることを確認します。

    必要に応じて、[Azure IoT Central](https://apps.azureiotcentral.com/?azure-portal=true) アプリを開きます。

1. 左側のナビゲーション メニューで、 **[デバイス]** をクリックします。

1. **[デバイス]** ([デバイス名] ではありません) で、**RefrigeratedTruck** をクリックします。

    これにより、作成したデバイスがこのデバイス テンプレートを使用できるようになります。 選択したデバイス テンプレートが、太字で表示されます。

1. **RefrigeratedTruck** のページで、 **[+ 新規]** をクリックします。

    デフォルトのデバイス名に **RefrigeratedTruck** というテキストが含まれていることを確認します。 そうでない場合は、適切なデバイス テンプレートを選択していません。

1. **[新しいデバイスの作成]** ダイアログ ボックスの **[デバイス ID]** に、「**RefrigeratedTruck2**」と入力します

1. **[デバイス名]** に「**RefrigeratedTruck - 2**」と入力します

1. **[新しいデバイスの作成]** ダイアログ ボックスの下部にある **[作成]** をクリックします。

    必要に応じて、上記のプロセスを追加のトラックに繰り返すことができます。

#### <a name="task-2-provision-the-new-devices"></a>タスク 2:新しいデバイスをプロビジョニングする

1. **[デバイス名]** で **RefrigeratedTruck - 2** をクリックします。

1. ページの左上にある **[接続]** をクリックします。

1. **[デバイス接続]** ダイアログ ボックスで、設定 **[デバイス ID]** と **[主キー]** の値をテキスト ファイルにコピーし、これが 2 台目のトラック用であることを確認します。

    この値は最初のトラックの値と同じであるため、 **[ID スコープ]** をコピーする必要はありません (個々のデバイスではなく、アプリを識別します)。

1. **[デバイス接続]** ダイアログ ボックスの下部にある **[閉じる]** をクリックします。

1. **[デバイス]** ページに戻り、作成した他のデバイスについてプロセスを繰り返して、 **[デバイス ID]** と **[主キー]** の値をテキスト ファイルにコピーします。

1. すべての新しいトラックの接続情報の記録が済んだら、 **[プロビジョニング状態]** が引き続き **[登録済み]** になっていることに注目してください。

    接続を確立するまで、これは変更されません。

#### <a name="task-3-create-new-apps-for-each-new-device"></a>タスク 3:新しいデバイスごとに新しいアプリを作成する

各トラックは、シミュレートされたデバイス アプリの個別に実行されているインスタンスによってシミュレートされます。 そのため、複数のバージョンのアプリを同時に実行する必要があります。

1. 新しいシミュレートされたデバイス アプリを作成するには、IoT Central アプリで作成した新しいトラックごとに、Visual Studio Code の個別のインスタンスを開きます。

1. トラック 1 に使われたデバイス コードをコピーし、他のトラック インスタンスごとにコード エディターに貼り付けます。

1. 新しいトラックごとに **[デバイス ID]** と **[主キー]** の設定に値が設定されていることを確認します。

    **スコープ ID** と **Azure Maps アカウントの主キー** は、すべてのデバイスで同じである必要があります。

1. 新しいプロジェクトごとに必要なライブラリを読み込む必要があります。

1. 各プロジェクトの `truckNum` を別の値に変更します。

1. プロジェクトごとに、ターミナル コマンド `dotnet run` を使用してアプリを起動します。

#### <a name="task-4-verify-the-telemetry-from-all-the-devices"></a>タスク 4:すべてのデバイスからのテレメトリを確認する

1. 作成したダッシュボードがすべてのトラックに対して動作することを確認します。

1. 各トラックのダッシュボードを使用して、さまざまな顧客への指示をトラックに試してみます。

1. 各ダッシュボードの **場所** マップを使用して、トラックが正しい方向に向かっているのを確認します。

    ラボを完了しました。おめでとうございます!

1. Azure Maps アカウントに使ったリソース グループを削除します。

    料金を最小にするには、使い終わったら Azure リソースを削除することが常に重要です。
