---
lab:
  title: 'ラボ: Azure VM での Windows Server のデプロイと構成'
  type: Answer Key
  module: 'Module 6: Deploying and Configuring Azure VMs'
ms.openlocfilehash: 19550acaddcfe670775ddd8bcc56d7df05ef72be
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907031"
---
# <a name="lab-answer-key-deploying-and-configuring-windows-server-on-azure-vms"></a>ラボ回答キー: Azure VM での Windows Server のデプロイと構成

## <a name="exercise-1-authoring-azure-resource-manager-arm-templates-for-azure-vm-deployment"></a>演習 1: Azure VM デプロイ用の Azure Resource Manager (ARM) テンプレートの作成

#### <a name="task-1-connect-to-your-azure-subscription-and-enable-enhanced-security-of-microsoft-defender-for-cloud"></a>タスク 1: Azure サブスクリプションに接続し、Microsoft Defender for Cloud のセキュリティ強化を有効にする

このタスクでは、Azure サブスクリプションに接続し、Microsoft Defender for Cloud のセキュリティ強化機能を有効にします。

1. **SEA-ADM1** に接続し、必要に応じて、パスワード **Pa55w.rd** を使用し、**CONTOSO\\Administrator** としてサインインします。
1. **SEA-ADM1** で Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。

>**注**: Azure サブスクリプションで Microsoft Defender for Cloud を既に有効にしている場合は、このタスクの残りのステップをスキップし、次に直接進みます。

1. Azure portal のツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**Microsoft Defender for Cloud**」を検索して選択します。
1. 「**Microsoft Defender for Cloud \| 使用開始**」ページで、 **[アップグレード]** を選択し、 **[エージェントのインストール]** を選択します。

#### <a name="task-2-generate-an-arm-template-and-parameters-files-by-using-the-azure-portal"></a>タスク 2: Azure portal を使用して ARM テンプレートとパラメーター ファイルを生成する

このタスクでは、Azure portal を使用してリソース グループを作成し、そのリソース グループにディスクを作成します。

1. **[SEA-ADM1]** 上の Azure portal のツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**仮想マシン**」を検索して選択します。 「**仮想マシン**」ページで、 **[+ 作成]** を選択し、 **[仮想マシン]** を選択します。
1. 「**仮想マシンの作成**」ページの **[基本]** タブで、次の設定を指定し、他のすべての設定は既定値のままにしますが、デプロイは行いません。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前。|
   |リソース グループ|新しいリソース グループの名前 **AZ800-L0601-RG**|
   |仮想マシン名|**az800l06-vm0**|
   |リージョン|Azure 仮想マシンをプロビジョニングできる Azure リージョンの名前を使用します|
   |可用性のオプション|インフラストラクチャの冗長性は必要ありません|
   |Image|**Windows Server 2022 Datacenter: Azure Edition - Gen2**|
   |Azure Spot インスタンス|いいえ|
   |サイズ|**Standard_D2s_v3**|
   |ユーザー名|**学生**|
   |パスワード|**Pa55w.rd1234**|
   |パブリック受信ポート|なし|
   |既存の Windows Server ライセンスを使用しますか|いいえ|

1. **[次へ: ディスク >]** を選択し、「**仮想マシンの作成**」ページの **[ディスク]** タブで、次の設定を指定します。他のすべての設定は既定値のままにします。

   |設定|値|
   |---|---|
   |OS ディスクの種類|**Standard HDD**|

1. **[次へ: ネットワーク >]** を選択し、「**仮想マシンの作成**」ページの **[ネットワーク]** タブで、 **[仮想ネットワーク]** テキスト ボックスの後にある **[新規作成]** ハイパーリンクを選択します*。
1. 「**仮想ネットワークの作成**」ページで、次の設定を指定し、他のすべての設定を既定値のままにして、 **[OK]** を選択します。

   |設定|値|
   |---|---|
   |名前|**az800l06-vnet**|
   |アドレス範囲|**10.60.0.0/20**|
   |サブネット名|**subnet0**|
   |サブネット範囲|**10.60.0.0/24**|

