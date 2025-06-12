---
lab:
  title: 'ラボ: ハイブリッド シナリオでの Windows Admin Center の使用'
  type: Answer Key
  module: 'Module 4: Facilitating hybrid management'
---

# ラボの回答キー: ハイブリッド シナリオでの Windows Admin Center の使用

## 演習 1: Windows Server を実行する Azure VM のプロビジョニング

#### タスク 1: Azure Resource Manager テンプレートを使用して Azure リソース グループを作成する

1. **SEA-ADM1** に接続し、必要に応じて、講師から提供された資格情報でサインインします。
1. **SEA-ADM1** で Microsoft Edge を起動して `https://portal.azure.com` にある Azure portal を開き、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。
1. Azure portal で、[検索] テキスト ボックスの横にあるツール バー アイコンを選択して、[Cloud Shell] ウィンドウを開きます。
1. **Bash** または **PowerShell** の選択を求めるプロンプトが表示されたら、**[PowerShell]** を選択します。

   >**注**: Cloud Shell を起動するのが初めてで、"**ストレージがマウントされていません**" というメッセージが表示される場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。

1. [Cloud Shell] ウィンドウのツールバーで、**[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択し、**C:\\Labfiles\\Lab04\\L04-sub_template.json** ファイルを Cloud Shell のホーム ディレクトリにアップロードします。
1. [Cloud Shell] ウィンドウで、次のコマンドを実行して、このラボでプロビジョニングするリソースを含めるリソース グループを作成します (`<Azure region>` プレースホルダーを、Azure 仮想マシンをデプロイできる Azure リージョンの名前 (**eastus** など) に置き換えます)。

   >**注**: このラボは、米国東部を使用してテストおよび検証されているため、このリージョンを使用してください。 通常、Azure VM をプロビジョニングできる Azure リージョンを特定するには、「[ご利用のリージョンの Azure クレジット プランを確認する](https://aka.ms/regions-offers)」を参照してください。

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

#### タスク 2: Azure Resource Manager テンプレートを使用して Azure VM を作成する

1. Cloud Shell 画面から、Azure Resource Manager テンプレート **C:\\Labfiles\\Lab04\\L04-rg_template.json** をアップロードします。
1. [Cloud Shell] ウィンドウで、次のコマンドを実行して、このラボで使用する、Windows Server を実行している Azure VM をデプロイします。

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az800l04rgDeployment `
     -ResourceGroupName $rgName `
     -TemplateFile $HOME/L04-rg_template.json
   ```

1. メッセージが表示されたら、講師から提供された資格情報を挿入します。

   >**注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 デプロイには約 5 分かかります。

1. Azure portal の [Cloud Shell] ウィンドウを閉じます。
1. Azure portal で、ツール バーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで「**az800l04-vnet**」仮想ネットワークを検索して選択します。
1. **[az800l04-vnet]** ページの **[設定]** セクションで **[サブネット]** を選択し、**[サブネット]** ページで **[+ サブネット]** を選択します。
1. **[サブネットの追加]** ページの設定ウィンドウで、以下の設定を指定して、**[追加]** を選択します (他の設定は既定値のまま)。

   |設定|Value|
   |---|---|
   |サブネットの目的|**仮想ネットワーク ゲートウェイ**|
   |開始アドレス|**10.4.3.224/27**|

## 演習 2: Azure ネットワーク アダプターを使用したハイブリッド接続の実装

#### タスク 1: Windows Admin Center を Azure に登録する

1. **SEA-ADM1** で **[スタート]** を選択し、**[Windows PowerShell (管理者)]** を選択します。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。

1. **[Windows PowerShell]** コンソールで、次のコマンドを入力し、Enter キーを押して、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.exe"
   ```
1. エクスプローラーを開き、**"ダウンロード"** フォルダーに移動して、**WindowsAdminCenter.exe** ファイルを実行します。 この結果、**Windows Admin Center (v2) インストーラー** ウィザードが起動します。
1. **[Windows Admin Center セットアップ ウィザードへようこそ]** ページで、**[次へ]** を選択します。
1. **ライセンス条項とプライバシーに関する声明**のページで、**条項に同意し**、**[次へ]** を選択します。
1. **[インストール モードの選択]** ページで、**[簡単セットアップ]** が選択されていることを確認し、**[次へ]** を選択します。
1. **[TLS 証明書の選択]** ページで、**[自己署名証明書を生成します (有効期限は 60 日です)]** が選択されていることを確認し、**[次へ]** を選択します。
1. **[自動更新]** ページで、**[ダウンロードまたはインストールせずに利用可能な更新プログラムを通知する]** を選択し、**[次へ]** を選択します。
1. **[診断データを Microsoft に送信する]** ページで、**[必須診断データ]** が選択されていることを確認し、**[次へ]** を選択します。
1. **[インストール]** を選択し、インストールが完了したら、**[Windows Admin Center を開始する: `https://SEA-ADM1.contoso.com:443`]** ボックスがオンになっていることを確認し、**[完了]** を選択します。

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` にアクセスします。

   >**注**: リンクが機能しない場合は、**SEA-ADM1** でエクスプローラー開き、ダウンロード フォルダーを選択し、ダウンロード フォルダー内の **WindowsAdminCenter.msi** ファイルを選択し、手動でインストールします。 インストールが完了したら、Microsoft Edge を更新します。

   >**注**: **NET::ERR_CERT_DATE_INVALID** エラーが発生した場合は、Edge ブラウザー ページの **[詳細設定]** を選択し、ページの下部にある **[sea-adm1-contoso.com (アンセーフ) に移動]** を選択します。 
   
1. メッセージが表示されたら、**[Windows セキュリティ]** ダイアログ ボックスに講師から提供された資格情報を入力してから、**[OK]** を選択します。

1. **[すべての接続]** ページで **[sea-adm1.contoso.com]** エントリを選択します。 
1. Windows Admin Center で、 **[ネットワーク]** 、 **[+ Azure ネットワーク アダプター (プレビュー) の追加]** の順に選択します。

   > **注**: **[アクション]** メニューが使用できない場合、画面の解像度によっては、**省略記号**アイコンを選択することが必要になる場合があります。

1. メッセージが表示されたら、**[Azure ネットワーク アダプターの追加]** ウィンドウで、**[Windows Admin Center を Azure に登録する]** を選択します。

   >**注**: これにより、Windows Admin Center 内の **[設定]** ページに [Azure] ウィンドウが自動的に表示されます。

1. Windows Admin Center の [Azure] ウィンドウの **[設定]** ページで、 **[登録]** を選択します。
1. **[Windows Admin Center で Azure を使用して作業を開始する]** ウィンドウで **[コピー]** を選択して、登録手順のステップの一覧に表示されたコードをコピーします。 
1. 登録手順のステップの一覧で **[コードの入力]** を選択します。

   >**注**: これにより、Microsoft Edge ウィンドウに別のタブが開き、**[コードの入力]** ページが表示されます。

1. **[コードの入力]** テキスト ボックスに、クリップボードにコピーしたコードを貼り付け、**[次へ]** を選択します。
1. **[サインイン]** ページで、前の演習で Azure サブスクリプションへのサインインに使用したものと同じユーザー名を入力し、**[次へ]** を選択します。次に、対応するパスワードを入力し、**[サインイン]** を選択します。
1. "**Are you trying to sign in to Windows Admin Center?**" (Windows Admin Center にサインインしますか?) というメッセージが表示されたら、**[続行]** を選びます。
1. Windows Admin Center で、正常にサインインしたことを確認し、Microsoft Edge ウィンドウに新しく開いたタブを閉じます。
1. **[Windows Admin Center で Azure を使用して作業を開始する]** ウィンドウで、**Microsoft Entra アプリケーション**が **[新規作成]** に設定されていることを確認した後、**[接続]** を選択します。
1. 登録手順のステップの一覧で **[サインイン]** を選択します。 これにより、**[要求されたアクセス許可]** というラベルの付いたポップアップ ウィンドウが開きます。
   >注: サインイン時にエラー メッセージが表示される場合は、前の手順でページを更新してみてください。
1. **[要求されたアクセス許可]** ポップアップ ウィンドウで、**[組織の代理として同意する]**、**[同意する]** の順に選択します。

#### タスク 2: Azure ネットワーク アダプターを作成する

>**注**: WAC コンソールの最近加えられた変更により、この手順は現在実行できません。

## 演習 3: Azure での Windows Admin Center ゲートウェイのデプロイ

#### タスク 1: Azure で Windows Admin Center ゲートウェイをインストールする

1. **SEA-ADM1** で、Azure portal が表示されているブラウザー ウィンドウに切り替えます。
1. Azure portal に戻り、**[Cloud Shell]** アイコンを選択して、[Cloud Shell] ウィンドウを開きます。
1. [Cloud Shell] ウィンドウのツール バーで **[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択して、**C:\\Labfiles\\Lab04\\Deploy-WACAzVM.ps1** ファイルを Cloud Shell のホーム ディレクトリにアップロードします。
1. [Cloud Shell] ウィンドウで、次のコマンドを実行して、Windows Admin Center プロビジョニング スクリプトで使用される **AzureRm** PowerShell コマンドレットの互換性を有効にします。

   ```powershell
   Enable-AzureRmAlias -Scope Process
   ```

1. [Cloud Shell] ウィンドウで、次のコマンドを実行して、Windows Admin Center プロビジョニング スクリプトを実行するために必要な変数の値を設定します。

   ```powershell
   $rgName = 'AZ800-L0401-RG'
   $vnetName = 'az800l04-vnet'
   $nsgName = 'az800l04-web-nsg'
   $subnetName = 'subnet1'
   $location = 'eastus'
   $pipName = 'wac-public-ip'
   $size = 'Standard_D2s_v3'
   ```

1. [Cloud Shell] ウィンドウで、次のコマンドを実行して、スクリプト パラメーター変数を設定します。

   ```powershell
   $scriptParams = @{
     ResourceGroupName = $rgName
     Name = 'az800l04-vmwac'
     VirtualNetworkName = $vnetName
     SubnetName = $subnetName
     GenerateSslCert = $true
     size = $size
     PublicIPAddressName = $pipName
     SecurityGroupName = $nsgName
   }
   ```

1. [Cloud Shell] ペインから、次のコマンドを実行して、PowerShell リモート処理の証明書の検証を無効にします (信頼されていないリポジトリからのインストールを確認するように求めるメッセージが表示されたら、「**A**」を入力し、Enter キーを押します)。

   ```powershell
   Install-Module -Name pswsman
   ```

   ```powershell
   Disable-WSManCertVerification -All
   ```

1. [Cloud Shell] ウィンドウから、次のコマンドを実行して、プロビジョニング スクリプトを起動します。

   ```powershell
   ./Deploy-WACAzVM.ps1 @scriptParams
   ```

1. ローカル管理者アカウントの名前を入力するように求められたら、講師から提供された**ユーザー名**を入力します。
1. ローカル管理者アカウントのパスワードを入力するように求められたら、講師から提供された**パスワード**を入力します。

   >**注**: プロビジョニング スクリプトが完了するまで待ちます。 これには 5 分ほどかかる場合があります。
   
1. [Cloud Shell] ペインを閉じます。
1. Azure portal のツールバーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、**[仮想マシン]** を検索して選択し、**[仮想マシン]** ページで **[az800l04-vmwac]** エントリを選択します。
1. **[接続]** セクションで、**[接続]**、**[RDP ファイルのダウンロード]** の順に選択します。
1. メッセージが表示されたら、講師から提供された資格情報を挿入します。
1. **az800l04-vmwac** VM へのリモート デスクトップ セッション内で、**[スタート]** を選択し、**[Windows PowerShell (管理者)]** を選択します。

1. **Windows PowerShell** コンソールで、次のコマンドを入力し、Enter キーを押して、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.exe"
   ```
1. エクスプローラーを開き、**"ダウンロード"** フォルダーに移動して、**WindowsAdminCenter.exe** ファイルを実行します。 この結果、**Windows Admin Center (v2) インストーラー** ウィザードが起動します。
1. **[Windows Admin Center セットアップ ウィザードへようこそ]** ページで、**[次へ]** を選択します。
1. **ライセンス条項とプライバシーに関する声明**のページで、**条項に同意し**、**[次へ]** を選択します。
1. **[インストール モードの選択]** ページで、**[簡単セットアップ]** が選択されていることを確認し、**[次へ]** を選択します。
1. **[TLS 証明書の選択]** ページで、**[自己署名証明書を生成します (有効期限は 60 日です)]** が選択されていることを確認し、**[次へ]** を選択します。
1. **[自動更新]** ページで、**[ダウンロードまたはインストールせずに利用可能な更新プログラムを通知する]** を選択し、**[次へ]** を選択します。
1. **[診断データを Microsoft に送信する]** ページで、**[必須診断データ]** が選択されていることを確認し、**[次へ]** を選択します。
1. **[インストール]** を選択し、インストールが完了したら、**[Windows Admin Center を開始する: `https://az800l04-vmwac:443`]** ボックスがオンになっていることを確認し、**[完了]** を選択します。

  >**注**: インストールには最大で 5 分かかる場合があります。

## 演習 4: Azure での Windows Admin Center ゲートウェイの機能の確認

#### タスク 1: Azure VM で実行されている Windows Admin Center ゲートウェイに接続する

1. **SEA-ADM1** の **[az800l04-vmwac]** ページで、左側メニューの **[概要]** エントリを選択し、**DNS 名**をコピーします。
1. **SEA-ADM1** で Microsoft Edge を起動し、**DNS 名**を`https://` 形式で貼り付けます。
1. Microsoft Edge ウィンドウで、"**お使いの接続は、プライベートではありません**" というメッセージを無視し、**[詳細]** を選択し、テキスト "**に続く**" を含むリンクを選択します。
1. メッセージが表示されたら、 **[サインインしてこのサイトにアクセスする]** ダイアログ ボックスで、講師から提供された資格情報を使用してサインインします。
1. Windows Admin Center の **[すべての接続]** ページで、**[az800l04-vmwac [ゲートウェイ]]** を選択します。
1. Windows Admin Center の [概要] ウィンドウを確認します。

#### タスク 2: Azure VM の PowerShell リモート処理を有効にする

1. **SEA-ADM1**で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えて、ツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**仮想マシン**」を検索して選択します。
1. **[仮想マシン]** ページで、**[az800l04-vm0]** を選択します。
1. **[az800l04-vm0]** ページの **[操作]** セクションで、**[コマンドの実行]** を選択し、**[RunPowerShellScript]** を選択します。
1. Windows リモート管理が無効になっている場合は、**[コマンド スクリプトの実行]** ページの **[PowerShell スクリプト]** セクションで、次のコマンドを入力し、**[実行]** を選択して有効にします。

   ```powershell
   winrm quickconfig -quiet
   ```

1. **[PowerShell スクリプト]** セクションで、前の手順で入力したテキストを次のコマンドに置き換え、**[実行]** を選択して、Windows リモート管理受信ポートを開きます。

   ```powershell
   Set-NetFirewallRule -Name WINRM-HTTP-In-TCP-PUBLIC -RemoteAddress Any
   ```

1. **[PowerShell スクリプト]** セクションで、前の手順で入力したテキストを次のコマンドに置き換え、**[実行]** を選択して、PowerShell リモート処理を有効にします。

   ```powershell
   Enable-PSRemoting -Force -SkipNetworkProfileCheck
   ```

#### タスク 3: Azure VM で実行されている Windows Admin Center ゲートウェイを使用して、Azure VM に接続する

1. **SEA-ADM1** で、**az800l04-vmwac** Azure VM 上で実行されている Windows Admin Center ゲートウェイのインターフェイスが表示されている Microsoft Edge ウィンドウで、**[Windows Admin Center]** を選択します。
1. **[すべての接続]** ページで、**[+ 追加]** を選択します。
1. **[リソースの追加または作成]** ページの **[サーバー]** セクションで、**[追加]** を選択します。
1. **[サーバー名]** テキスト ボックスに 「**az800l04-vm0**」と入力します。
1. **[この接続に別のアカウントを使用する]** オプションを選択し、講師から提供された資格情報を入力し、**[資格情報を使用して追加する]** を選択します。
1. 接続の一覧で、**[az800l04-vm0]** を選択します
1. Azure VM に正常に接続されたら、Windows Admin Center の **az800l04-vmwac** Azure VM の [概要] ウィンドウを確認します。

## 演習 5: Azure 環境のプロビジョニング解除

#### タスク 1: Cloud Shell で PowerShell セッションを開始する

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。
1. Azure portal が表示されている Microsoft Edge ウィンドウで、[Cloud Shell] アイコンを選択して [Cloud Shell] ウィンドウを開きます。

#### タスク 2: ラボでプロビジョニングしたすべての Azure リソースを特定する

1. [Cloud Shell] ウィンドウで次のコマンドを実行して、このラボで作成されたすべてのリソース グループの一覧を表示します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L040*'
   ```

1. 次のコマンドを実行して、このラボで作成したすべてのリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L040*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**注**: このコマンドは非同期で実行されるため (-AsJob パラメーターによって決定されます)、同じ PowerShell セッション内で、直後に別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。
