---
lab:
  title: 'ラボ: ID サービスおよびグループ ポリシーの実装'
  module: 'Module 1: Identity services in Windows Server'
ms.openlocfilehash: 34c6259da5db4beca5e31998564c06102902ca48
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907039"
---
# <a name="lab-implementing-identity-services-and-group-policy"></a>ラボ: ID サービスおよびグループ ポリシーの実装

## <a name="scenario"></a>シナリオ

あなたは Contoso Ltd. に管理者として勤務しています。この会社は、いくつかの新しい場所でビジネスを拡大しています。 現在、Active Directory Domain Services (AD DS) 管理チームは、非対話型のリモート ドメイン コントローラーの展開のために Windows Server で使用できる方法を評価しています。 また、チームは特定の AD DS 管理タスクを自動化する方法を探しています。 さらに、チームは、グループ ポリシー オブジェクト (GPO) に基づいて構成管理を確立したいと考えています。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Server Core に新しいドメイン コントローラーを展開する。
- グループ ポリシーを構成する。

## <a name="estimated-time-45-minutes"></a>予想所要時間: 45 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-ADM1**、および **AZ-800T00A-SEA-SVR1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-ADM1**、および **AZ-800T00A-SEA-SVR1** 仮想マシンによって、**SEA-DC1**、**SEA-SVR1**、および **SEA-ADM1** のインストールがホストされます。 

1. **SEA-ADM1** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

## <a name="exercise-1-deploying-a-new-domain-controller-on-server-core"></a>演習 1: Server Core への新しいドメイン コントローラーの展開

### <a name="scenario"></a>シナリオ

ビジネスの再構築の一環として、Contoso では、リモートの場所での IT の関与を最小限に抑え、リモート サイトに新しいドメイン コントローラーを展開したいと考えています。 DC 展開を使用して、新しいドメイン コントローラーを展開する必要があります。

この演習の主なタスクは次のとおりです。

1. 新しい Windows Server Core サーバーに AD DS を展開します。
1. GUI ツールおよび Windows PowerShell を使用して、AD DS オブジェクトを管理します。

#### <a name="task-1-deploy-ad-ds-on-a-new-windows-server-core-server"></a>タスク 1: 新しい Windows Server Core サーバーに AD DS を展開する

1. **SEA-ADM1** に切り替え、 **[サーバー マネージャー]** から Windows PowerShell を開きます。
1. Windows PowerShell で **Install-WindowsFeature** コマンドレットを使用して、**SEA-SVR1** に AD DS 役割をインストールします。
1. **Get-WindowsFeature** コマンドレットを使用してインストールを確認します。
1. **[Active Directory Domain Services]** 、 **[リモート サーバー管理ツール]** 、および **[役割管理ツール]** チェックボックスをオンにしていることを確かめます。 **[AD DS** および **AD LDS ツール]** ノードの場合は、**Windows PowerShell 用 Active Directory モジュール** のみをインストールし、Active Directory 管理センターなどのグラフィカル ツールをインストールする必要はありません。

> **注**: サーバーを中央管理する場合、通常は各サーバーに GUI ツールは必要ありません。 それらをインストールする場合は、**RSAT-ADDS** コマンドを使用して **Add-WindowsFeature** コマンドレットを実行し、AD DS ツールを指定する必要があります。

> **注**: インストール プロセスが完了してから、AD DS 役割がインストールされていることを確認する前に待つ必要がある場合があります。 **Get-WindowsFeature** コマンドから予想される結果が得られない場合は、数分後に再度試すことができます。

#### <a name="task-2-prepare-the-ad-ds-installation-and-promote-a-remote-server"></a>タスク 2: AD DS インストールの準備をして、リモート サーバーを昇格させる

1. **SEA-ADM1** の **[サーバー マネージャー]** の **[すべてのサーバー]** ノードで、**SEA-SVR1** をマネージド サーバーとして追加します。
1. **SEA-ADM1** の **[サーバー マネージャー]** から、次の設定を使用して AD DS ドメイン コントローラーとして **SEA-SVR1** を構成します。

   - 種類: 既存のドメインの追加ドメイン コントローラー
   - ドメイン: `contoso.com`
   - 資格情報: パスワード **Pa55w.rd** を持つ **CONTOSO\\Administrator**
   - ディレクトリ サービス復元モード (DSRM) パスワード: **Pa55w.rd**
   - DNS とグローバル カタログの選択を削除しない