1. 「**仮想マシンの作成**」ページに戻り、 **[ネットワーク]** タブで、次の設定を指定します。他のすべての設定は既定値のままにします。

   |設定|値|
   |---|---|
   |パブリック IP|なし|
   |NIC ネットワーク セキュリティ グループ|None|
   |Accelerated Networking|Off|
   |この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか?|いいえ|

1. **[次へ: 管理 >]** を選択し、「**仮想マシンの作成**」ページの **[管理]** タブで、次の設定を指定します。他のすべての設定は既定値のままにします。

   |設定|値|
   |---|---|
   |ブート診断|**マネージド ストレージ アカウントで有効にする (推奨)**|
   
1. **[次へ: 詳細設定 >]** を選択し、「**仮想マシンの作成**」ページの **[詳細設定]** タブで、使用可能な設定を変更せずに確認し、 **[確認と作成]** を選択します。

   >**注**: 仮想マシンは作成しないでください。 この目的で、自動生成されたテンプレートを使用します。

#### <a name="task-3-download-the-arm-template-and-parameters-files-from-the-azure-portal"></a>タスク 3: Azure portal から ARM テンプレートとパラメーター ファイルをダウンロードする

1. Azure portal の「**仮想マシンの作成**」ページで、 **[オートメーション用のテンプレートをダウンロードする]** を選択します。
1. 「**テンプレート**」ページで **[ダウンロード]** を選択します。
1. **template.zip** の横にある省略記号ボタンを選択し、ポップアップ メニューで **[フォルダーに表示]** を選択します。 これにより、 **[ダウンロード]** フォルダーの内容が表示されたエクスプローラーが開きます。
1. エクスプローラーで、**template.zip** を **SEA-ADM1** の **C:\\Labfiles\\Lab06** フォルダーにコピーします (必要に応じて新しいフォルダーを作成します)。
1. 「**テンプレート**」ページから「**仮想マシンの作成**」ページに戻り、デプロイを完了せずに閉じます。

## <a name="exercise-2-modifying-arm-templates-to-include-vm-extension-based-configuration"></a>演習 2: VM 拡張機能ベースの構成を含むように ARM テンプレートを変更する

#### <a name="task-1-review-the-arm-template-and-parameters-files-for-azure-vm-deployment"></a>タスク 1: Azure VM デプロイ用の ARM テンプレートとパラメーターのファイルを確認する

1. **SEA-ADM1** で、エクスプローラーを開始し、**C:\\Labfiles\\Lab06** フォルダーを参照します。
1. **template.zip** ファイルの内容を同じフォルダーに抽出します。
1. メモ帳で **template.json** ファイルを開き、内容を確認します。 メモ帳ウィンドウは開いたままにしておきます。
1. エクスプローラーから、メモ帳で **C:\\Labfiles\\Lab06\\parameters.json** ファイルを開き、その内容を確認します。
1. **parameters.json** ファイルが表示されているメモ帳ウィンドウを閉じます。

#### <a name="task-2-add-an-azure-vm-extension-section-to-the-existing-template"></a>タスク 2: 既存のテンプレートに Azure VM 拡張機能セクションを追加する

1. **SEA-ADM1** の **template.json** ファイルの内容が表示されているメモ帳ウィンドウで、`    "resources": [` 行の直後に次のコードを挿入します。

   ```json
        {
           "type": "Microsoft.Compute/virtualMachines/extensions",
           "name": "[concat(parameters('virtualMachineName'), '/customScriptExtension')]",
           "apiVersion": "2018-06-01",
           "location": "[resourceGroup().location]",
           "dependsOn": [
               "[concat('Microsoft.Compute/virtualMachines/', parameters('virtualMachineName'))]"
           ],
           "properties": {
               "publisher": "Microsoft.Compute",
               "type": "CustomScriptExtension",
               "typeHandlerVersion": "1.7",
               "autoUpgradeMinorVersion": true,
               "settings": {
                   "commandToExecute": "powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools && powershell.exe remove-item 'C:\\inetpub\\wwwroot\\iisstart.htm' && powershell.exe Add-Content -Path 'C:\\inetpub\\wwwroot\\iisstart.htm' -Value $('Hello World from ' + $env:computername)"
              }
           }
        },
   ```

1. 変更を保存して、ファイルを閉じます。

