---
lab:
  title: 'ラボ: Azure VM での Windows Server のデプロイと構成'
  module: 'Module 6: Deploying and Configuring Azure VMs'
ms.openlocfilehash: d2505e999d7e5194fd7a407cc035834dee2464f3
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907002"
---
# <a name="lab-deploying-and-configuring-windows-server-on-azure-vms"></a>ラボ: Azure VM での Windows Server のデプロイと構成

## <a name="scenario"></a>シナリオ

現在のインフラストラクチャに関する懸念事項に対処する必要があります。 Windows Server ベースのワークロードを実行している Azure VM に適用する必要がある追加の制御に関して、運用モデルが古い、自動化の使用が限定的、および情報セキュリティ チームの懸念があります。 あなたは、Windows Server で実行されている Azure VM の自動デプロイと構成プロセスを開発して実装することにしました。

このプロセスには、Azure VM 拡張機能を使用した Azure Resource Manager (ARM) テンプレートと OS 構成が伴います。 また、AppLocker によるアプリケーション許可リスト、ファイルの整合性チェック、アダプティブ ネットワークまたは DDoS 保護など、オンプレミス システムに既に適用されている以外の追加のセキュリティ保護メカニズムも組み込まれます。 また、JIT 機能を利用して、Azure VM への管理アクセスをロンドン本社に関連付けられているパブリック IP アドレス範囲に制限します。

目標は、管理容易性とセキュリティの要件を満たす方法で、Windows Server を実行している Azure VM をデプロイして構成することです。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure VM デプロイ用の ARM テンプレートを作成する。
- VM 拡張機能ベースの構成を含むように ARM テンプレートを変更する。
- ARM テンプレートを使用して Windows Server を実行している Azure VM をデプロイする。
- Windows Server を実行している Azure VM への管理アクセスを構成する。
- Azure VM で Windows Server のセキュリティを構成する。
- Azure 環境をプロビジョニング解除する。

## <a name="estimated-time-90-minutes"></a>予想所要時間: 90 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1** および **AZ-800T00A-ADM1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-800T00A-SEA-DC1** と **AZ-800T00A-SEA-ADM1** の各仮想マシンが **SEA-DC1** と **SEA-ADM1** のインストールをホストしています。

1. **SEA-ADM1** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、使用可能な VM 環境と Azure サブスクリプションを使用します。 ラボを開始する前に、Azure サブスクリプションと、そのサブスクリプションの所有者または共同作成者ロールを持つユーザー アカウントがあることを確認してください。

## <a name="exercise-1-authoring-arm-templates-for-azure-vm-deployment"></a>演習 1: Azure VM デプロイ用の ARM テンプレートの作成

### <a name="scenario"></a>シナリオ

Azure ベースの操作を効率化するために、Azure VM への Windows Server の自動デプロイと構成プロセスを開発して実装することにしました。 デプロイは、情報セキュリティ チームの要件に準拠し、高可用性を含む Contoso, Ltd. の意図されたターゲット運用モデルに準拠する必要があります。

この演習の主なタスクは次のとおりです。

1. Azure サブスクリプションに接続し、Microsoft Defender for Cloud のセキュリティ強化を有効にする。
1. Azure portal を使用して ARM テンプレートとパラメーター ファイルを生成する。
1. Azure portal を使用して ARM テンプレートとパラメーター ファイルをダウンロードする。

#### <a name="task-1-connect-to-your-azure-subscription-and-enable-enhanced-security-of-microsoft-defender-for-cloud"></a>タスク 1: Azure サブスクリプションに接続し、Microsoft Defender for Cloud のセキュリティ強化を有効にする

このタスクでは、Azure サブスクリプションに接続し、Microsoft Defender for Cloud のセキュリティ強化機能を有効にします。

1. **SEA-ADM1** に接続し、必要に応じて、パスワード **Pa55w.rd** を使用し、**CONTOSO\\Administrator** としてサインインします。
1. **SEA-ADM1** で Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。

>**注**: Azure サブスクリプションで Microsoft Defender for Cloud を既に有効にしている場合は、このタスクの残りのステップをスキップし、次に直接進みます。

