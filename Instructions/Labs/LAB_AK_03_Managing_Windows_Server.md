---
lab:
  title: 'ラボ: Windows Server の管理'
  type: Answer Key
  module: 'Module 3: Windows Server administration'
---

# <a name="lab-answer-key-managing-windows-server"></a>ラボの回答キー: Windows Server の管理

## <a name="exercise-1-implementing-and-using-remote-server-administration"></a>演習 1: リモート サーバー管理の実装と使用

#### <a name="task-1-install-windows-admin-center"></a>タスク 1: Windows Admin Center をインストールする

1. **SEA-ADM1** に接続し、必要に応じて、パスワード **Pa55w.rd** を使用し、**Contoso\\Administrator** としてサインインします。
1. **SEA-ADM1** 上で **[スタート]** を選択し、**[Windows PowerShell (管理者)]** を選択します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから Enter キーを押して、Windows Admin Center の最新バージョンをダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait until the installation completes. This should take about 2 minutes.
   
   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: After installation completes, you may encounter the error message 'ERR_Connection_Refused'. If this occurs, restart SEA-ADM1 to correct the issue.

#### <a name="task-2-add-servers-for-remote-administration"></a>タスク 2: リモート管理用のサーバーを追加する

1. **SEA-ADM1** で Microsoft Edge を開始して、`https://SEA-ADM1.contoso.com` にアクセスします。 
1. ダイアログが表示されたら、**[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力して、**[OK]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. **[このリリースの新機能]** ポップアップ ウィンドウを確認し、右上隅の **[閉じる]** を選択します。
1. **[すべての接続]** ページを確認します。**sea-adm1.contoso.com** エントリが含まれていることに注目します。 
1. **[すべての接続]** ページで、**[+ 追加]** を選択します。 
1. リソースの追加または作成ペインの **[サーバー]** タイルで、 **[追加]** を選択します。
1. **[サーバー名]** テキスト ボックスに、「**sea-dc1.contoso.com**」と入力します。
1. **[Use another account for this connection]\(この接続に別のアカウントを使用する\)** オプションが選択されていることを確かめ、次の資格情報を入力してから、 **[Add with credentials]\(資格情報を使用して追加\)** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: After performing step 7, if an error message that says <bpt id="p2">**</bpt>You can add this server to your list of connections, but we can't confirm it's available.<ept id="p2">**</ept> appears, select <bpt id="p1">**</bpt>Add<ept id="p1">**</ept>. In the All Connections pane,  select <bpt id="p1">**</bpt>sea-svr1.contoso.com<ept id="p1">**</ept>, and then select <bpt id="p2">**</bpt>Manage as<ept id="p2">**</ept>. In the <bpt id="p1">**</bpt>Specify your credentials<ept id="p1">**</ept> dialog box, ensure that the <bpt id="p2">**</bpt>Use another account for this connection<ept id="p2">**</ept> option is selected, enter the Administrator credentials, and then select <bpt id="p3">**</bpt>Continue<ept id="p3">**</ept>.

   > **注**: シングル サインオンを実行するには、Kerberos の制約付き委任を設定する必要があります。

#### <a name="task-3-configure-windows-admin-center-extensions"></a>タスク 3: Windows Admin Center 拡張機能を構成する

1. **SEA-ADM1** 上の、Windows Admin Center を表示している Microsoft Edge ウィンドウの右上隅で、**[設定]** アイコン (歯車ホイール) を選択します。
1. In the left pane, select <bpt id="p1">**</bpt>Extensions<ept id="p1">**</ept>. Review the available extensions.
1. Select the <bpt id="p1">**</bpt>Security (Preview)<ept id="p1">**</ept> extension, and then select <bpt id="p2">**</bpt>Install<ept id="p2">**</ept>. The extension will install and Windows Admin Center will refresh.

   > **注**: **セキュリティ (プレビュー)** 拡張機能を使用できない場合は、別の Microsoft 拡張機能を選択します。

1. 詳細ウィンドウで、**[インストール済みの拡張機能]** を選択し、一覧に DNS (プレビュー) 拡張機能が含まれていることを確認します。
1. 上部メニューの **[設定]** の横にあるドロップダウン矢印を選択してから、**[サーバー マネージャー]** を選択します。
1. **[サーバーの接続]** ページで、**[sea-dc1.contoso.com]** リンクを選択します。
1. **[この接続に別のアカウントを使用する]** オプションを確実に選択し、**[すべての接続にこれらの資格情報を使用する]** を選択し、次の資格情報を入力してから、**[続行]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. To install the DNS PowerShell tools, in the left pane, in the list of <bpt id="p1">**</bpt>Tools<ept id="p1">**</ept>, select <bpt id="p2">**</bpt>DNS<ept id="p2">**</ept>, and then select <bpt id="p3">**</bpt>Install<ept id="p3">**</ept>. The tools will take less than a minute to install.
1. **Contoso.com** ゾーンを選択し、その DNS レコードの一覧を確認します。

#### <a name="task-4-verify-remote-administration"></a>タスク 4: リモート管理を確認する

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, in Windows Admin Center, in the left pane, in the list of <bpt id="p2">**</bpt>Tools<ept id="p2">**</ept>, select <bpt id="p3">**</bpt>Overview<ept id="p3">**</ept>. Note that the details pane of Windows Admin Center displays basic server information and performance monitoring.
1. In the left pane, in the list of <bpt id="p1">**</bpt>Tools<ept id="p1">**</ept>, scroll down and review the basic administration tools available. Select <bpt id="p1">**</bpt>Roles &amp; features<ept id="p1">**</ept> and note which roles and features are listed as installed and which ones are available to install. Scroll down, select the <bpt id="p1">**</bpt>Telnet Client<ept id="p1">**</ept> checkbox, and then select <bpt id="p2">**</bpt>+ Install<ept id="p2">**</ept> at the top of the pane.
1. **[Install Roles and Features]\(役割と機能のインストール\)** ペインで **[はい]** を選び、Telnet クライアントが正常にインストールされたことを確認するメッセージが表示されるのを待ちます。
1. 左側のウィンドウの一番下にある、 **[ツール]** の一覧の下で、 **[設定]** を選択します。
1. 右側の **[設定]** セクションで、**[リモート デスクトップ]** を選択します。
1. **[リモート デスクトップ]** セクションで、オプションの **[このコンピューターへのリモート接続を許可する]** チェック ボックスをオンにして、**[保存]** を選択します。
1. 左側のウィンドウにある **[ツール]** の一覧で、 **[リモート デスクトップ]** を選択します。
1. [リモート デスクトップ] ウィンドウで、**[このマシンによって提示された証明書に自動的に接続する]** チェックボックスをオンにし、**[接続]** を選択します。
1. メッセージが表示されたら、 **[確認]** を選択してから、 **[接続]** を選択します。
1. Windows Admin Center インターフェイス内で **SEA-DC1** にリモート デスクトップを介して正常に接続したことを確認します。
1. **[切断]** を選択します。
1. Microsoft Edge ウィンドウを閉じます。

#### <a name="task-5-administer-servers-with-remote-powershell"></a>タスク 5: リモート PowerShell を使用してサーバーを管理する

1. **SEA-ADM1** 上で、**PowerShell** コンソール セッションに切り替えます。 
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して、**SEA-DC1** への PowerShell リモート処理セッションを開始します。

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```
1. **[SEA-DC1]** プロンプトで、次のコマンドを入力し、Enter キーを押して、アプリケーション ID サービス (AppIDSvc) の状態を表示します。

   ```powershell
   Get-Service -Name AppIDSvc
   ```

   > **注**: サービスが現在停止中であることを確認します。

1. **[SEA-DC1]** プロンプトで、次のコマンドを入力し、Enter キーを押すことで、アプリケーション ID サービスを開始します。

   ```powershell
   Start-Service -Name AppIDSvc
   ```
1. **[SEA-DC1]** プロンプトで、次のコマンドを入力し、Enter キーを押して、アプリケーション ID サービス (AppIDSvc) の状態を表示します。

   ```powershell
   Get-Service -Name AppIDSvc
   ```

   > **注**: サービスが現在実行中であることを確認します。

### <a name="results"></a>結果

After completing this exercise, you will have installed Windows Admin Center and connected it to the servers in your lab environment. You performed a number of remote management tasks including installing a feature as well as enabling and testing Remote Desktop connectivity. Finally, you used PowerShell Remoting to check the status of a service and then to start it.