## <a name="exercise-3-deploying-azure-vms-running-windows-server-by-using-arm-templates"></a>演習 3: ARM テンプレートを使用して Windows Server を実行している Azure VM をデプロイする

#### <a name="task-1-deploy-an-azure-vm-by-using-an-arm-template"></a>タスク 1: ARM テンプレートを使用して Azure VM をデプロイする

1. **SEA-ADM1** で、Azure portal が表示されているブラウザー ウィンドウに切り替えます。
1. Azure portal のツール バーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**Template deployment (カスタム テンプレートを使用してデプロイ)** 」を検索して選択します。
1. 「**カスタム デプロイ**」ページで、 **[エディターで独自のテンプレートを作成する]** を選択します。
1. 「**テンプレートの編集**」ページで、 **[ファイルの読み込み]** を選択し、前の演習で編集したテンプレート ファイル **template.json** をアップロードして、 **[保存]** を選択します。
1. 「**カスタム デプロイ**」ページで、 **[パラメーターの編集]** を選択します。
1. 「**パラメーターの編集**」ページで、 **[ファイルの読み込み]** を選択し、前の演習で確認したパラメーター ファイル **parameters.json** をアップロードして、 **[保存]** を選択します。
1. 「**カスタム デプロイ**」ページに戻り、次の設定を指定して、その他の設定は既定値のままにします。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**AZ800-L0601-RG**|
   |リージョン|Azure VM をプロビジョニングできる Azure リージョンの名前|
   |管理パスワード|**Pa55w.rd1234**|

1. **[確認と作成]** を選択し、次に **[作成]** を選択します。

   >**注**: デプロイには約 10 分かかります。

1. デプロイが正常に完了したことを確認します。

#### <a name="task-2-review-results-of-the-azure-vm-deployment"></a>タスク 2: Azure VM のデプロイの結果を確認する

1. Azure portal のツール バーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**リソース グループ**」を検索して選択します。
1. 「**リソース グループ**」ページで、 **[AZ800-L0601-RG]** エントリを選択します。
1. 「**AZ800-L0601-RG**」ページの「**概要**」ページで、Azure VM **az800l06-vm0** が含まれるリソースのリストを確認します。
1. リソースの一覧で、Azure VM **[az800l06-vm0]** エントリを選択します。 
1. 「**az800l06-vm0**」ページで、 **[拡張機能]** を選択し、拡張機能の一覧で、**customScriptExtension** が正常にプロビジョニングされていることを確認します。
1. 「**AZ800-L0601-RG**」ページに戻り、 **[設定]** セクションで **[デプロイ]** を選択します。
1. 「**AZ800-L0601-RG \| デプロイ**」ページで、 **[Microsoft.Template]** リンクを選択します。
1. 「**Microsoft.Template \| 概要**」ページで、 **[テンプレート]** を選択します。これは、デプロイに使用したテンプレートと同じであることに注意してください。

## <a name="exercise-4-configuring-administrative-access-to-azure-vms-running-windows-server"></a>演習 4: Windows Server を実行している Azure VM への管理アクセスの構成

#### <a name="task-1-verify-the-status-of-azure-microsoft-defender-for-cloud"></a>タスク 1: Azure Microsoft Defender for Cloud の状態を確認する

1. Azure portal のツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**Microsoft Defender for Cloud**」を検索して選択します。
1. Microsoft Defender for Cloud の「**概要**」ページで、左側の垂直メニューの **[管理]** セクションの、 **[環境設定]** を選択します。 
1. 「**環境設定**」ページで、Azure サブスクリプションを表すエントリを選択します。
1. 「**設定 \| Defender プラン**」ページで、タイル **[すべての Microsoft Defender for Cloud プランの有効化]** が選択されていることを確認し、左側の垂直メニューの **[設定]** セクションで、 **[自動プロビジョニング]** を選択します。
1. 「**設定 \| 自動プロビジョニング**」ページの拡張機能の一覧で、 **[Azure VM の Log Analytics エージェント]** エントリの右側にある **[構成の編集]** リンクを選択します。
1. **[拡張機能のデプロイ構成]** で、 **[Azure VM を Defender for Cloud で作成された既定のワークスペースに接続する]** のエントリが選択されていることを確認し、 **[適用]** を選択し、「**設定 \| 自動プロビジョング**」ページで **[保存]** を選択します。

