---
lab:
  title: 'ラボ: Windows Server でのネットワーク インフラストラクチャ サービスの実装と構成'
  type: Answer Key
  module: 'Module 7: Network Infrastructure services in Windows Server'
---

# ラボの回答キー : Windows Server でのネットワーク インフラストラクチャ サービスの実装と構成

## 演習 1: DHCP の展開と構成

#### タスク 1: DHCP 役割をインストールする

1. **SEA-ADM1** に接続してから、必要に応じて、講師から提供された資格情報でサインインします。
1. **SEA-ADM1** 上で **[スタート]** を選択し、**[Windows PowerShell (管理者)]** を選択します。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。

1. **[Windows PowerShell]** コンソールで、次のコマンドを入力し、Enter キーを押して、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` にアクセスします。
 
   >**注**: リンクが機能しない場合は、**SEA-ADM1** でエクスプローラー開き、ダウンロード フォルダーを選択し、ダウンロード フォルダー内の **WindowsAdminCenter.msi** ファイルを選択し、手動でインストールします。 インストールが完了したら、Microsoft Edge を更新します。

   >**注**: **NET::ERR_CERT_DATE_INVALID** エラーが発生した場合は、Edge ブラウザー ページの **[詳細設定]** を選択し、ページの下部にある **[sea-adm1-contoso.com (アンセーフ) に移動]** を選択します。

1. メッセージが表示されたら、**[Windows セキュリティ]** ダイアログ ボックスに講師から提供された資格情報を入力してから、**[OK]** を選択します。

1. [すべての接続] ペインで、**[+ 追加]** を選択します。
1. リソースの追加または作成ペインの **[サーバー]** タイルで、 **[追加]** を選択します。
1. **[サーバー名]** テキスト ボックスに、「**sea-svr1.contoso.com**」と入力します。 
1. **[この接続に別のアカウントを使用する]** オプションが選択されていることを確認し、講師から提供された資格情報を入力してから、**[資格情報を使用して追加する]** を選択します。

   > **注**: シングル サインオンを実行するには、Kerberos の制約付き委任を設定する必要があります。

1. **[sea-svr1.contoso.com]** ページの **[ツール]** リストで、**[役割と機能]** を選択します。
1. [役割と機能] ペインで、**[DHCP サーバー]** チェックボックスをオンにし、**[+ インストール]** を選択します。
1. [Install Roles and Features](役割と機能のインストール) ペインで、**[はい]** を選択します。

   > **注**: DHCP 役割がインストールされていることを示す通知が行われるまで待ちます。 必要に応じて、**[通知]** アイコンを選択して現在の状態を確認します。

1. **Microsoft Edge** ページを更新して **[sea-svr1.contoso.com]** ページに戻り、**[ツール]** リストで **[DHCP]** を選んでから、詳細ペインで **[インストール]** を選択して DHCP PowerShell ツールをインストールします。 

   > **注**: **sea-svr1.contoso.com** の **[ツール]** リストで **DHCP** エントリを利用できない場合は、**Microsoft Edge** ページを更新して、もう一度やり直してください。
   ネットワーク パフォーマンスによっては、DHCP サーバーが表示されるまでに最大 5 分かかる場合があります。

1. DHCP PowerShell ツールがインストールされていることを知らせる通知を待ちます。 必要に応じて、**[通知]** アイコンを選択して現在の状態を確認します。

#### タスク 2: DHCP サーバーを承認する

1. **SEA-ADM1** で、**[スタート]** を選択してから、**[サーバー マネージャー]** を選びます。
1. **[サーバー マネージャー]** で、メニューの **[通知]** を選択してから、**[DHCP 構成を完了する]** を選びます。
1. **[DHCP インストール後の構成ウィザード]** ウィンドウの **[説明]** 画面で、**[次へ]** を選択します。
1. **[承認]** 画面で、**CONTOSO\Administrator** オプションが選択されていることを確実にしてから **[コミット]** を選択します。
1. 両方のタスクが完了したら、**[閉じる]** を選択します。

#### タスク 3: スコープを作成する

1. **SEA-ADM1** で、**SEA-SVR1** の **[DHCP]** 設定が表示されている Microsoft Edge ウィンドウの Windows Admin Center に切り替えます。

   > **注**:メニューに DHCP オプションが表示されるまでに数分かかる場合があります。 必要に応じて、sea-svr1 への接続を更新します。 DHCP PowerShell ツールのインストールを求められたら、**[インストール]** を選択します。

1. **[DHCP]** ページで、 **[+ 新しいスコープ]** を選択します。
1. 新しいスコープの作成ペインで、次の設定を指定してから、**[作成]** を選択します。

   - プロトコル: **IPv4**
   - 名前: **ContosoClients**
   - 開始 IP アドレス: **10.100.150.50**
   - 終了 IP アドレス: **10.100.150.254**
   - DHCP クライアント サブネット マスク: **255.255.255.0**
   - ルーター (デフォルト ゲートウェイ): **10.100.150.1**
   - DHCP クライアントのリース期間: **4 日**

1. **SEA-ADM1** で、**[サーバー マネージャー]** に切り替え、**[サーバー マネージャー]** で **[ツール]** を選択してから、**[DHCP]** を選びます。
1. **[DHCP]** ウィンドウの [操作] ペインで、**[その他の操作]** を選択してから、**[承認されたサーバーの管理]** を選びます。
1. **[承認されたサーバーの管理]** ウィンドウで **[更新]** を選択し、承認された DHCP サーバーのリストに **sea-svr1.contoso.com** が表示されることを確かめてから、**[承認されたサーバーの管理]** ウィンドウを閉じます。
1. **[DHCP]** ウィンドウの [操作] ペインで、**[その他の操作]** を選択してから、**[サーバーの追加]** を選びます。
1. **[サーバーの追加]** ダイアログ ボックスで、**[この承認された DHCP サーバー]** を選択し、**[sea-svr1.contoso.com]** を選んでから **[OK]** を選択します。
1. **[DHCP]** ウィンドウで、**sea-svr1**、 **[IPv4]** 、 **[スコープ [10.100.150.0] ContosoClients]** の順に展開し、 **[スコープ オプション]** を選択します。
1. [操作] ペインで **[その他のアクション]** を選択してから、**[オプションの構成]** を選択します。
1. **[スコープ オプション]** ダイアログ ボックスで、**[006 DNS サーバー]** チェックボックスをオンにします。
1. **[サーバー名]** テキスト ボックスに「**sea-dc1.contoso.com**」と入力し、**[解決]** を選択して、名前が **172.16.10.10** に解決されていることを確認し、**[追加]**、**[OK]** の順に選択します。

#### タスク 4: DHCP フェールオーバーを構成する

1. **SEA-ADM1** の **[DHCP]** ウィンドウで、**[IPv4]** を選び、[操作] ペインで **[その他の操作]** を選択してから、**[フェールオーバーの構成]** を選びます。
1. **[フェールオーバーの構成]** ウィンドウで、**[すべて選択]** チェックボックスがオンになっていることを確認してから **[次へ]** を選択します。
1. **[フェールオーバーに使用するパートナー サーバーを指定します]** 画面で、**[サーバーの追加]** を選択します。
1. **[サーバーの追加]** ダイアログ ボックスで、**[この承認された DHCP サーバー]** を選択し、**[sea-dc1.contoso.com]** を選んでから **[OK]** を選択します。
1. **[フェールオーバーに使用するパートナー サーバーの指定]** 画面に戻り、**[パートナー サーバー]** ドロップダウン リストに **[sea-dc1]** が表示されていることを確かめてから、**[次へ]** を選択します。
1. **[新しいフェールオーバー リレーションシップの作成]** 画面で、次の情報を入力してから **[次へ]** を選択します。

   - リレーションシップ名: **SEA-SVR1 から SEA-DC1**
   - クライアントの最大リード タイム: **1 時間**
   - モード: **ホット スタンバイ**
   - パートナー サーバーの役割: **スタンバイ**
   - スタンバイ サーバー用に予約されているアドレス: **5%**
   - 状態の切り替え間隔: **無効**
   - メッセージ認証を有効にする: **有効**
   - 共有シークレット: **DHCP フェールオーバー**

1. **[完了]** を選択します。
1. **[フェールオーバーの構成]** ダイアログ ボックスで、**[閉じる]** を選択します。
1. **[DHCP]** ウィンドウの [操作] ペインで、**[その他の操作]** を選択してから、**[サーバーの追加]** を選びます。
1. **[サーバーの追加]** ダイアログ ボックスで、**[この承認された DHCP サーバー]** を選択し、**[sea-dc1.contoso.com]** を選んでから **[OK]** を選択します。
1. **SEA-ADM1** の **[DHCP]** ウィンドウで、**[sea-dc1]** ノードを展開し、**[IPv4]** を選択してから、2 つのスコープが一覧表示されていることを確認します。
1. **[スコープ [172.16.0.0] Contoso]** を選び、[操作] ペインで **[その他のアクション]** を選択してから、**[フェールオーバーの構成]** を選びます。
1. **[フェールオーバーの構成]** ウィンドウで、**[次へ]** を選択します。
1. **[フェールオーバーに使用するパートナー サーバーの指定]** 画面で、**[パートナー サーバー]** ボックスに「**172.16.10.12**」と入力し、**[このサーバーで構成された既存のフェールオーバー リレーションシップを再利用する (存在する場合)]** チェックボックスをオンにしてから、**[次へ]** を選択します。
1. **[このサーバーで既に構成されているフェールオーバー リレーションシップから選択]** 画面で、**[次へ]** を選択してから、**[完了]** を選択します。
1. **[フェールオーバーの構成]** ダイアログ ボックスで、**[閉じる]** を選択します。
1. **sea-svr1 ** で **[IPv4]** を選択してから、両方のスコープが一覧表示されていることを確認します。 必要に応じて、**F5** キーを押して更新します。

#### タスク 5: DHCP 機能を確認する

1. **SEA-ADM1** で、**[スタート]** を選択してから、**[設定]** を選択します。
1. **[設定]** ウィンドウで **[ネットワークとインターネット]** を選択し、**[ネットワークと共有センター]** を選びます。
1. **[ネットワークと共有センター]** で、**[イーサネット]** を選んでから、**[プロパティ]** を選択します。
1. **[イーサネットのプロパティ]** ダイアログ ボックスで、**[インターネット プロトコル バージョン 4 (TCP/IPv4)]** を選んでから、**[プロパティ]** を選択します。
1. **[インターネット プロトコル バージョン 4 (TCP/IPv4) のプロパティ]** ダイアログ ボックスで、**[IP アドレスを自動的に取得する]** を選択し、**[DNS サーバーアドレスを自動的に取得する]** を選んでから、**[OK]** を選択します。
1. **[閉じる]** を選択してから、 **[イーサネットの状態]** ウィンドウで **[詳細]** を選択します。
1. **[ネットワーク接続の詳細]** ダイアログ ボックスで、DHCP が有効になっていること、IP アドレスが取得されたこと、**sea-svr1 ** DHCP サーバーによってリースが発行されたことを確認します。
1. **[閉じる]** を選択して、 **[イーサネットの状態]** ウィンドウに戻ります。
1. **SEA-ADM1** の **[DHCP]** ウィンドウで、**[172.16.10.12]** ノードを展開し、**[IPv4]** ノードを展開します。次に、**[スコープ [172.16.0.0] Contoso]** ノードを展開し、**[アドレス リース]** を選択します。
1. **SEA-ADM1.contoso.com** リースを表すエントリがあることを確認します。
1. **SEA-ADM1** の **[DHCP]** ウィンドウで、**[sea-dc1]** ノードを展開し、**[IPv4]** ノードを展開します。次に、**[スコープ [172.16.0.0] Contoso]** ノードを展開し、**[アドレス リース]** を選択します。
1. ここでも、**SEA-ADM1.contoso.com** リースを表すエントリがあることを確認します。
1. **sea-svr1 ** を選び、[操作] ペインで **[その他の操作]** を選択し、 **[すべてのタスク]** を選んでから **[停止]** を選択します。
1. **SEA-ADM1** で、**[イーサネットの状態]** ウィンドウに戻り、**[無効]** を選択します。
1. **[ネットワークと共有センター]** ウィンドウに戻り、**[アダプターの設定の変更]** を選択し、**[イーサネット]** を選んでから **[このネットワーク デバイスを有効にする]** を選択します。
1. 有効な**イーサネット**接続をダブルクリックして、その状態ウィンドウを表示します。
1. **[イーサネットの状態]** ウィンドウで、**[詳細]** を選択します。
1. **[ネットワーク接続の詳細]** ダイアログ ボックスで、DHCP が有効になっていること、IP アドレスが取得されたこと、および **SEA-DC1 (172.16.10.10)** DHCP サーバーによってリースが発行されたことを確認します。
1. **[閉じる]** を選択して、 **[イーサネットの状態]** ウィンドウに戻ります。
1. **[イーサネットの状態]** ウィンドウで、**[プロパティ]** を選択します。
1. **[イーサネットのプロパティ]** ダイアログ ボックスで、**[インターネット プロトコル バージョン 4 (TCP/IPv4)]** を選んでから、**[プロパティ]** を選択します。
1. **[インターネット プロトコル バージョン 4 (TCP/IPv4) のプロパティ]** ダイアログ ボックスで、**[次の IP アドレスを使用する]** を選択し、次の設定を指定します。

   - IP アドレス: **172.16.10.11**
   - サブネット マスク: **255.255.0.0**
   - デフォルト ゲートウェイ: **172.16.10.1**

1. **[インターネット プロトコル バージョン 4 (TCP/IPv4) のプロパティ]** ダイアログ ボックスで、**[次の DNS サーバーのアドレスを使用する]** を選択し、**[優先 DNS サーバー]** を **172.16.10.10** に設定してから **[OK]** を選択します。

   > **注**: **[イーサネットの状態]** ウィンドウは開いたままにしておきます。 このラボで後ほど必要になります。 

## 演習 2: DNS の展開と構成

#### タスク 1: DNS 役割をインストールする

1. **SEA-ADM1** で、Windows Admin Center の **sea-svr1.contoso.com** への接続が表示されている Microsoft Edge ウィンドウに戻ります。 
1. **[ツール]** リストで、**[役割と機能]** を選択します。
1. [役割と機能] ペインで、**[DNS サーバー]** チェックボックスをオンにし、**[+ インストール]** を選択します。
1. [Install Roles and Features](役割と機能のインストール) ペインで、**[はい]** を選択します。

   > **注**: DNS 役割がインストールされていることを示す通知が行われるまで待ちます。 必要に応じて、**[通知]** アイコンを選択して現在の状態を確認します。

1. **[Microsoft Edge]** ページを更新し、**[sea-svr1.contoso.com]** ページに戻り、**[ツール]** リストで **[DNS]** を選んでから、詳細ペインで **[インストール]** を選択して DNS PowerShell ツールをインストールします。 

   > **注**: **sea-svr1.contoso.com** の **[ツール]** リストで **DNS** エントリを利用できない場合は、**Microsoft Edge** ページを更新して、もう一度試してください。

1. DNS PowerShell ツールがインストールされていることを示す通知が表示されるまで待ちます。 必要に応じて、**[通知]** アイコンを選択して現在の状態を確認します。

#### タスク 2: DNS ゾーンを作成する

1. **SEA-ADM1** の Windows Admin Center で、[DNS] ペインの **[操作]** を選択し、**[操作]** メニューの **[+ 新しい DNS ゾーンの作成]** を選択します。
1. **[新しい DNS ゾーンの作成]** ダイアログ ボックスで、以下の設定を指定してから、**[作成]** を選択します。

   - ゾーンの種類: **プライマリ**
   - ゾーン名: **TreyResearch.net**
   - ゾーン ファイル: **新しいファイルを作成**
   - ゾーン ファイル名: **TreyResearch.net.dns**
   - 動的更新: **動的更新を許可しない**

1. [DNS] ペインに戻り、 **[TreyResearch.net]** を選んでから、 **[+ 新しい DNS レコードの作成]** を選択します。
1. **[新しい DNS レコードの作成]** ペインで以下の設定を指定してから、 **[作成]** を選択します。

   - DNS レコードの種類: **ホスト (A)**
   - レコード名: **TestApp**
   - IP アドレス: **172.30.99.234**
   - Time to Live: **600**

1. **SEA-ADM1** で **[スタート]** を選択してから、**[Windows PowerShell]** を選びます。
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから Enter キーを押して、新しい DNS レコードで名前解決が提供されていることを確認します。

   ```powershell
   Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
   ```

#### タスク 3: 転送を構成する

1. **SEA-ADM1** で、サーバー マネージャーに切り替えます。
1. サーバー マネージャーで、**[ツール]** を選択してから、**[DNS]** を選びます。
1. **[DNS マネージャー]** で、**[DNS]** を選択し、その状況依存のメニューを表示してから、メニューで **[DNS サーバーに接続]** を選択します。
1. **[DNS サーバーに接続]** ダイアログ ボックスで、**[次のコンピューター]** を選択し、「**SEA-SVR1.contoso.com**」と入力してから、**[OK]** を選択します。
1. **[DNS マネージャー]** で、**[SEA-SVR1.contoso.com]** を選び、その状況依存のメニューを表示してから、**[プロパティ]** を選択します。
1. **[SEA-SVR1.contoso.com のプロパティ]** ダイアログ ボックスで、**[フォワーダー]** タブを選択してから、**[編集]** を選択します。
1. **[フォワーダーの編集]** ダイアログ ボックスの **[転送サーバーの IP アドレス]** ボックスに「**131.107.0.100**」と入力ししてから、**[OK]** を選択します。
1. **[SEA-SVR1.contoso.com のプロパティ]** ダイアログ ボックスで、**[OK]** を選択します。

#### タスク 4: 条件付き転送を構成する

1. **SEA-ADM1** の **[DNS マネージャー]** で、**[SEA-SVR1.contoso.com]** を展開し、**[条件付きフォワーダー]** を選択します。
1. **[条件付きフォワーダー]** を選択し、その状況依存のメニューを表示してから、メニューで **[新しい条件付きフォワーダー]** を選択します。
1. **[新しい条件付きフォワーダー]** ダイアログ ボックスの **[DNS ドメイン]** ボックスに、「**Contoso.com**」と入力します。
1. **[マスター サーバーの IP アドレス]** ボックスに「**172.16.10.10**」と入力してから、**Enter** キーを押します。

   > **注**: **[新しい条件付きフォワーダー]** ダイアログ ボックス内の検証列の**不明なエラーが発生しました**というメッセージは無視してください。

1. **[OK]** を選択します。
1. **SEA-ADM1** で、**Windows PowerShell** コンソールに切り替えます。
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して条件付き転送を確認します。

   ```powershell
   Resolve-DnsName -Server sea-svr1.contoso.com -Name sea-dc1.contoso.com
   ```

#### タスク 5: DNS ポリシーを構成する

1. **SEA-ADM1** で、Windows Admin Center の **sea-svr1.contoso.com** への接続が表示されている Microsoft Edge ウィンドウに戻ります。
1. **ツール**一覧で、**PowerShell** を選択し、メッセージが表示されたら、講師から提供された資格情報でサインインします。
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して本社のサブネットを作成します。

   ```powershell
   Add-DnsServerClientSubnet -Name "HeadOfficeSubnet" -IPv4Subnet '172.16.10.0/24'
   ```
1. 次のコマンドを入力してから、Enter キーを押して本社のゾーン スコープを作成します。

   ```powershell
   Add-DnsServerZoneScope -ZoneName 'TreyResearch.net' -Name 'HeadOfficeScope'
   ```
1. 次のコマンドを入力してから、Enter キーを押して本社のスコープの新しいリソース レコードを追加します。

   ```powershell
   Add-DnsServerResourceRecord -ZoneName 'TreyResearch.net' -A -Name 'testapp' -IPv4Address '172.30.99.100' -ZoneScope 'HeadOfficeScope'
   ```
1. 次のコマンドを入力してから、Enter キーを押して、本社のサブネットとゾーンのスコープをリンクする新しいポリシーを作成します。

   ```powershell
   Add-DnsServerQueryResolutionPolicy -Name 'HeadOfficePolicy' -Action ALLOW -ClientSubnet 'eq,HeadOfficeSubnet' -ZoneScope 'HeadOfficeScope,1' -ZoneName 'TreyResearch.net'
   ```

#### タスク 6: DNS ポリシー機能を確認する

1. **SEA-ADM1** で、**Windows PowerShell** コンソールに切り替えます。
1. **Windows PowerShell**コンソールで、「`ipconfig`」と入力してから、Enter キーを押して現在の IP 構成を表示します。

   > **注**: イーサネット アダプターの IP アドレスは、ポリシーで構成されている **HeadOfficeSubnet** の一部であることに注意してください。

1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して `testapp.treyresearch.net` DNS レコードの解決をテストします。

   ```powershell
   Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
   ```

   > **注**: 名前が、**HeadOfficePolicy** で構成された IP アドレス **172.30.99.100** に解決されることを確認します。

1. **SEA-ADM1** で、**[イーサネットの状態]** ウィンドウに戻ります。
1. **[イーサネットの状態]** ウィンドウで、**[プロパティ]** を選択します。
1. **[イーサネットのプロパティ]** ダイアログ ボックスで、**[インターネット プロトコル バージョン 4 (TCP/IPv4)]** を選んでから、**[プロパティ]** を選択します。
1. **[インターネット プロトコル バージョン 4 (TCP/IPv4) のプロパティ]** ダイアログ ボックスで、現在割り当てられている IP アドレス (**172.16.10.11**) を、**HeadOfficeSubnet** の IP アドレスの範囲外の IP アドレス **172.16.11.11** に変更してから **[OK]** を選択します。
1. **SEA-ADM1** で、**Windows PowerShell** コンソールに切り替えます。
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して `testapp.treyresearch.net` DNS レコードの解決をテストします。

   ```powershell
   Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
   ```

   > **注**: 名前が **172.30.99.234** に解決されることを確認します。 **SEA-ADM1** の IP アドレスが **HeadOfficeSubnet** 内に存在しなくなったため、これは想定内です。 `testapp.treyresearch.net` をターゲットとする **(172.16.10.0/24)** の **HeadOfficeSubnet** からの DNS クエリは、**172.30.99.100** に解決されます。 `testapp.treyresearch.net` をターゲットとするこのサブネット外からの DNS クエリは、**172.30.99.234** に解決されます。

   > **注**: ここで、**SEA-ADM1** の IP アドレスをその元の値に戻します。

1. **[イーサネットの状態]** ウィンドウに戻り、**[プロパティ]** を選択します。
1. **[イーサネットのプロパティ]** ダイアログ ボックスで、**[インターネット プロトコル バージョン 4 (TCP/IPv4)]** を選んでから、**[プロパティ]** を選択します。
1. **[インターネット プロトコル バージョン 4 (TCP/IPv4) のプロパティ]** ダイアログ ボックスで、現在割り当てられている IP アドレス (**172.16.11.11**) をその元の値 (**172.16.10.11**) に変更し、**[OK]** を選択します。
1. **[閉じる]** を 2 回選択します。
1. 開いているすべてのウィンドウを閉じます。
