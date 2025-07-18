---
lab:
  title: 'ラボ: ID サービスおよびグループ ポリシーの実装'
  type: Answer Key
  module: 'Module 1: Identity services in Windows Server'
---

# ラボの回答キー: ID サービスおよびグループ ポリシーの実装

>**注**: 以前提供されていたラボ シミュレーションは廃止されました。

## 演習 1: Server Core への新しいドメイン コントローラーの展開

#### タスク 1: 新しい Windows Server Core サーバーに AD DS を展開する

1. **SEA-ADM1** に接続し、必要に応じて、講師から提供された資格情報でサインインします。
1. **SEA-ADM1** で **[スタート]** を選択してから、**[Windows PowerShell (管理者)]** を選びます。
1. AD DS サーバーの役割をインストールするには、Windows PowerShell コマンド プロンプトで次のコマンドを入力してから Enter キーを押します。
    
   ```powershell
   Install-WindowsFeature –Name AD-Domain-Services –ComputerName SEA-SVR1
   ```
1. **SEA-SVR1** に AD DS 役割がインストールされていることを確認するには、以下のコマンドを入力してから Enter キーを押します。
    
   ```powershell
   Get-WindowsFeature –ComputerName SEA-SVR1
   ```
1. 前のコマンドの出力で、**Active Directory Domain Services** チェックボックスを探し、それが選択されていることを確認します。 その後、**リモート サーバー管理ツール**を探します。 その下の **[役割管理ツール]** ノードに注目してください。その後、**[AD DS および AD LDS ツール]** ノードも選択されていることを確認します。

   > **注**: **[AD DS および AD LDS ツール]** ノードでは、**Windows PowerShell 用 Active Directory モジュール**のみがインストールされ、Active Directory 管理センターなどのグラフィカル ツールはインストールされていません。 サーバーを中央管理する場合、通常は各サーバーにこれらは必要ありません。 それらをインストールする場合は、**RSAT-ADDS** コマンドを使用して **Add-WindowsFeature** コマンドレットを実行し、AD DS ツールを指定する必要があります。

   > **注**: インストール プロセスが完了してから、AD DS 役割がインストールされていることを確認する前に、しばらく待つ必要がある場合があります。 **Get-WindowsFeature** コマンドから予想される結果が得られない場合は、数分後に再度試すことができます。

#### タスク 2: AD DS インストールの準備をして、リモート サーバーを昇格させる