#### <a name="task-2-review-just-in-time-vm-access-settings"></a>タスク 2: Just-In-Time VM アクセスの設定を確認する

1. Microsoft Defender for Cloud の「**概要**」ページに戻り、 **[クラウド セキュリティ]** セクションで、 **[ワークロードの保護]** を選択します。
1. \* 「**Microsoft Defender for Cloud \| ワークロードの保護**」ページで、 **[Just In Time VM アクセス]** を選択します。
1. 「**Just In Time VM アクセス**」ページで、 **[構成済み]** 、 **[未構成]** 、および **[サポート対象外]** のタブを確認します。

   >**注**: 新しくデプロイされた VM が **[サポート対象外]** タブに表示されるまで、最大で 24 時間かかる場合があります。待機するよりも、次の演習に進んでください。

## <a name="exercise-5-configuring-windows-server-security-in-azure-vms"></a>演習 5: Azure VM での Windows Server のセキュリティの構成

#### <a name="task-1-create-and-configure-an-nsg"></a>タスク 1: NSG を作成して構成する

1. Azure portal のツール バーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**ネットワーク セキュリティ グループ**」を検索して選択します。
1. [**ネットワーク セキュリティ グループ**] ページで、 **[+ 作成]** を選択します。
1. 「**ネットワーク セキュリティ グループの作成**」ページの **[基本]** タブで、以下の設定を指定します (それ以外は既定値のままにしておきます)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**AZ800-L0601-RG**|
   |名前|**az800l06-vm0-nsg1**|
   |リージョン|Azure VM **az800l06-vm0** をプロビジョニングした Azure リージョンの名前|

1. 「**ネットワーク セキュリティ グループの作成**」ページの **[基本]** タブで、 **[確認と作成]** を選択し、 **[作成]** を選択します。
1. Azure portal で、「**AZ800-L0601-RG**」ページに戻り、リソースの一覧で、新しく作成されたネットワークセキュリティ グループ **az800l06-vm0-nsg1** を表すエントリを選択します。
1. 「**az800l06-vm0-nsg1**」ページで、既定の受信および送信のセキュリティ規則の一覧を確認し、 **[設定]** セクションで **[受信セキュリティ規則]** を選択します。
1. 「**az800l06-vm0-nsg1 \| 受信セキュリティ規則**」ページで、 **[+ 追加]** を選択します。
1. 「**受信セキュリティ規則の追加**」ページで、次の設定を指定し、それ以外はすべて既定値のままにして、 **[追加]** を選択します。

   |設定|値|
   |---|---|
   |source|**[任意]**|
   |Source port ranges|*|
   |到着地|**[任意]**|
   |サービス|**HTTP**|
   |アクション|**許可**|
   |Priority|**300**|
   |名前|**AllowHTTPInBound**|

#### <a name="task-2-configure-inbound-http-access-to-an-azure-vm"></a>タスク 2: Azure VM への受信 HTTP アクセスを構成する

1. Azure portal で、「**AZ800-L0601-RG**」ページに戻り、リソースの一覧で、Azure VM **az800l06-vm0** を表すエントリを選択します。
1. 「**az800l06-vm0**」ページで、 **[ネットワーク]** を選択します。
1. 「**az800l06-vm0\| ネットワーク**」ページで、**az800l06-vm0** にアタッチされているネットワーク インターフェイスを示すリンクを選択します。
1. ネットワーク インターフェイスのプロパティを表示しているページの左側にある垂直メニューの **[設定]** セクションで、 **[ネットワーク セキュリティ グループ]** を選択します。 
1. 「**ネットワーク セキュリティ グループ**」ページで、ドロップダウン リストから **[az800l06-vm0-nsg1]** を選択し、 **[保存]** を選択します。
1. ネットワーク インターフェイスのプロパティが表示されたページに戻り、 **[IP 構成]** を選択して、 **[ipconfig1]** エントリを選択します。
1. 「**ipconfig1**」ページの **[パブリック IP アドレス]** セクションで、 **[関連付け]** を選択し、 **[パブリック IP アドレス]** ドロップダウン リストの下で **[新規作成]** を選択します。
1. **[パブリック IP アドレスの追加]** ウィンドウで、次の設定を指定し、 **[OK]** を選択します。

   |設定|値|
   |---|---|
   |名前|**az800l06-vm0-pip1**|
   |SKU|**Standard**|