1. Azure portal で、「**Microsoft Defender for Cloud**」ページを参照します。
1. 「**Microsoft Defender for Cloud \| 使用開始**」ページで、Microsoft Defender for Cloud のセキュリティを強化し、Microsoft Defender for Cloud エージェントの自動インストールを有効にしてください。

#### <a name="task-2-generate-an-arm-template-and-parameters-files-by-using-the-azure-portal"></a>タスク 2: Azure portal を使用して ARM テンプレートとパラメーター ファイルを生成する

1. Azure portal から、次の設定を使用して新しい Azure VM を作成するプロセスをステップ実行します (それ以外はすべて既定値のままにします)。ただし、デプロイは行いません。

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
   |OS ディスクの種類|**Standard HDD**|
   |名前|**az800l06-vnet**|
   |アドレス範囲|**10.60.0.0/20**|
   |サブネット名|**subnet0**|
   |サブネット範囲|**10.60.0.0/24**|
   |パブリック IP|なし|
   |NIC ネットワーク セキュリティ グループ|None|
   |Accelerated Networking|Off|
   |この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか?|いいえ|
   |ブート診断|**マネージド ストレージ アカウントで有効にする (推奨)**|

1. 「**仮想マシンの作成**」ページの **[確認と作成]** タブまで移動したら、タスク 3 に進みます。

   >**注**: 仮想マシンは作成しないでください。 この目的で、自動生成されたテンプレートを使用します。

#### <a name="task-3-download-the-arm-template-and-parameters-files-from-the-azure-portal"></a>タスク 3: Azure portal から ARM テンプレートとパラメーター ファイルをダウンロードする

1. 「**仮想マシンの作成**」ページの **[確認と作成]** タブから、自動化のためのテンプレートをダウンロードして、ラボ VM の **C:\\Labfiles\\Mod06** フォルダーにコピーします。
1. Azure portal で、「**仮想マシンの作成**」ページを閉じます。

## <a name="exercise-2-modifying-arm-templates-to-include-vm-extension-based-configuration"></a>演習 2: VM 拡張機能ベースの構成を含むように ARM テンプレートを変更する

### <a name="scenario"></a>シナリオ

Azure リソースのデプロイの自動化に加えて、Azure VM で実行されている Windows サーバーを自動的に構成することもできます。 これを実現するには、Azure カスタム スクリプト拡張機能の使用をテストします。

この演習の主なタスクは次のとおりです。

1. Azure VM デプロイ用の ARM テンプレートとパラメーターのファイルを確認する。
1. 既存のテンプレートに Azure VM 拡張機能セクションを追加する。

#### <a name="task-1-review-the-arm-template-and-parameters-files-for-azure-vm-deployment"></a>タスク 1: Azure VM デプロイ用の ARM テンプレートとパラメーターのファイルを確認する

1. ダウンロードしたアーカイブの内容を **C:\\Labfiles\\Mod06** フォルダーに抽出します。
1. メモ帳で **template.json** ファイルを開き、内容を確認します。 メモ帳ウィンドウは開いたままにしておきます。
1. メモ帳で **C:\\Labfiles\\Mod06\\parameters.json** ファイルを開き、確認して、メモ帳ウィンドウを閉じます。

#### <a name="task-2-add-an-azure-vm-extension-section-to-the-existing-template"></a>タスク 2: 既存のテンプレートに Azure VM 拡張機能セクションを追加する

1. ラボ VM の **template.json** ファイルの内容が表示されているメモ帳ウィンドウで、`    "resources": [` 行の直後に次のコードを挿入します。

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

### <a name="scenario"></a>シナリオ

ARM テンプレートが構成された状態で、概念実証 Azure サブスクリプションへのデプロイを実行して、その機能を確認します。

この演習の主なタスクは次のとおりです。

1. ARM テンプレートを使用して Azure VM をデプロイする。
1. Azure VM のデプロイの結果を確認する。