1.  **SEA-ADM1** の **[スタート]** メニューで **[サーバー マネージャー]** を選んでから、**[サーバー マネージャー]** で **[すべてのサーバー]** ビューを選択します。
1. **[管理]** メニューで、**[サーバーの追加]** を選択します。
1. **[サーバーの追加]** ダイアログ ボックスで、既定の設定を維持し、**[今すぐ検索]** を選択します。
1. サーバーの **[Active Directory]** リストで **[SEA-SVR1]** を選び、矢印を選択して **[選択済み]** リストに追加してから、**[OK]** を選択します。
1. **SEA-ADM1** で、**SEA-SRV1** への AD DS 役割のインストールが完了していること、およびサーバーが**サーバー マネージャー**に追加されたことを確かめます。 その後、**[通知]** フラグ シンボルを選択します。
1. **SEA-SVR1** の展開後の構成に注意してください。その後、**[このサーバーをドメイン コントローラーに昇格する]** リンクを選択します。
1. **[Active Directory Domain Services 構成ウィザード]** の **[展開構成]** ページの **[展開操作の選択]** の下で、**[既存のドメインにドメイン コントローラーを追加する]** が選択されていることを確認します。
1. `Contoso.com` ドメインが指定されていることを確かめてから、**[この操作を実行する資格情報を指定する]** セクションで **[変更]** を選択します。
1. **[展開操作の資格情報]** ダイアログ ボックスに講師から提供された資格情報を挿入します。
1. **[OK]** を選択し、 **[次へ]** を選択します。
1. **[ドメイン コントローラーのオプション]** ページで、**[ドメイン ネーム システム (DNS) サーバー]** と **[グローバル カタログ (GC)]** チェックボックスがオンになっていることを確かめます。 **[読み取り専用ドメイン コントローラー (RODC)]** チェック ボックスがオフになっていることを確かめます。
1. **[ディレクトリ サービス復元モード (DSRM) パスワードの入力]** セクションで、講師から提供されたパスワードを入力して確認してから、**[次へ]** を選択します。
1. **[DNS オプション]** ページで、**[次へ]** を選択します。
1. **[追加オプション]** ページで、**[次へ]** を選択します。
1. **[パス]** ページで、**データベース** フォルダー、**ログ ファイル** フォルダー、および **SYSVOL** フォルダーの既定のパス設定を保持し、**[次へ]** を選択します。
1. 生成された Windows PowerShell スクリプトを開くには、**[オプションの確認]** ページで **[スクリプトの表示]** を選択します。
1. メモ帳で、生成された Windows PowerShell スクリプトを編集します。

   - 番号記号 (**#**) で始まるコメント行を削除します。
   - **Import-Module** 行を削除します。
   - 各行の末尾にあるアクサン グラーブ (**`**) を削除します。
   - 改行を削除します。

1. これで、**Install-ADDSDomainController** コマンドとすべてのパラメーターが 1 行に収まりました。 その行の前にカーソルを置き、**[編集]** メニューで、**[すべて選択]** を選んで行全体を選択します。 メニューの **[編集]** を選んでから、**[コピー]** を選択します。

1. 確認を求めるメッセージが表示されたら、**[はい]** を選択してウィザードをキャンセルします。
1. Windows PowerShell のコマンド プロンプトで、次のコマンドを入力します。

   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 { }
   ```
1. 中かっこ (**{ }**) の間にカーソルを置き、クリップボードからコピーされたスクリプト行の内容を貼り付けます。 完全なコマンドは次の形式になるはずです。
    
   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 {Install-ADDSDomainController -NoGlobalCatalog:$false -CreateDnsDelegation:$false -Credential (Get-Credential) -CriticalReplicationOnly:$false -DatabasePath "C:\Windows\NTDS" -DomainName "Contoso.com" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SiteName "Default-First-Site-Name" -SysvolPath "C:\Windows\SYSVOL" -Force:$true}
   ```
1. コマンドを呼び出すには、Enter キーを押します。
1. **[Windows PowerShell 資格情報要求]** ダイアログ ボックスに講師から提供された資格情報を入力します。
1. パスワードの入力を求めるメッセージが表示されたら、**[SafeModeAdministratorPassword]** テキスト ボックスに「**Pa55w.rd**」と入力し、Enter キーを押します。
1. 確認を求めるメッセージが表示されたら、**[SafeModeAdministratorPassword の確認]** テキスト ボックスに「**Pa55w.rd**」と入力し、Enter キーを押します。
1. コマンドが実行され、**正常な状態**というメッセージが返されるまで待ちます。 **SEA-SVR1** 仮想マシンが再起動します。
1. ファイルを保存せずにメモ帳を閉じます。
1. **SEA-SVR1** が再起動した後、**SEA-ADM1** で **[サーバー マネージャー]** に切り替え、左側の **[AD DS]** ノードを選択します。 **SEA-SVR1** がサーバーとして追加されたこと、および警告通知が表示されなくなったことに注意してください。

   > **注**: **[更新]** を選択する必要がある場合があります。

#### タスク 3: AD DS でオブジェクトを管理する

1. **SEA-ADM1** のコンソール セッションに接続されていることを確かめます。
1. **Windows PowerShell (管理者)** に切り替えます。
1. Contoso AD DS ドメインに **Seattle** という組織単位 (OU) を作成するには、次のコマンドを入力し、Enter キーを押します。

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle" -Path "DC=contoso,DC=com" -ProtectedFromAccidentalDeletion $true -Server SEA-DC1.contoso.com
   ```
1. **Seattle** OU で **Ty Carlson** のユーザー アカウントを作成するには、次のコマンドを入力し、Enter キーを押します。

   ```powershell
   New-ADUser -Name Ty -DisplayName 'Ty Carlson' -GivenName Ty -Surname Carlson -Path 'OU=Seattle,DC=contoso,DC=com'
   ```
1. Ty のユーザー アカウントのパスワードを設定するには、次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Set-ADAccountPassword Ty
   ```
1. 現在のパスワードの入力を求めるメッセージが表示されたら、Enter キーを押します。
1. 目的のパスワードの入力を求めるメッセージが表示されたら、「**Pa55w.rd**」と入力し、Enter キーを押します。
1. パスワードをもう一度入力するよう求められたら、「**Pa55w.rd**」と入力し、Enter キーを押します。
1. アカウントを有効にするには、次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Enable-ADAccount Ty
   ```
1. **SeattleBranchUsers** という名前のドメイン グローバル グループを作成するには、次のコマンドを入力し、Enter キーを押します。

   ```powershell
   New-ADGroup SeattleBranchUsers -Path 'OU=Seattle,DC=contoso,DC=com' -GroupScope Global -GroupCategory Security
   ```
1. 新しく作成されたグループに **Ty** ユーザー アカウントを追加するには、次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Add-ADGroupMember -Identity SeattleBranchUsers -Members Ty
   ```
1. ユーザーがグループに存在していることを確認するには、次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Get-ADGroupMember -Identity SeattleBranchUsers
   ```
1. ローカルの管理者グループにユーザーを追加するには、次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Add-LocalGroupMember -Group 'Administrators' -Member 'CONTOSO\Ty'
   ```

   > **注**: これは、**CONTOSO\\Ty** ユーザー アカウントで **SEA-ADM1** にサインインできるようにするために必要です。

**結果**: この演習が完了すると、AD DS に新しいドメイン コントローラーとマネージド オブジェクトを正常に作成したことになります。

## 演習 2: グループ ポリシーの構成

#### タスク 1: GPO を作成および編集する

1. **SEA-ADM1** で、サーバー マネージャーから **[ツール]** を選択し、**[グループ ポリシーの管理]** を選びます。
1. 必要に応じて、**[グループ ポリシーの管理]** ウィンドウに切り替えます。
1. **[グループ ポリシーの管理]** コンソールのナビゲーション ペインで、**Forest:Contoso.com**、**Domains**、および **Contoso.com** を展開してから、**[グループ ポリシー オブジェクト]** コンテナーを選択します。
1. ナビゲーション ペインで、**[グループ ポリシー オブジェクト]** コンテナーのコンテキスト メニューを右クリックするかそこにアクセスして、**[新規]** を選択します。
1. **[名前]** テキスト ボックスに「**CONTOSO Standards**」と入力し、**[OK]** を選択します。
1. 詳細ペインで、**CONTOSO Standards** グループ ポリシー オブジェクト (GPO) のコンテキスト メニューを右クリックまたはそこにアクセスし、**[編集]** を選択します。
1. **[グループ ポリシー管理エディター]** ウィンドウのナビゲーション ペインで、**[ユーザーの構成]**、**[ポリシー]**、**[管理用テンプレート]** の順に展開してから **[システム]** を選択します。
1. **[レジストリ編集ツールへアクセスできないようにする]** ポリシー設定をダブルクリックするか、その設定を選択してから、Enter キーを押します。
1. **[レジストリ編集ツールへアクセスできないようにする]** ダイアログ ボックスで、**[有効]**、**[OK]** の順に選択します。
1. ナビゲーション ペインで、**[ユーザーの****構成]**、**[ポリシー]**、**[管理用テンプレート]** の順に展開し、**[コントロール パネル]** を展開してから **[個人用設定]** を選択します。
1. 詳細ペインで、**[スクリーン セーバーのタイムアウト]** ポリシー設定をダブルクリックまたは選択してから、Enter キーを押します。
1. **[スクリーン セーバーのタイムアウト]** ダイアログ ボックスで、**[有効]** を選択します。 **[秒]** テキスト ボックスに「**600**」と入力してから、**[OK]** を選択します。 
1. **[パスワードでスクリーン セーバーを保護する]** ポリシー設定をダブルクリックまたは選択してから、Enter キーを押します。
1. **[スクリーン セーバーをパスワードで保護する]** ダイアログ ボックスで、**[有効]** を選んでから、**[OK]** を選択します。
1. **[グループ ポリシー管理エディター]** ウィンドウを閉じます。

#### タスク 2: GPO をリンクする

1. **[グループ ポリシーの管理]** ウィンドウのナビゲーション ペインで、`Contoso.com` ドメインのコンテキスト メニューを右クリックまたはそこにアクセスし、**[既存の GPO のリンク]** を選択します。
1. **[GPO の選択]** ダイアログ ボックスで、**[CONTOSO Standards]** を選んでから、**[OK]** を選択します。

#### タスク 3: GPO の設定の効果を確認する

1. **SEA-ADM1** で、タスク バーの検索ボックスに「**コントロール パネル**」と入力します。 
1. **[最も一致する検索結果]** リストで、**[コントロール パネル]** を選択します。
1. **[システムとセキュリティ]** を選択してから、**[Windows ファイアウォールによるアプリケーションの許可]** を選びます。
1. **[許可されているアプリと機能]** リストで、**[リモート イベント ログ管理]** エントリを探し、**[ドメイン]** 列のチェックボックスをオンにしてから、**[OK]** を選択します。 
1. サインアウトしてから、パスワード **Pa55w.rd** を使用して **CONTOSO\\Ty** としてサインインします。
1. タスク バーの検索ボックスに、「**コントロール パネル**」と入力します。
1. **[最も一致する検索結果]** リストで、**[コントロール パネル]** を選択します。
1. [コントロール パネル] の検索ボックスに「**スクリーン セーバー**」と入力し、**[スクリーン セーバーの変更]** を選択します (オプションが表示されるまでに数分かかる場合があります)。
1. **[スクリーン セーバーの設定]** ダイアログ ボックスで、**[待ち時間]** オプションが淡色表示になっていることに注目してください。 タイムアウトを変更することはできません。**[再開時にログオン画面に戻る]** オプションが選択されて淡色表示になっており、設定を変更できないことに注目にしてください。

   > **注**: **[再開時にログオン画面に戻る]** オプションが選択されて淡色表示になっていない場合は、コマンド プロンプトを開いて `gpupdate /force` を実行し、前の手順を繰り返します。

1. **[スタート]** のコンテキスト メニューを右クリックまたはそこにアクセスし、**[ファイル名を指定して実行]** を選択します。
1. **[ファイル名を指定して実行]** ダイアログ ボックスの **[名前]** ボックスに「**regedit**」と入力し、**[OK]** を選択します。 **レジストリ編集は、管理者によって使用不可にされています**というエラー メッセージに注意してください。
1. **[レジストリエディター]** ダイアログ ボックスで、**[OK]** を選択します。
1. サインアウトしてから、講師から提供された資格情報でサインインします。

#### タスク 4: 必要な GPO を作成してリンクする

1. **SEA-ADM1** で、サーバー マネージャーから **[ツール]** を選択し、**[グループ ポリシーの管理]** を選びます。
1. 必要に応じて、**[グループ ポリシーの管理]** ウィンドウに切り替えます。
1. **[グループ ポリシーの管理]** コンソールのナビゲーション ペインで、**Forest:Contoso.com**、**Domains**、および **Contoso.com** を展開してから、**[Seattle]** を選択します。
1. **Seattle** 組織単位 (OU) のコンテキスト メニューを右クリックするかそこにアクセスしてから、**[このドメインに GPO を作成し、このコンテナーにリンクする]** を選択します。
1. **[新しい GPO]** ダイアログ ボックスの **[名前]** テキスト ボックスに「**Seattle Application Override**」と入力し、**[OK]** を選択します。
1. 詳細ペインで、**Seattle Application Override** GPO のコンテキスト メニューを右クリックまたはそこにアクセスし、**[編集]** を選択します。
1. **コンソール** ツリーで、**[ユーザーの構成]**、**[ポリシー]**、**[管理用テンプレート]** の順に展開し、**[コントロール パネル]** を展開してから **[個人用設定]** を選択します。
1. **[スクリーン セーバーのタイムアウト]** ポリシー設定をダブルクリックするか、設定を選択して、Enter キーを押します。
1. **[無効]** を選択してから、**[OK]** を選択します。
1. **[グループ ポリシー管理エディター]** ウィンドウを閉じます。

#### タスク 5: 優先順位を確認する

1. **[グループ ポリシー管理コンソール]** ツリーに戻り、**Seattle** OU が選択されていることを確かめます。
1. **[グループ ポリシーの継承]** タブを選択し、その内容を確認します。

   > **注**: Seattle Application Override GPO は CONTOSO Standards GPO より優先順位が高くなります。 Seattle Application Override GPO で先ほど構成したスクリーン セーバーのタイムアウト ポリシー設定は、CONTOSO Standards GPO の設定の後に適用されます。 そのため、新しい設定で CONTOSO Standards GPO 設定が上書きされます。 Seattle Application Override GPO のスコープ内のユーザーに対して、スクリーン セーバーのタイムアウトが無効になります。

#### タスク 6: セキュリティ フィルター処理を使用して GPO のスコープを構成する

1. **SEA-ADM1** の **[グループ ポリシーの管理]** コンソールのナビゲーション ペインで、必要に応じて、**Seattle** OU を展開し、**Seattle** OU の下にある **Seattle Application Override** GPO を選択します。
1. **[グループ ポリシー管理コンソール]** ダイアログ ボックスで、**グループ ポリシー オブジェクト (GPO) へのリンクを選択しました。リンク プロパティへの変更以外、ここで行われた変更は GPO にグローバルに適用され、この GPO がリンクされた他の場所すべてに影響します**というメッセージを確認します。
1. **[今後このメッセージを表示しない]** チェックボックスをオンにしてから、**[OK]** を選択します。
1. **[セキュリティ フィルター処理]** セクションを確認し、認証されたユーザーに GPO が既定で適用されることに注意してください。
1. **[セキュリティ フィルター処理]** セクションで、**[認証されたユーザー]** を選んでから **[削除]** を選択します。
1. **[グループ ポリシーの管理]** ダイアログ ボックスで **[OK]** を選択し、**[グループ ポリシーの管理]** の警告を確認してからもう一度 **[OK]** を選択します。

   > **注**: グループ ポリシーでは、ユーザーの GPO 設定を正常に適用するために、各コンピューター アカウントにドメイン コントローラーから GPO データを読み取るアクセス許可が求められます。 GPO のセキュリティ フィルター処理設定を変更する場合は、これに注意してください。

1. 詳細ペインで、**[追加]** を選択します。
1. **[ユーザー、コンピューター、またはグループの選択]** ダイアログ ボックスで、**[選択するオブジェクト名を入力してください (例):]** テキスト ボックスに「**SeattleBranchUsers**」と入力してから、**[OK]** を選択します。
1. 詳細ペインの **[セキュリティ フィルター処理]** で、**[追加]** を選択します。
1. **[ユーザー、コンピューター、またはグループの選択]** ダイアログ ボックスで、**[オブジェクトの種類]** を選択します。
1. **[オブジェクトの種類]** ダイアログ ボックスで、**[コンピューター]** チェックボックスをオンにしてから、**[OK]** を選択します。
1. **[ユーザー、コンピューター、またはグループの選択]** ダイアログ ボックスで、**[選択するオブジェクト名を入力してください (例):]** ボックスに「**SEA-ADM1**」と入力してから **[OK]** を選択します。

#### タスク 7: 設定の適用を確認する

1. ナビゲーション ペインの **[グループ ポリシーの管理]** で、**[グループ ポリシーのモデル作成]** を選択します。
1. **[グループ ポリシーのモデル作成]** のコンテキスト メニューを右クリックするかそこにアクセスし、**[グループ ポリシーのモデル作成ウィザード]** を選択します。
1. **[グループ ポリシーのモデル作成ウィザード]** で、**[次へ]** を選択します。
1. **[ドメイン コントローラーの選択]** ページで、既定の設定を受け入れ、**[次へ]** を選択します。
1. **[ユーザーとコンピューターの選択]** ページの **[ユーザー情報]** セクションで、**[ユーザー]** を選択してから、**[ユーザー]** テキスト ボックスに「**CONTOSO\Ty**」と入力するか、**[参照]** コマンド ボタンを使用して **Ty** ユーザー アカウントを見つけます。
1. **[ユーザーとコンピューターの選択]** ページの **[コンピューター情報]** セクションで、 **[コンピューター]** を選択してから、 **[コンピューター]** テキスト ボックスに「**CONTOSO\SEA-ADM1**」と入力するか、 **[参照]** コマンド ボタンを使用して **SEA-ADM1** コンピューターを見つけます。
1. **[ユーザーとコンピューターの選択]** ページで、**[次へ]** を選択します。
1. **[詳細シミュレーション オプション]** ページで、既定の設定を受け入れ、**[次へ]** を選択します。
1. **[代替 Active Directory パス]** ページの、ユーザーとコンピューターの場所に注意してください。その後、**[次へ]** を選択します。
1. **[ユーザー セキュリティ グループ]** ページで、グループのリストに **CONTOSO\\ SeattleBranchUsers** が含まれていることを確認してから、**[次へ]** を選択します。
1. **[コンピューター セキュリティ グループ]** ページで、**[次へ]** を選択します。
1. **[ユーザーの WMI フィルター]** ページで、既定の設定を受け入れ、**[次へ]** を選択します。
1. **[コンピューターの WMI フィルター]** ページで、既定の設定を受け入れ、**[次へ]** を選択します。
1. **[選択の要約]** ページで、**[次へ]** を選択します。
1. メッセージが表示されたら、**[完了]** を選択します。
1. 詳細ペインで、**[詳細]** タブを選択し、**[すべて表示]** を選びます。
1. レポートで、 **[ユーザーの詳細]** セクションが見つかるまで下にスクロールし、 **[コントロールパネル] の [個人用設定]** セクションを見つけます。 **[スクリーン セーバーのタイムアウト]** 設定が無効になっており、優勢な GPO がSeattle Application Override GPO に設定されていることに注意してください。
1. **[グループ ポリシーの管理]** コンソールを閉じます。

**結果**: この演習が完了すると、GPO を正常に作成して構成したことになります。