1. **[オプションの確認]** ページで、 **[スクリプトの表示]** を選択します。
1. メモ帳で、次のように生成された Windows PowerShell スクリプトを編集します。

   - 番号記号 (#)で始まるコメント行を削除します。
   - Import-Module 行を削除します。
   - 各行の末尾にあるアクサン グラーブ (`) を削除します。
   - 改行を削除します。

1. これで、**Install-ADDSDomainController** コマンドとすべてのパラメーターが 1 行に収まったので、コマンドをコピーします。
1. **[Active Directory Domain Services 構成ウィザード]** に切り替えてから、 **[キャンセル]** を選択します。
1. Windows PowerShell に切り替えてから、コマンド プロンプトで次のコマンドを入力します。

   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 { }
   ```
1. コピーしたコマンドを中かっこ ({ }) の間に貼り付け、結果のコマンドを実行してインストールを開始します。 完全なコマンドは次の形式になるはずです。

   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 {Install-ADDSDomainController -NoGlobalCatalog:\$false -CreateDnsDelegation:\$false -Credential (Get-Credential) -CriticalReplicationOnly:\$false -DatabasePath "C:\Windows\NTDS" -DomainName "Contoso.com" -InstallDns:\$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:\$false -SiteName "Default-First-Site-Name" -SysvolPath "C:\Windows\SYSVOL" -Force:\$true}
   ```

1. 次の資格情報を入力します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. **SafeModeAdministratorPassword** を **Pa55w.rd** として設定します。
1. **SEA-SVR1** が再起動した後、**SEA-ADM1** で **[サーバー マネージャー]** に切り替えてから **[AD DS]** ノードを選択します。 **SEA-SVR1** がドメイン コントローラーとして追加されたこと、および警告通知が表示されなくなったことに注意してください。 **[更新]** を選択する必要がある場合があります。

#### <a name="task-3-manage-objects-in-ad-ds"></a>タスク 3: AD DS でオブジェクトを管理する

1. **SEA-ADM1** で、**Windows PowerShell** コンソールに切り替えます。
1. **Seattle** という組織単位 (OU) を作成するには、**Windows PowerShell** コンソールで次のコマンドを実行します。

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle" -Path "DC=contoso,DC=com" -ProtectedFromAccidentalDeletion $true -Server SEA-DC1.contoso.com
   ```
1. **Seattle** OU で **Ty Carlson** のユーザー アカウントを作成するには、次のコマンドを実行します。

   ```powershell
   New-ADUser -Name Ty -DisplayName 'Ty Carlson' -GivenName Ty -Surname Carlson -Path 'OU=Seattle,DC=contoso,DC=com'
   ```
1. ユーザーのパスワードを **Pa55w.rd** に設定するには、次のコマンドを実行します。

   ```powershell
   Set-ADAccountPassword Ty
   ```

> **注**: 現在のパスワードは空白です。

1. ユーザー アカウントを有効にするには、次のコマンドを実行します。

   ```powershell
   Enable-ADAccount Ty
   ```
1. **SeattleBranchUsers** という名前のドメイン グローバル グループを作成するには、次のコマンドを実行します。

   ```powershell
   New-ADGroup SeattleBranchUsers -Path 'OU=Seattle,DC=contoso,DC=com' -GroupScope Global -GroupCategory Security
   ```
1. 新しく作成されたグループに **Ty** ユーザー アカウントを追加するには、次のコマンドを実行します。

   ```powershell
   Add-ADGroupMember -Identity SeattleBranchUsers -Members Ty
   ```
1. ユーザーがグループに存在することを確認するには、次のコマンドを実行します。

   ```powershell
   Get-ADGroupMember -Identity SeattleBranchUsers
   ```
1. ローカルの管理者グループにユーザーを追加するには、次のコマンドを実行します。

   ```powershell
   Add-LocalGroupMember -Group 'Administrators' -Member 'CONTOSO\Ty'
   ```

   > **注**: これは、**CONTOSO\\Ty** ユーザー アカウントで **SEA-ADM1** にサインインできるようにするために必要です。

### <a name="results"></a>結果

この演習が完了すると、AD DS に新しいドメイン コントローラーとマネージド オブジェクトを作成したことになります。

## <a name="exercise-2-configuring-group-policy"></a>演習 2: グループ ポリシーの構成

### <a name="scenario"></a>シナリオ

グループ ポリシーの実装の一環として、Office アプリ用のカスタム管理テンプレートをインポートし、設定を構成したいと考えています。

この演習の主なタスクは次のとおりです。

1. GPO 設定を作成して編集します。
1. クライアント コンピューターで設定を適用して確認します。

### <a name="task-1-create-and-edit-a-gpo"></a>タスク 1: GPO を作成および編集する

1. **SEA-ADM1** で、 **[サーバー マネージャー]** から **[グループ ポリシーの管理]** コンソールを開きます。
1. **Contoso Standards** という名前の GPO を **[グループ ポリシー オブジェクト]** コンテナーに作成します。
1. **Contoso Standards** GPO をグループ ポリシー管理エディターで開いてから、 **[ユーザーの構成]、[ポリシー]、[管理用テンプレート]、[システム]** の順に移動します。
1. **[レジストリ編集ツールへアクセスできないようにする]** ポリシー設定を有効にします。
1. **[ユーザーの構成]、[ポリシー]、[管理用テンプレート]、[コントロール パネル]、[個人用設定]** フォルダーの順に移動し、**スクリーン セーバー** のタイムアウト ポリシーを **600** 秒になるように構成します。
1. **[パスワードでスクリーン セーバーを保護する]** ポリシー設定を有効にしてから、 **[グループ ポリシー管理エディター]** ウィンドウを閉じます。

### <a name="task-2-link-the-gpo"></a>タスク 2: GPO をリンクする

  - **Contoso Standards** GPO を `contoso.com` ドメインにリンクします。

### <a name="task-3-review-the-effects-of-the-gpos-settings"></a>タスク 3: GPO の設定の効果を確認する

1. **SEA-ADM1** で、 **[コントロール パネル]** を開きます。
1. **Windows Defender ファイアウォール** インターフェイスを使用して、 **[リモート イベントのログ管理]** ドメイン トラフィックを有効にします。 
1. サインアウトしてから、パスワード **Pa55w.rd** を使用して **CONTOSO\\Ty** としてサインインします。
1. スクリーン セーバーの待機時間を変更し、設定を再開してみます。 グループ ポリシーによりこれらの操作がブロックされていることを確認します。
1. レジストリ エディターを実行してみます。 グループ ポリシーによりこの操作がブロックされていることを確認します。 
1. サインアウトしてから、パスワード **Pa55w.rd** を使用して **CONTOSO\\Administrator** としてサインインします。

#### <a name="task-4-create-and-link-the-required-gpos"></a>タスク 4: 必要な GPO を作成してリンクする

1. **SEA-ADM1** の **[グループ ポリシーの管理]** コンソールで、**Seattle** OU にリンクされている **Seattle Application Override** という名前の新しい GPO を作成します。
1. **[スクリーン セーバーのタイムアウト]** ポリシー設定が無効になるように構成してから、 **[グループ ポリシー管理エディター]** ウィンドウを閉じます。

### <a name="task-5-verify-the-order-of-precedence"></a>タスク 5: 優先順位を確認する

1. **SEA-ADM1** で、 **[サーバー マネージャー]** から **[グループ ポリシーの管理]** コンソールを開きます。
1. **[グループ ポリシー管理コンソール]** ツリーで、**Seattle** OU を選択します。
1. **[グループ ポリシーの継承]** タブを選択し、その内容を確認します。

   > **注**: Seattle Application Override GPO は CONTOSO Standards GPO より優先順位が高くなります。 Seattle Application Override GPO で先ほど構成したスクリーン セーバーのタイムアウト ポリシー設定は、CONTOSO Standards GPO の設定の後に適用されます。 そのため、新しい設定で CONTOSO Standards GPO 設定が上書きされます。 Seattle Application Override GPO のスコープ内のユーザーに対して、スクリーン セーバーのタイムアウトが無効になります。

### <a name="task-6-configure-the-scope-of-a-gpo-with-security-filtering"></a>タスク 6: セキュリティ フィルター処理を使用して GPO のスコープを構成する

1. **SEA-ADM1** の **[グループ ポリシーの管理]** コンソールで、**Seattle Application Override** GPO を選択します。 **[セキュリティ フィルター処理]** セクションで、認証されたすべてのユーザーに GPO が既定で適用されていることに注目してください。
1. **[セキュリティ フィルター処理]** セクションで、まず **認証されたユーザー** を削除してから **SeattleBranchUsers** グループと **SEA-ADM1** コンピューター アカウントを追加します。

### <a name="task-7-verify-the-application-of-settings"></a>タスク 7: 設定の適用を確認する

1. [グループ ポリシーの管理] のナビゲーション ペインで、 **[グループ ポリシーのモデル作成]** を選択します。
1. **[グループ ポリシーのモデル作成ウィザード]** を起動します。
1. ターゲット ユーザーとコンピューターをそれぞれ **CONTOSO\\Ty** ユーザー アカウントと **SEA-ADM1** コンピューターに設定します。
1. ウィザードの残りのページをステップ実行し、既定の設定を変更せずに確認し、ウィザードを完了すると、その結果を含むレポートが生成されます。
1. レポートが作成された後、詳細ペインで **[詳細]** タブを選択してから、 **[すべて表示]** を選びます。
1. レポートで、 **[ユーザーの詳細]** セクションが見つかるまで下にスクロールし、 **[コントロールパネル] の [個人用設定]** セクションを見つけます。 **[スクリーン セーバーのタイムアウト]** 設定が、Seattle Application Override GPO から取得されていることがわかるはずです。
1. **[グループ ポリシーの管理]** コンソールを閉じます。

### <a name="results"></a>結果

この演習が完了すると、GPO を正常に作成して構成したことになります。