#### <a name="task-1-deploy-an-azure-vm-by-using-an-arm-template"></a>タスク 1: ARM テンプレートを使用して Azure VM をデプロイする

1. **SEA-ADM1** の Azure portal で、「**カスタム デプロイ**」ページを参照し、 **[エディターで独自のテンプレートを作成する]** オプションを選択します。
1. テンプレート ファイルとパラメーター ファイルを「**カスタム デプロイ**」ページに読み込みます。
1. 次の設定でテンプレートをデプロイし、他のすべての設定は既定値のままにします。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**AZ800-L0601-RG**|
   |リージョン|Azure VM をプロビジョニングできる Azure リージョンの名前|
   |管理パスワード|**Pa55w.rd1234**|

   >**注**: デプロイには約 10 分かかります。

1. デプロイが正常に完了したことを確認します。

#### <a name="task-2-review-results-of-the-azure-vm-deployment"></a>タスク 2: Azure VM のデプロイの結果を確認する

1. Azure portal で **AZ800-L0601-RG** リソース グループ ページを参照し、そのリソースの一覧 (特に Azure VM **az800l06-vm0**) を確認します。
1. **az800l06-vm0** Azure VM ページを参照し、**customScriptExtension** が正常にプロビジョニングされたことを確認します。
1. **AZ800-L0601-RG** リソース グループ ページに戻り、そのデプロイを確認し、デプロイに使用された **Microsoft.Template** がデプロイに使用したテンプレートと一致することを確認します。

## <a name="exercise-4-configuring-administrative-access-to-azure-vms-running-windows-server"></a>演習 4: Windows Server を実行している Azure VM への管理アクセスの構成

### <a name="scenario"></a>シナリオ

Azure VM が Windows Server を実行している状態で、オンプレミスの管理ワークステーションからリモートで管理する機能をテストする必要があります。

この演習の主なタスクは次のとおりです。

1. Azure Microsoft Defender for Cloud の状態を確認する。
1. Just-In-Time VM アクセスの設定を確認する。

#### <a name="task-1-verify-the-status-of-azure-microsoft-defender-for-cloud"></a>タスク 1: Azure Microsoft Defender for Cloud の状態を確認する

1. Azure portal で、「**Microsoft Defender for Cloud**」ページを参照します。
1. Microsoft Defender for Cloud のセキュリティ強化機能が有効になっていることを確認します。

#### <a name="task-2-review-the-just-in-time-access-settings"></a>タスク 2: Just-In-Time アクセス設定を確認する

1. Azure portal で、「**Microsoft Defender for Cloud\| ワークロード保護**」ページを参照し、 **[Just In Time VM アクセス]** 設定を確認します。
1. 「**Just In Time VM アクセス**」ページで、 **[構成済み]** 、 **[未構成]** 、および **[サポート対象外]** のタブを確認します。

   >**注**: 新しくデプロイされた VM が **[サポート対象外]** タブに表示されるまで、最大で 24 時間かかる場合があります。待機するよりも、次の演習に進んでください。

## <a name="exercise-5-configuring-windows-server-security-in-azure-vms"></a>演習 5: Azure VM での Windows Server のセキュリティの構成

### <a name="scenario"></a>シナリオ

Azure VM が Windows Server を実行している状態で、オンプレミスの管理ワークステーションからリモートで管理する機能をテストする必要があります。

この演習の主なタスクは次のとおりです。

1. NSG を作成して構成する。
1. Azure VM への受信 HTTP アクセスを構成する。
1. Azure VM の JIT 状態の再評価をトリガーする。
1. JIT VM アクセス経由で Azure VM に接続する。

#### <a name="task-1-create-and-configure-an-nsg"></a>タスク 1: NSG を作成して構成する

1. Azure portal で、次の設定で NSG を作成し、他のすべての設定は既定値のままにします。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**AZ800-L0601-RG**|
   |名前|**az800l06-vm0-nsg1**|
   |リージョン|Azure VM **az800l06-vm0** をプロビジョニングした Azure リージョンの名前|

1. 次の設定で、新しく作成されたネットワーク セキュリティ グループに受信セキュリティ規則を追加します。他のすべての設定は既定値のままにします。

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

