---
lab:
  title: 'ラボ: ハイブリッド シナリオでの Windows Admin Center の使用'
  type: Answer Key
  module: 'Module 4: Facilitating hybrid management'
---

# <a name="lab-answer-key-using-windows-admin-center-in-hybrid-scenarios"></a>ラボの回答キー: ハイブリッド シナリオでの Windows Admin Center の使用

## <a name="exercise-1-provisioning-azure-vms-running-windows-server"></a>                **メモ:** このラボをご自分のペースでクリックして進めることができる、 **[ラボの対話型シミュレーション](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Using%20Windows%20Admin%20Center%20in%20hybrid%20scenarios)** が用意されています。

#### <a name="task-1-create-an-azure-resource-group-by-using-an-azure-resource-manager-template"></a>対話型シミュレーションとホストされたラボの間に若干の違いがある場合がありますが、示されている主要な概念とアイデアは同じです。

1. 演習 1: Windows Server を実行する Azure VM のプロビジョニング
1. タスク 1: Azure Resource Manager テンプレートを使用して Azure リソース グループを作成する
1. **SEA-ADM1** に接続し、必要に応じて、パスワード **Pa55w.rd** を使用し、**CONTOSO\\Administrator** としてサインインします。
1. **SEA-ADM1** で Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。

   >Azure portal で、[検索] テキスト ボックスの横にあるツール バー アイコンを選択して、[Cloud Shell] ウィンドウを開きます。

