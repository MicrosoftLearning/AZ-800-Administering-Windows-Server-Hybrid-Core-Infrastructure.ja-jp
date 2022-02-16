---
lab:
  title: 'ラボ: ハイブリッド シナリオでの Windows Admin Center の使用'
  module: 'Module 4: Facilitating hybrid management'
ms.openlocfilehash: e380f49ae9b53ee46e48a21ac40e40cbddd51852
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907023"
---
# <a name="lab-using-windows-admin-center-in-hybrid-scenarios"></a>ラボ: ハイブリッド シナリオでの Windows Admin Center の使用

## <a name="scenario"></a>シナリオ

マネージド システムの場所に関係なく、一貫した運用と管理のモデルに関する懸念事項に対処するには、オンプレミスと Microsoft Azure 仮想マシン (VM) で実行されているさまざまなバージョンの Windows Server オペレーティング システムを含むハイブリッド環境で Windows Admin Center の機能をテストします。

目標は、マネージド システムの場所に関係なく、Windows Admin Center を一貫して使用できることの確認です。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure ネットワーク アダプターを使用してハイブリッド接続をテストします。
- Azure で Windows Admin Center ゲートウェイをデプロイします。
- Azure で Windows Admin Center ゲートウェイの機能を検証します。

## <a name="estimated-time-90-minutes"></a>予想所要時間: 90 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1** および **AZ-800T00A-ADM1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-800T00A-SEA-DC1** VM と **AZ-800T00A-SEA-ADM1** VM で **SEA-DC1** と **SEA-ADM1** のインストールをホストしています

1. **SEA-ADM1** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、使用可能な VM 環境と Azure サブスクリプションを使用します。 ラボを開始する前に、Azure サブスクリプションがあり、そのサブスクリプションの所有者ロールまたは共同作成者ロールを持ち、そのサブスクリプションに関連付けられている Azure Active Directory (Azure AD) テナントのグローバル管理者ロールを持っているユーザー アカウントがあることを確認します。

## <a name="exercise-1-provisioning-azure-vms-running-windows-server"></a>演習 1: Windows Server を実行する Azure VM のプロビジョニング

### <a name="scenario"></a>シナリオ

オンプレミス サーバーと Azure 仮想ネットワークの間にハイブリッド接続を確立できることを確認する必要があります。 そのためにはまず、Azure Resource Manager テンプレートを使用して、Windows Server を実行する Azure VM をプロビジョニングします。

この演習の主なタスクは次のとおりです。

1. Azure Resource Manager テンプレートを使用して Azure リソース グループを作成する。
1. Azure Resource Manager テンプレートを使用して Azure VM を作成する。

#### <a name="task-1-create-an-azure-resource-group-by-using-an-azure-resource-manager-template"></a>タスク 1: Azure Resource Manager テンプレートを使用して Azure リソース グループを作成する