1. 「**ipconfig1**」ページに戻り、 **[保存]** を選択します。
1. ネットワーク インターフェイスのプロパティが表示されたページに戻り、 **[概要]** を選択します。 インターフェイスに割り当てられているパブリック IP アドレスの値を確認します。
1. 別のブラウザー タブを開き、その IP アドレスに移動して、 **[Hello World from az800l06-vm0]** が表示された Web ページが開くことを確認します。
1. ラボ コンピューターからリモート デスクトップ アプリを起動し、同じ IP アドレスに接続します。 接続に失敗することを確認します。

   >**注**: Azure VM は現在、TCP ポート 3389 経由でインターネットからアクセスできないので、これは想定されています。 TCP ポート 80 経由でのみアクセスできます。

#### <a name="task-3-trigger-re-evaluation-of-the-jit-status-of-an-azure-vm"></a>タスク 3: Azure VM の JIT 状態の再評価をトリガーする

>**注**: このタスクは、Azure VM の JIT 状態の再評価をトリガーするために必要です。 既定では、これには最大 24 時間かかる場合があります。

1. Azure portal で、「**AZ800-L0601-RG**」ページに戻り、リソースの一覧で、Azure VM **az800l06-vm0** を表すエントリを選択します。
1. 「**az800l06-vm0**」ページで、 **[構成]** を選択します。 
1. 「**az800l06-vm0 \| 構成**」ページで、 **[JIT VM アクセスを有効にする]** を選択し、 **[Azure Security Center を開く]** リンクを選択します。
1. 「**Just In Time VM アクセス**」ページで、**az800l06-vm0** Azure VM を表すエントリが **[構成済み]** タブに表示されていることを確認します。

#### <a name="task-4-connect-to-the-azure-vm-via-jit-vm-access"></a>タスク 4: JIT VM アクセス経由で Azure VM に接続する

1. 「**az800l06-vm0**」ページに戻り、 **[接続]** を選択し、ドロップダウン リストで **[RDP]** を選択します。 
1. 「**az800l06-vm0 \| 接続**」ページの **[接続元 IP アドレス]** セクションで、 **[自分の IP]** を選択し、 **[アクセス権の要求]** を選択します。
1. 要求が承認されたことを示す通知を待ち、 **[RDP ファイルのダウンロード]** を選択し、プロンプトに従ってターゲット Azure VM に接続します。
1. 資格情報のプロンプトが表示されたら、次の値を指定して、 **[OK]** を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**学生**|
   |パスワード|**Pa55w.rd1234**|

1. Azure VM で実行されているオペレーティング システムがリモート デスクトップ経由で正常にアクセスできることを確認し、リモート デスクトップ セッションを閉じます。

## <a name="exercise-6-deprovisioning-the-azure-environment"></a>演習 6: Azure 環境のプロビジョニング解除

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>タスク 1: Cloud Shell で PowerShell セッションを開始する

1. **SEA-ADM1** の Azure portal を表示している Microsoft Edge ウィンドウで、Cloud Shell アイコンを選んで Azure Cloud Shell ウィンドウを開きます。
1. **Bash** または **PowerShell** の選択を求めるプロンプトが表示されたら、 **[PowerShell]** を選択します。

   >**注**: Cloud Shell を起動するのが初めてで、**ストレージがマウントされていません** というメッセージが表示される場合は、このラボで使用しているサブスクリプションを選択してから、 **[ストレージの作成]** を選択します。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>タスク 2: ラボでプロビジョニングしたすべての Azure リソースを特定する

1. [Cloud Shell] ウィンドウで次のコマンドを実行して、このラボで作成されたすべてのリソース グループのリストを表示します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*'
   ```
1. 次のコマンドを実行して、このラボで作成したすべてのリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*' | Remove-AzResourceGroup -Force -AsJob
   ```
   >**注**: このコマンドは非同期で実行されます ( *-AsJob* パラメーターによって決定されます)。 そのため、同じ PowerShell セッション内ですぐに別の PowerShell コマンドを実行できるようになりますが、リソース グループが実際に削除されるまでに数分かかります。