1. **Bash** または **PowerShell** の選択を求めるプロンプトが表示されたら、**[PowerShell]** を選択します。
1. **注**: Cloud Shell を起動するのが初めてで、"**ストレージがマウントされていません**" というメッセージが表示される場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 [Cloud Shell] ウィンドウのツールバーで、**[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択し、**C:\\Labfiles\\Lab04\\L04-sub_template.json** ファイルを Cloud Shell のホーム ディレクトリにアップロードします。

   >[Cloud Shell] ウィンドウで、次のコマンドを実行して、このラボでプロビジョニングするリソースを含めるリソース グループを作成します  (`<Azure region>` プレースホルダーを、Azure 仮想マシンをデプロイできる Azure リージョンの名前 (**eastus** など) に置き換えます)。

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

#### <a name="task-2-create-an-azure-vm-by-using-an-azure-resource-manager-template"></a>**注**: このラボは、米国東部を使用してテストおよび検証されているため、このリージョンを使用してください。

1. 通常、Azure VM をプロビジョニングできる Azure リージョンを特定するには、「[ご利用のリージョンの Azure クレジット プランを確認する](https://aka.ms/regions-offers)」を参照してください。
1. タスク 2: Azure Resource Manager テンプレートを使用して Azure VM を作成する

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az800l04rgDeployment `
     -ResourceGroupName $rgName `
     -TemplateFile $HOME/L04-rg_template.json `
     -TemplateParameterFile $HOME/L04-rg_template.parameters.json
   ```

   >[Cloud Shell] ウィンドウで、Azure Resource Manager テンプレート **C:\\Labfiles\\Lab04\\L04-rg_template.json**、および対応する Azure Resource Manager パラメーター ファイル **C:\\Labfiles\\Lab04\\L04-rg_template.parameters.json** をアップロードします。 [Cloud Shell] ウィンドウで、次のコマンドを実行して、このラボで使用する、Windows Server を実行している Azure VM をデプロイします。

1. **注**: このデプロイが完了するまで待ってから、次の演習に進んでください。
1. デプロイには約 5 分かかります。
1. Azure portal の [Cloud Shell] ウィンドウを閉じます。
1. Azure portal で、ツール バーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで「**az800l04-vnet**」仮想ネットワークを検索して選択します。

## <a name="exercise-2-implementing-hybrid-connectivity-by-using-the-azure-network-adapter"></a>**[az800l04-vnet]** ページで **[サブネット]** を選択し、**[サブネット]** ページで **+ [ゲートウェイ サブネット]** を選択します。

#### <a name="task-1-register-windows-admin-center-with-azure"></a>**[サブネットの追加]** ページで、**[サブネット アドレス範囲]** を **[10.4.3.224/27]** に設定し、**[保存]** を選択して **GatewaySubnet** を作成します。

1. 演習 2: Azure ネットワーク アダプターを使用したハイブリッド接続の実装

   >タスク 1: Windows Admin Center を Azure に登録する

1. **SEA-ADM1** で **[スタート]** を選択し、**[Windows PowerShell (管理者)]** を選択します。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. **注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **[Windows PowerShell]** コンソールで、次のコマンドを入力し、Enter キーを押して、最新バージョンの Windows Admin Center をダウンロードします。 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。

1. **注**: インストールが完了するまで待ちます。

   >これには 2 分ほどかかります。 **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` にアクセスします。 
   
1. **注**: リンクが機能しない場合は、**SEA-ADM1** で **WindowsAdminCenter.msi** ファイルにアクセスし、コンテキスト メニューを開いて **[修復]** を選択します。

   - 修復が完了した後、Microsoft Edge を更新します。
   - メッセージが表示されたら、**[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力し、**[OK]** を選択します。

1. ユーザー名: **CONTOSO\\Administrator** 
1. パスワード: **Pa55w.rd**

   > **[すべての接続]** ページで **[sea-adm1.contoso.com]** エントリを選択します。

1. Windows Admin Center で、**[ネットワーク]**、**[アクション]**、**[+ Azure ネットワーク アダプター (プレビュー) の追加]** の順に選択します。

   >**注**: **[アクション]** メニューが使用できない場合、画面の解像度によっては、**省略記号**アイコンを選択することが必要になる場合があります。

1. メッセージが表示されたら、**[Azure ネットワーク アダプターの追加]** ウィンドウで、**[Windows Admin Center を Azure に登録する]** を選択します。
1. **注**: これにより、Windows Admin Center 内の **[設定]** ページに [Azure] ウィンドウが自動的に表示されます。 
1. Windows Admin Center の [Azure] ウィンドウの **[設定]** ページで、 **[登録]** を選択します。

   >**[Windows Admin Center で Azure を使用して作業を開始する]** ウィンドウで **[コピー]** を選択して、登録手順のステップの一覧に表示されたコードをコピーします。

1. 登録手順のステップの一覧で **[コードの入力]** を選択します。
1. **注**: これにより、Microsoft Edge ウィンドウに別のタブが開き、**[コードの入力]** ページが表示されます。
1. **[コードの入力]** テキスト ボックスに、クリップボードにコピーしたコードを貼り付け、**[次へ]** を選択します。
1. **[サインイン]** ページで、前の演習で Azure サブスクリプションへのサインインに使用したものと同じユーザー名を入力し、**[次へ]** を選択します。次に、対応するパスワードを入力し、**[サインイン]** を選択します。
1. "**Are you trying to sign in to Windows Admin Center?**" (Windows Admin Center にサインインしますか?) というメッセージが表示されたら、**[続行]** を選びます。
1. Windows Admin Center で、正常にサインインしたことを確認し、Microsoft Edge ウィンドウに新しく開いたタブを閉じます。 **[Windows Admin Center で Azure を使用して作業を開始する]** ウィンドウで、**Azure Active Directory アプリケーション**が **[新規作成]** に設定されていることを確認した後、**[接続]** を選択します。
1. 登録手順のステップの一覧で **[サインイン]** を選択します。

#### <a name="task-2-create-an-azure-network-adapter"></a>これにより、**[要求されたアクセス許可]** というラベルの付いたポップアップ ウィンドウが開きます。

1. **[要求されたアクセス許可]** ポップアップ ウィンドウで、**[組織の代理として同意する]**、**[同意する]** の順に選択します。
1. タスク 2: Azure ネットワーク アダプターを作成する
1. **SEA-ADM1**で、Windows Admin Center が表示されている Microsoft Edge ウィンドウに戻り、 **[sea-adm1.contoso.com]** ページにアクセスして、 **[ネットワーク]** を選択します。

   |Windows Admin Center の **[ネットワーク]** ページで、 **[アクション]** メニューから **[+ Azure ネットワーク アダプター (プレビュー) の追加]** エントリを再度選択します。|**[Add Azure Network Adapter]\(Azure ネットワーク アダプターの追加\)** 設定ペインで、次の設定を指定して、 **[作成]** を選びます (他の設定は既定値のままにします)。|
   |---|---|
   |設定|値|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |場所|eastus|
   |仮想ネットワーク|az800l04-vnet|
   |ゲートウェイ サブネット|10.4.3.224/27|
   |ゲートウェイ SKU|VpnGw1|
   |クライアント アドレス空間|192.168.0.0/24|

1. 認証証明書
1. 自動生成された自己署名ルート証明書とクライアント証明書

>**SEA-ADM1** の Azure portal が表示されている Microsoft Edge ウィンドウのツール バーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**仮想ネットワーク ゲートウェイ**」を検索して選びます。 **[仮想ネットワーク ゲートウェイ]** ページで **[更新]** を選び、仮想ネットワーク ゲートウェイの一覧に、名前が **WAC-Created-vpngw-96** で始まる新しいエントリが表示されることを確認します。

## <a name="exercise-3-deploying-windows-admin-center-gateway-in-azure"></a>**注**: Azure 仮想ネットワーク ゲートウェイのプロビジョニングには最大 45 分かかります。

#### <a name="task-1-install-windows-admin-center-gateway-in-azure"></a>プロビジョニングが完了するのを待たずに、次の演習に進んでください。

1. 演習 3: Azure での Windows Admin Center ゲートウェイのデプロイ
1. タスク 1: Azure で Windows Admin Center ゲートウェイをインストールする
1. **SEA-ADM1** で、Azure portal が表示されているブラウザー ウィンドウに切り替えます。
1. Azure portal に戻り、**[Cloud Shell]** アイコンを選択して、[Cloud Shell] ウィンドウを開きます。

   ```powershell
   Enable-AzureRmAlias -Scope Process
   ```

1. [Cloud Shell] ウィンドウのツール バーで **[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択して、**C:\\Labfiles\\Lab04\\Deploy-WACAzVM.ps1** ファイルを Cloud Shell のホーム ディレクトリにアップロードします。

   ```powershell
   $rgName = 'AZ800-L0401-RG'
   $vnetName = 'az800l04-vnet'
   $nsgName = 'az800l04-web-nsg'
   $subnetName = 'subnet1'
   $location = 'eastus'
   $pipName = 'wac-public-ip'
   $size = 'Standard_D2s_v3'
   ```

1. [Cloud Shell] ウィンドウで、次のコマンドを実行して、Windows Admin Center プロビジョニング スクリプトで使用される **AzureRm** PowerShell コマンドレットの互換性を有効にします。

   ```powershell
   $scriptParams = @{
     ResourceGroupName = $rgName
     Name = 'az800l04-vmwac'
     VirtualNetworkName = $vnetName
     SubnetName = $subnetName
     GenerateSslCert = $true
     size = $size
     PublicIPAddressName = $pipname
   }
   ```

1. [Cloud Shell] ウィンドウで、次のコマンドを実行して、Windows Admin Center プロビジョニング スクリプトを実行するために必要な変数の値を設定します。

   ```powershell
   Install-Module -Name pswsman
   Disable-WSManCertVerification -All
   ```

1. [Cloud Shell] ウィンドウで、次のコマンドを実行して、スクリプト パラメーター変数を設定します。

   ```powershell
   ./Deploy-WACAzVM.ps1 @scriptParams
   ```

1. [Cloud Shell] ペインから、次のコマンドを実行して、PowerShell リモート処理の証明書の検証を無効にします (信頼されていないリポジトリからのインストールを確認するように求めるメッセージが表示されたら、「**A**」を入力し、Enter キーを押します)。
1. [Cloud Shell] ウィンドウから、次のコマンドを実行して、プロビジョニング スクリプトを起動します。

   >ローカル Administrator アカウントの名前を入力するように求めるダイアログが表示されたら、「**Student**」と入力します ローカル Administrator アカウントのパスワードを入力するように求めるダイアログが表示されたら、「**Pa55w.rd1234**」と入力します

1. **注**: プロビジョニング スクリプトが完了するまで待ちます。

   >これには 5 分ほどかかる場合があります。 スクリプトが正常に完了したことを確認し、Windows Admin Center インストールをホストする Azure VM の完全修飾名を含む URL を示す最後のメッセージに注意してください。

1. **注**: Azure VM の完全修飾名を記録します。

#### <a name="task-2-review-results-of-the-script-provisioning"></a>このラボで後ほど必要になります。

1. [Cloud Shell] ペインを閉じます。
1. タスク 2: スクリプト プロビジョニングの結果を確認する
1. Azure portal のツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**リソース グループ**」を検索して選択します。次に、**[リソース グループ]** ページで **[AZ800-L0401-RG]** エントリを選択します。
1. **[AZ800-L0401-RG]** の **[概要]** ページで、Azure VM の **az800l04-vmwac** を含むリソースの一覧を確認します。

## <a name="exercise-4-verifying-functionality-of-the-windows-admin-center-gateway-in-azure"></a>リソースの一覧で Azure VM **[az800l04-vmwac]** エントリを選択し、**[az800l04-vmwac]** ページで **[ネットワーク]** を選択します。

#### <a name="task-1-connect-to-the-windows-admin-center-gateway-running-in-azure-vm"></a>**[az800l04-vmwac | ネットワーク]** ページの **[受信ポートの規則]** タブで、TCP ポート 5986 で接続を許可する受信ポートの規則、および TCP ポート 443 で接続を許可する受信規則をそれぞれ示すエントリに注目します。

1. 演習 4: Azure での Windows Admin Center ゲートウェイの機能の確認
1. タスク 1: Azure VM で実行されている Windows Admin Center ゲートウェイに接続する
1. **SEA-ADM1**で Microsoft Edge を起動し、前の演習で特定した Windows Admin Center インストールをホストするターゲット Azure VM の完全修飾名を含む URL に移動します。
1. Microsoft Edge ウィンドウで、"**お使いの接続は、プライベートではありません**" というメッセージを無視し、**[詳細]** を選択し、テキスト "**に続く**" を含むリンクを選択します。
1. メッセージが表示されたら、**[サインインしてこのサイトにアクセスする]** ダイアログ ボックスで、ユーザー名 **Student** とパスワード **Pa55w.rd1234** を使用してサインインします。

#### <a name="task-2-enable-powershell-remoting-on-an-azure-vm"></a>Windows Admin Center の **[すべての接続]** ページで、**[az800l04-vmwac [ゲートウェイ]]** を選択します。

1. Windows Admin Center の [概要] ウィンドウを確認します。
1. タスク 2: Azure VM の PowerShell リモート処理を有効にする
1. **SEA-ADM1**で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えて、ツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**仮想マシン**」を検索して選択します。
1. **[仮想マシン]** ページで、**[az800l04-vm0]** を選択します。

   ```powershell
   winrm quickconfig -quiet
   ```

1. **[az800l04-vm0]** ページの **[操作]** セクションで、**[コマンドの実行]** を選択し、**[RunPowerShellScript]** を選択します。

   ```powershell
   Set-NetFirewallRule -Name WINRM-HTTP-In-TCP-PUBLIC -RemoteAddress Any
   ```

1. Windows リモート管理が無効になっている場合は、**[コマンド スクリプトの実行]** ページの **[PowerShell スクリプト]** セクションで、次のコマンドを入力し、**[実行]** を選択して有効にします。

   ```powershell
   Enable-PSRemoting -Force -SkipNetworkProfileCheck
   ```

#### <a name="task-3-connect-to-an-azure-vm-by-using-the-windows-admin-center-gateway-running-in-azure-vm"></a>**[PowerShell スクリプト]** セクションで、前の手順で入力したテキストを次のコマンドに置き換え、**[実行]** を選択して、Windows リモート管理受信ポートを開きます。

1. **[PowerShell スクリプト]** セクションで、前の手順で入力したテキストを次のコマンドに置き換え、**[実行]** を選択して、PowerShell リモート処理を有効にします。
1. タスク 3: Azure VM で実行されている Windows Admin Center ゲートウェイを使用して、Azure VM に接続する
1. **SEA-ADM1** で、**az800l04-vmwac** Azure VM 上で実行されている Windows Admin Center ゲートウェイのインターフェイスが表示されている Microsoft Edge ウィンドウで、**[Windows Admin Center]** を選択します。
1. **[すべての接続]** ページで、**[+ 追加]** を選択します。
1. **[リソースの追加または作成]** ページの **[サーバー]** セクションで、**[追加]** を選択します。
1. **[サーバー名]** テキスト ボックスに 「**az800l04-vm0**」と入力します。
1. **[この接続に別のアカウントを使用する]** オプションを選択し、ユーザー名 **Student** とパスワード **Pa55w.rd1234** を指定し、**[資格情報を使って追加する]** を選択します。

## <a name="exercise-5-deprovisioning-the-azure-environment"></a>接続の一覧で、**[az800l04-vm0]** を選択します

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>Azure VM に正常に接続されたら、Windows Admin Center の **az800l04-vmwac** Azure VM の [概要] ウィンドウを確認します。

1. 演習 5: Azure 環境のプロビジョニング解除
1. タスク 1: Cloud Shell で PowerShell セッションを開始する

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>**SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。

1. Azure portal が表示されている Microsoft Edge ウィンドウで、[Cloud Shell] アイコンを選択して [Cloud Shell] ウィンドウを開きます。

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L040*'
   ```

1. タスク 2: ラボでプロビジョニングしたすべての Azure リソースを特定する

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L040*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >[Cloud Shell] ウィンドウで次のコマンドを実行して、このラボで作成されたすべてのリソース グループの一覧を表示します。