1. **SEA-ADM1** で Microsoft Edge を起動し、Azure portal を参照して、Azure 資格情報で認証します。
1. Azure portal の ［Cloud Shell］ ペインで PowerShell セッションを開きます。
1. **C:\\Labfiles\\Lab04\\L04-sub_template.json** ファイルを Cloud Shell ホーム ディレクトリにアップロードします。
1. Cloud Shell］ ペインで、次のコマンドを実行して、このラボでプロビジョニングするリソースが入ることになるリソース グループを作成します。 (`<Azure region>` プレースホルダーを、Azure 仮想マシンをデプロイできる Azure リージョンの名前 （**eastus** など) に置き換えます)。

   >**注**: このラボは East US を使用してテストおよび検証されているため、このリージョンを使用してください。 通常、Azure VM をプロビジョニングできる Azure リージョンを特定する場合には、「[ご利用のリージョンの Azure クレジット プランを確認する](https://aka.ms/regions-offers)」を参照してください。

   ```powershell
   $location = '<Azure region>'
   $rgName = 'AZ800-L0401-RG'
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az800l04subDeployment `
     -TemplateFile $HOME/L04-sub_template.json `
     -rgLocation $location `
     -rgName $rgName
   ```

#### <a name="task-2-create-an-azure-vm-by-using-an-azure-resource-manager-template"></a>タスク 2: Azure Resource Manager テンプレートを使用して Azure VM を作成する

1. [Cloud Shell] ペインで、Azure Resource Manager テンプレート **C:\\Labfiles\\Lab04\\L04-rg_template.json**、および対応する Azure Resource Manager パラメーター ファイル **C:\\Labfiles\\Lab04\\L04-rg_template.parameters.json** をアップロードします。
1. [Cloud Shell] ペインで、次のコマンドを実行して、このラボで使用する、Windows Server を実行している Azure VM をデプロイします。

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az800l04rgDeployment `
     -ResourceGroupName $rgName `
     -TemplateFile $HOME/L04-rg_template.json `
     -TemplateParameterFile $HOME/L04-rg_template.parameters.json
   ```

   >**注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 デプロイには約 5 分かかります。

1. Azure portal の [Cloud Shell] ペインを閉じます。
1. Azure portal で、**az800l04-vnet** 仮想ネットワークに対して、IP アドレスの範囲が **10.4.3.224/27** の **GatewaySubnet** を追加します。

## <a name="exercise-2-implementing-hybrid-connectivity-by-using-the-azure-network-adapter"></a>演習 2: Azure ネットワーク アダプターを使用したハイブリッド接続の実装

### <a name="scenario"></a>シナリオ

オンプレミス サーバーと前の演習でプロビジョニングした Azure VM の間にハイブリッド接続を確立できることを確認する必要があります。 この目的のために、Windows Admin Center の Azure ネットワーク アダプター機能を使用します。

この演習の主なタスクは次のとおりです。

1. Windows Admin Center を Azure に登録する。
1. Azure ネットワーク アダプターを作成する。

#### <a name="task-1-register-windows-admin-center-with-azure"></a>タスク 1: Windows Admin Center を Azure に登録する

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を開始します。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を実行します。

1. **Windows PowerShell** コンソールで、次のコマンドを実行してから Enter キーを押し、Windows Admin Center の最新バージョンをダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` で Windows Admin Center のローカル インスタンスに接続します。 

   >**注**: リンクが機能しない場合には、**SEA-ADM1** で **WindowsAdminCenter.msi** ファイルを参照し、コンテキスト メニューを開いて **[修復]** を選択します。 修復が完了したら、Microsoft Edge を最新の情報に更新します。 

1. ダイアログが表示されたら、 **[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力し、 **[OK]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. Windows Admin Center ページで、Azure ネットワーク アダプターの追加を試します。
1. メッセージが表示されたら、前の演習で使用した Azure サブスクリプションに Windows Admin Center を登録します。

#### <a name="task-2-create-an-azure-network-adapter"></a>タスク 2: Azure ネットワーク アダプターを作成する

1. **SEA-ADM1** で、Windows Admin Center が表示されている Microsoft Edge ウィンドウを使用して、もう一度 Azure ネットワーク アダプターの作成を試みます。
1. 次の設定で Azure ネットワーク アダプターを作成します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |場所|eastus|
   |仮想ネットワーク|az800l04-vnet|
   |ゲートウェイ サブネット|10.4.3.224/27|
   |ゲートウェイ SKU|VpnGw1|
   |クライアント アドレス空間|192.168.0.0/24|
   |認証証明書|自動生成された自己署名ルート証明書とクライアント証明書|

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替え、**WAC-Created-vpngw-** で始まる名前の新しい仮想ネットワーク ゲートウェイがプロビジョニングされていることを確認します。

   >**注**: Azure 仮想ネットワーク ゲートウェイのプロビジョニングには最大 45 分かかります。 プロビジョニングが完了するのを待たずに、次の演習に進んでください。

## <a name="exercise-3-deploying-windows-admin-center-gateway-in-azure"></a>演習 3: Azure での Windows Admin Center ゲートウェイのデプロイ

### <a name="scenario"></a>シナリオ

Windows Admin Center を使用して、Windows Server OS を実行している Azure VM を管理する機能を評価する必要があります。 これを行うには、まず、このラボの最初の演習で実装した Azure 仮想ネットワークに Windows Admin Center ゲートウェイをインストールします。

この演習の主なタスクは次のとおりです。

1. Azure で Windows Admin Center ゲートウェイをインストールする。
1. スクリプト プロビジョニングの結果を確認する。

#### <a name="task-1-install-windows-admin-center-gateway-in-azure"></a>タスク 1: Azure で Windows Admin Center ゲートウェイをインストールする

1. **SEA-ADM1** で、Azure portal が表示されているブラウザー ウィンドウに切り替えます。
1. Azure portal の ［Cloud Shell］ ペインで PowerShell セッションを開始します。
1. [Cloud Shell] ペインで、**C:\\Labfiles\\Lab04\\Deploy-WACAzVM.ps1** ファイルを Cloud Shell ホーム ディレクトリにアップロードします。
1. [Cloud Shell] ペインで、次のコマンドを実行して、Windows Admin Center プロビジョニング スクリプトが使用する **AzureRm** PowerShell コマンドレットの互換性を有効にします。

   ```powershell
   Enable-AzureRmAlias -Scope Process
   ```
1. 次のコマンドを実行して、Windows Admin Center プロビジョニング スクリプトを実行するために必要な変数値を設定します。

   ```powershell
   $rgName = 'AZ800-L0401-RG'
   $vnetName = 'az800l04-vnet'
   $nsgName = 'az800l04-web-nsg'
   $subnetName = 'subnet1'
   $location = 'eastus'
   $pipName = 'wac-public-ip'
   $size = 'Standard_D2s_v3'
   ```
1. 次のコマンドを実行して、スクリプト パラメーター変数を設定します。

   ```powershell
   $scriptParams = @{
     ResourceGroupName = $rgName
     Name = 'az800l04-vmwac'
     VirtualNetworkName = $vnetName
     SubnetName = $subnetName
     GenerateSslCert = $true
     size = $size
   }
   ```
1. 次のコマンドを実行して、PowerShell リモート処理の証明書の検証を無効にします。

   ```powershell
   install-module pswsman
   Disable-WSManCertVerification -All
   ```
1. 次のコマンドを実行して、プロビジョニング スクリプトを起動します。

   ```powershell
   ./Deploy-WACAzVM.ps1 @scriptParams
   ```
1. ローカル管理者アカウントの名前を入力するように求められたら、「**Student**」と入力します。
1. ローカル管理者アカウントのパスワードを入力するように求められたら、「**Pa55w.rd1234**」と入力します。

    >**注**: プロビジョニング スクリプトが完了するまで待ちます。 これには 5 分ほどかかる場合があります。

1. スクリプトが正常に完了したことを確認し、Windows Admin Center インストールをホストする Azure VM の完全修飾名が含まれる URL が記載された最後のメッセージに注目します。

   >**注**: Azure VM の完全修飾名を記録します。 このラボで後ほど必要になります。

1. [Cloud Shell] ペインを閉じます。

#### <a name="task-2-review-results-of-the-script-provisioning"></a>タスク 2: スクリプト プロビジョニングの結果を確認する

1. Azure portal で、**AZ800-L0401-RG** リソース グループのページを参照します。
1. **AZ800-L0401-RG** ページの **[概要]** ページで、Azure VM **az800l04-vmwac** が含まれるリソースのリストを確認します。
1. **az800l04-vmwac | Networking** ページの **[受信ポートの規則]** タブで、TCP ポート 5986 で接続を許可する受信ポートの規則、および TCP ポート 443 で接続を許可する受信規則をそれぞれ示すエントリに注目します。

## <a name="exercise-4-verifying-functionality-of-the-windows-admin-center-gateway-in-azure"></a>演習 4: Azure での Windows Admin Center ゲートウェイの機能の確認

### <a name="scenario"></a>シナリオ

必須コンポーネントがすべて揃っている状態で、このラボの最初の演習でプロビジョニングした Azure 仮想ネットワークにデプロイした Azure VM をターゲットとして、WAC 機能をテストします。

この演習の主なタスクは次のとおりです。

1. Azure VM で実行されている Windows Admin Center ゲートウェイに接続する。
1. Azure VM の PowerShell リモート処理を有効にする。
1. Azure VM で実行されている Windows Admin Center ゲートウェイを使用して、Azure VM に接続する。

#### <a name="task-1-connect-to-the-windows-admin-center-gateway-running-in-azure-vm"></a>タスク 1: Azure VM で実行されている Windows Admin Center ゲートウェイに接続する

1. **SEA-ADM1** で Microsoft Edge を起動し、前の演習で特定した、Azure VM を実行している Windows Admin Center ゲートウェイに接続します。
1. メッセージが表示されたら、ユーザー名 **Student** とパスワード **Pa55w.rd1234** でサインインします。
1. Windows Admin Center の [すべての接続] ペインで、**az800l04-vmwac [ゲートウェイ]** を選択します。
1. Windows Admin Center の [概要] ペインを確認します。

#### <a name="task-2-enable-powershell-remoting-on-an-azure-vm"></a>タスク 2: Azure VM の PowerShell リモート処理を有効にする

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウで、**az800l04-vm0** Azure VM のページを参照します。
1. **az800l04-vm0** ページで、Windows リモート管理が無効な場合には **[ファイル名を指定して実行]** 機能で以下のコマンドを実行して有効にします。

   ```powershell
   winrm quickconfig -quiet
   ```

1. **[ファイル名を指定して実行]** 機能で以下のコマンドを実行して、Windows リモート管理受信ポートを開きます。

   ```powershell
   Set-NetFirewallRule -Name WINRM-HTTP-In-TCP-PUBLIC -RemoteAddress Any
   ```

1. **[ファイル名を指定して実行]** 機能で以下のコマンドを実行して、PowerShell リモート処理を有効にします。

   ```powershell
   Enable-PSRemoting -Force -SkipNetworkProfileCheck
   ```

#### <a name="task-3-connect-to-an-azure-vm-by-using-the-windows-admin-center-gateway-running-in-azure-vm"></a>タスク 3: Azure VM で実行されている Windows Admin Center ゲートウェイを使用して、Azure VM に接続する

1. **SEA-ADM1** で、**az800l04-vmwac** Azure VM で実行されている Windows Admin Center ゲートウェイのインターフェイスが表示されている Microsoft Edge ウィンドウで、**az800l04-vm0** という名前を使用してこの Azure VM への接続を追加します。
1. メッセージが表示されたら、ユーザー名 **Student**、パスワード **Pa55w.rd1234** を使用して認証します。
1. VM に正常に接続されたら、Windows Admin Center の **az800l04-vmwac** Azure VM の [概要] ペインを確認します。

## <a name="exercise-5-deprovisioning-the-azure-environment"></a>演習 5: Azure 環境のプロビジョニング解除

### <a name="scenario"></a>シナリオ

Azure 関連の料金を最小限に抑えるため、このラボでプロビジョニングした Azure リソースをプロビジョニング解除します。

この演習の主なタスクは次のとおりです。

1. Cloud Shell で PowerShell セッションを開始する。
1. ラボでプロビジョニングしたすべての Azure リソースを特定する。

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>タスク 1: Cloud Shell で PowerShell セッションを開始する

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。
1. Azure portal の ［Cloud Shell］ ペインで PowerShell セッションを開きます。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>タスク 2: ラボでプロビジョニングしたすべての Azure リソースを特定する

1. [Cloud Shell] ペインで次のコマンドを実行して、このラボで作成されたすべてのリソース グループのリストを表示します。

   ```powershell
   Get-AzResourceGroup -Name 'az800l04*'
   ```

1. 次のコマンドを実行して、このラボで作成したすべてのリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'az800l04*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**注**: このコマンドは非同期で実行されます (-AsJob パラメーターによって決定されます)。 そのため、同じ PowerShell セッション内ですぐに別の PowerShell コマンドを実行できるようになりますが、リソース グループが実際に削除されるまでに数分かかります。

### <a name="results"></a>結果

このラボを完了すると、Contoso の管理容易性とセキュリティの要件を満たす、Windows Server を実行している Azure VM がデプロイおよび構成されました。

### <a name="prepare-for-the-next-module"></a>次のモジュールの準備

次のモジュールの準備が完了したら、ラボを終了します。