1. Azure portal で **az800l06-vm0** Azure VM にアタッチされているネットワーク インターフェイスのページを参照し、前のタスクで作成したネットワーク セキュリティ グループに関連付けます。
1. Azure portal で **az800l06-vm0** Azure VM にアタッチされているネットワーク インターフェイスの IP 構成を参照し、それを次の設定で新しいパブリック IP アドレスと関連付け、それ以外はすべて既定値のままにします。

   |設定|値|
   |---|---|
   |名前|**az800l06-vm0-pip1**|
   |SKU|**Standard**|

1. ラボ VM からブラウザー タブを開き、新しく作成されたパブリック IP アドレスを参照し、**Hello World from az800l06-vm0** のメッセージがページに表示されていることを確認します。
1. ラボ VM から同じ IP アドレスへのリモート デスクトップ接続を確立し、接続試行が失敗することを確認します。

   >**注**: Azure VM は現在、TCP ポート 3389 経由でインターネットからアクセスできないので、これは想定されています。 TCP ポート 80 経由でのみアクセスできます。

#### <a name="task-3-trigger-re-evaluation-of-the-jit-status-of-an-azure-vm"></a>タスク 3: Azure VM の JIT 状態の再評価をトリガーする

>**注**: このタスクは、Azure VM の JIT 状態の再評価をトリガーするために必要です。 既定では、これには最大 24 時間かかる場合があります。

1. Azure portal で、「**az800l06-vm0**」ページに戻ります。
1. 「**az800l06-vm0**」ページで、 **[構成]** を選択します。 
1. 「**az800l06-vm0 \| 構成**」ページで、 **[JIT VM アクセスを有効にする]** を選択し、 **[Azure Security Center を開く]** リンクを選択します。
1. 「**Just-In-Time VM アクセス**」ページで、**az800l06-vm0** Azure VM を表すエントリが **[構成済み]** タブに表示されていることを確認します。

#### <a name="task-4-connect-to-the-azure-vm-via-jit-vm-access"></a>タスク 4: JIT VM アクセス経由で Azure VM に接続する

1. Azure portal の「**az800l06-vm0**」ページから、JIT VM アクセスを要求します。
1. 要求が承認されると、ターゲット Azure VM へのリモート デスクトップ セッションが開始します。
1. 資格情報のプロンプトが表示されたら、次の値を指定します。
   
   |設定|値|
   |---|---|
   |ユーザー名|**学生**|
   |パスワード|**Pa55w.rd1234**|

1. Azure VM で実行されているオペレーティング システムがリモート デスクトップ経由で正常にアクセスできることを確認し、リモート デスクトップ セッションを閉じます。

## <a name="exercise-6-deprovisioning-the-azure-environment"></a>演習 6: Azure 環境のプロビジョニング解除

### <a name="scenario"></a>シナリオ

Azure 関連の料金を最小限に抑えるため、このラボでプロビジョニングした Azure リソースをプロビジョニング解除します。

この演習の主なタスクは次のとおりです。

1. Cloud Shell で PowerShell セッションを開始する。
1. ラボでプロビジョニングしたすべての Azure リソースを特定する。

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>タスク 1: Cloud Shell で PowerShell セッションを開始する

1. Azure portal の [Azure Cloud Shell] ウィンドウで PowerShell セッションを開きます。
1. **Cloud Shell** を開始するのが初めての場合は、既定の設定を受け入れます。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>タスク 2: ラボでプロビジョニングしたすべての Azure リソースを特定する

1. [Cloud Shell] ウィンドウで次のコマンドを実行して、このラボで作成されたすべてのリソース グループのリストを表示します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*'
   ```
1. 次のコマンドを実行して、このラボで作成したすべてのリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*' | Remove-AzResourceGroup -Force -AsJob
   ```

## <a name="results"></a>結果

このラボを完了すると、Contoso, Ltd. の管理容易性とセキュリティの要件を満たす方法で Windows Server を実行している Azure VM がデプロイおよび構成されます。