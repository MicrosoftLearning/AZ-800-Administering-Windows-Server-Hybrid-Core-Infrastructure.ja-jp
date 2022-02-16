---
lab:
  title: 'ラボ: AD DS と Azure AD の統合の実装'
  type: Answer Key
  module: 'Module 2: Implementing Identity in Hybrid Scenarios'
ms.openlocfilehash: 8cbcc563a86ca5c1c997a69884e1b132cbf2f86e
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137906962"
---
# <a name="lab-implementing-integration-between-ad-ds-and-azure-ad"></a>ラボ: AD DS と Azure AD の統合の実装

## <a name="exercise-1-preparing-azure-ad-for-ad-ds-integration"></a>演習 1: AD DS 統合のための Azure AD の準備

#### <a name="task-1-create-a-custom-domain-in-azure"></a>タスク 1: Azure でカスタム ドメインを作成する

1. **SEA-ADM1** に接続し、必要に応じて、パスワード **Pa55w.rd** を使用し、**Contoso\\Administrator** としてサインインします。
1. **SEA-ADM1** で Microsoft Edge を起動し、Azure portal を参照して、Azure 資格情報で認証します。
1. Azure portal で、 **[Azure Active Directory]** に移動します。
1. **[Azure Active Directory]** ページで、 **[カスタム ドメイン名]** を選択します。
1. **[カスタムドメイン名]** ページで、 **[カスタムドメインの追加]** を選択します。
1. **[カスタム ドメイン名]** ペインで、 **[カスタム ドメイン名]** ボックスに「**contoso.com**」と入力し、 **[ドメインの追加]** を選択します。
1. `contoso.com` の [カスタムドメイン名] ページで、ドメインを確認するために使用するドメイン ネーム システム (DNS) レコードの種類を確認します。
1. ドメイン名を確認せずにウィンドウを閉じます。

   > **注**: 一般には、DNS レコードを使用してドメインを確認しますが、このラボでは検証済みドメインを使用する必要はありません。

#### <a name="task-2-create-a-user-with-the-global-administrator-role"></a>タスク 2: グローバル管理者ロールを持つユーザーを作成する

1. **SEA-ADM1** の Azure portal 内の **[Azure Active Directory]** ページで、 **[ユーザー]** を選択します。
1. **[すべてのユーザー]** ページで、 **[新しいユーザー]** を選択します。
1. **[新しいユーザー]** ページの **[ID]** で、 **[ユーザー名]** および **[名前]** テキスト ボックスに「**admin**」と入力します。

   > **注**: **[ユーザー名]** の [ドメイン名] ドロップダウンメニューに、`onmicrosoft.com` で終わる既定のドメイン名が表示されていることを確認します。

1. **[パスワード]** で、 **[パスワードを表示する]** チェックボックスをオンにします。 このラボで後ほど使用するので、パスワードを記録しておきます。
1. **[グループとロール]** の **[ロール]** の横にある **[ユーザー]** を選択します。
1. **[ディレクトリ ロール]** ページで、ロールの一覧から **[グローバル管理者]** を選択し、 **[選択]** を選びます。
1. **[新しいユーザー]** ページに戻り、 **[使用場所]** ドロップダウン リストの **[米国]** を選択します。
1. **[新しいユーザー]** ページで、 **[作成]** を選択します。

#### <a name="task-3-change-the-password-for-the-user-with-the-global-administrator-role"></a>タスク 3: グローバル管理者ロールを持つユーザーのパスワードを変更する

1. Azure portal の右上でご自分のユーザー アカウントを選択して、 **[サインアウト]** を選択します。
1. **[アカウントの選択]** ページで、 **[別のアカウントを使用する]** を選択します。
1. **[サインイン]** ページで、前に作成したユーザー アカウントの完全修飾ユーザー名を入力し、 **[次へ]** を選択します。
1. 現在のパスワードについては、前の手順で書き留めたパスワードを使用します。
1. 複雑なパスワードを 2 回入力し、 **[サインイン]** を選択します。

   > **注**: 使用した複雑なパスワードは、このラボで後ほど使用するので、記録しておきます。

## <a name="exercise-2-preparing-on-premises-ad-ds-for-azure-ad-integration"></a>演習 2: Azure AD 統合のためのオンプレミス AD DS の準備

#### <a name="task-1-install-idfix"></a>タスク 1: IdFix をインストールする

1. **SEA-ADM1** で Microsoft Edge を起動し、 **https://github.com/microsoft/idfix** を参照します。
1. **Github** ページの **[ClickOnce の起動]** で、 **[起動]** を選択します。
1. ステータスバーで、 **[ファイルを開く]** を選択します。
1. **[アプリケーションのインストール-セキュリティの警告]** ダイアログ ボックスで、 **[インストール]** を選択します。
1. **[IdFix プライバシーステートメント]** ダイアログ ボックスで、免責事項を確認し、 **[OK]** を選択します。

#### <a name="task-2-run-idfix"></a>タスク 2: IdFix を実行する

1. **IdFix** ウィンドウで、 **[クエリ]** を選択します。
1. **[スキーマの警告]** ダイアログ ボックスが表示されたら、 **[はい]** を選択します。
1. Active Directory でオブジェクトの一覧を確認し、 **[エラー]** および **[属性]** 列に注目します。 このシナリオでは、**ContosoAdmin** の **displayName** の値が空白で、ツールの推奨される新しい値が **更新** 列に表示されます。
1. **IdFix** ウィンドウで、 **[アクション]** ドロップダウン メニューから **[編集]** を選択し、 **[適用]** を選択すると、推奨される変更が自動的に実装されます。
1. **[保留の適用]** ダイアログ ボックスで、 **[はい]** を選択します。
1. IdFix ツールを閉じます。

## <a name="exercise-3-downloading-installing-and-configuring-azure-ad-connect"></a>演習 3: Azure AD Connect のダウンロード、インストール、構成

#### <a name="task-1-install-and-configure-azure-ad-connect"></a>タスク 1: Azure AD Connect のインストールと構成

1. **SEA-ADM1** 上の、Azure portal が表示されている Microsoft Edge ウィンドウで、 **[Azure Active Directory]** を参照します。
1. **[Azure Active Directory]** ページで、 **[Azure AD Connect]** を選びます。
1. **[Azure AD Connect]** ページで、 **[Azure AD Connect のダウンロード]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ページで、 **[ダウンロード]** を選択します。
1. ステータスバーで、 **[ファイルを開く]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ページで、 **[ライセンス条項とプライバシーに関する声明に同意します]** チェックボックスをオンにし、 **[続行]** を選択します。
1. **[簡易設定]** ページで、 **[簡易設定を使用する]** を選択します。
1. **[Azure AD への接続]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力し、 **[次へ]** を選択します。
1. **[AD DS への接続]** ページで、次の資格情報を入力し、 **[次へ]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. **[Azure AD のサインイン構成]** ページで、追加した新しいドメインが Active Directory UPN サフィックスの一覧に表示されていますが、その状態は **[未検証]** と表示されていることに注目してください。

   > **注**: 指定されたドメイン名は、検証済みドメインである必要はありません。 通常は Azure AD Connect をインストールする前にドメインを検証しますが、このラボでは検証手順は必要ありません。

1. **[一部の UPN サフィックスが検証済みドメインに一致していなくても続行する]** チェックボックスをオンにし、 **[次へ]** を選択します。
1. **[構成の準備完了]** ページで、アクションの一覧を確認し、 **[インストール]** を選択します。
1. **[構成が完了しました]** ページで、 **[終了]** を選択します。

## <a name="exercise-4-verifying-integration-between-ad-ds-and-azure-ad"></a>演習 4: AD DS と Azure AD の統合の検証

#### <a name="task-1-verify-synchronization-in-the-azure-portal"></a>タスク 1: Azure portal で同期を検証する

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。 
1. **Azure AD Connect** ページを更新し、 **[Active Directory からのプロビジョニング]** の下の情報を確認します。
1. **[Azure Active Directory]** ページで、 **[ユーザー]** を選びます。
1. ユーザー一覧には Active Directory から同期されたユーザーが含まれていることにご注意ください。

   > **注**: ディレクトリ同期が開始されると、Active Directory オブジェクトが Azure AD ポータルに表示されるまでに 15 分かかることがあります。

1. Microsoft Edge で、 **[Azure Active Directory]** ページに戻ります。
1. **[Azure Active Directory]** ページで、 **[グループ]** を選びます。
1. Active Directory から同期されたグループの一覧に注目します。 

#### <a name="task-2-verify-synchronization-in-the-synchronization-service-manager"></a>タスク 2: Synchronization Service Manager で同期を検証する

1. **SEA-ADM1** の **[スタート]** メニューで、 **[Azure AD Connect]** を展開し、 **[同期サービス]** を選択します。
1. **[Synchronization Service Manager]** ウィンドウの **[操作]** タブで、Active Directory オブジェクトを同期するために実行されたタスクを確認します。
1. **[コネクタ]** タブを選択し、2 つのコネクタに注目します。

   > **注**: 1 つのコネクタが AD DS 用であり、もう 1 つは Azure AD テナント用です。 

1. **[Synchronization Service Manager]** ウィンドウを閉じます。

#### <a name="task-3-update-a-user-account-in-active-directory"></a>タスク 3: Active Directory でユーザー アカウントを更新する

1. **SEA-ADM1** で、サーバー マネージャーの **[ツール]** メニューから **[Active Directory ユーザーとコンピューター]** を選択します。
1. **[Active Directory ユーザーとコンピューター]** で、**Sales** 組織単位 (OU) を展開し、**Ben Miller** のプロパティを開きます。
1. ユーザーのプロパティで、 **[組織]** タブを選択します。
1. **[役職]** テキストボックスに「**マネージャー**」と入力し、 **[OK]** を選択します。

#### <a name="task-4-create-a-user-account-in-active-directory"></a>タスク 4: Active Directory でユーザー アカウントを作成する

1. **[Active Directory ユーザーとコンピューター]** で、**Sales** OU のコンテキスト メニューを右クリックするか、またはこれにアクセスして、 **[新規作成]** 、 **[ユーザー]** の順に選択します。
1. **[新しいオブジェクト - ユーザー]** ウィンドウで、各フィールドについて次のユーザーの詳細を入力し、 **[次へ]** を選択します。

   - 名: **Jordan**
   - 姓: **Mitchell**
   - ユーザーログオン名: **Jordan**

1. **[パスワード]** および **[パスワードの確認]** フィールドに「**Pa55w.rd**」と入力し、 **[次へ]** を選択します。
1. **[完了]** を選択します。

#### <a name="task-5-sync-changes-to-azure-ad"></a>タスク 5: Azure AD に対する変更を同期する

1. **SEA-ADM1** の **[スタート]** メニューで、**Windows PowerShell** を選択します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力しから Enter キーを押して同期をトリガーします。

   ```powershell
   Start-ADSyncSyncCycle
   ```

   > **注**: 同期サイクルが開始されると、Active Directory オブジェクトが Azure AD ポータルに表示されるまでに 15 分かかることがあります。

#### <a name="task-6-verify-changes-in-azure-ad"></a>タスク 6: Azure AD での変更を検証する

1. **SEA-ADM1** で、Microsoft Edge ウィンドウに切り替えて Azure portal を表示し、 **[Azure Active Directory]** ページに戻ります。
1. **[Azure Active Directory]** ページで、 **[ユーザー]** を選びます。
1. **[すべてのユーザー]** ページで、ユーザー **Ben** を検索します。
1. ユーザー **Ben Miller** のプロパティ ページを開き、**役職** の属性が Active Directory から同期されたことを確認します。
1. Microsoft Edge で、 **[すべてのユーザー]** ページに戻ります。
1. **[すべてのユーザー]** ページで、ユーザーを **Jordan** 検索します。
1. ユーザー **Jordan Mitchell** の [プロパティ] ページを開き、Active Directory から同期されたユーザー アカウントの属性を確認します。

## <a name="exercise-5-implementing-azure-ad-integration-features-in-ad-ds"></a>演習 5 AD DS での Azure AD 統合機能の実装

#### <a name="task-1-enable-self-service-password-reset-in-azure"></a>タスク 1: Azure でセルフサービス パスワード リセットを有効にする

1. **SEA-ADM1** 上の、Azure portal が表示されている Microsoft Edge ウィンドウで、 **[Azure Active Directory]** ページを参照します。
1. **[Azure Active Directory]** ページで、 **[ライセンス]** を選びます。
1. **[ライセンス]** ページで、 **[すべての製品]** を選択します。
1. **[すべての製品]** ページで 、 **[+ 試してみる/購入]** を選択します。
1. **[アクティブ化]** ペインの **[AZURE AD PREMIUM P2]** で **[無料試用版]** を選択し、 **[アクティブ化]** を選択します。
1. **[すべての製品]** ページを参照し、 **[Azure Active Directory Premium P2]** を選択します。
1. **[Azure Active Directory Premium P2] \| [ライセンス済みユーザー]** ページで **[+ 割り当て]** を選択します。
1. **[ライセンスの割り当て]** ページで、 **[+ ユーザーとグループの追加]** を選択します。
1. **[ユーザーとグループの追加]** ページで、**管理者** を検索し、結果の一覧から **管理者** アカウントを選択して、 **[選択]** を選択します。
1. **[ライセンスの割り当て]** ページに戻り、 **[確認と割り当て]** を選択し、 **[割り当て]** を選択します。

   > **注**: これは、このラボで後ほど Azure AD パスワード保護を実装するために必要です。

1. **[Azure Active Directory]** ページに戻ります。
1. **[Azure Active Directory]** ページで、 **[パスワードのリセット]** を選びます。
1. **[パスワードのリセット]** ページで、構成を適用するユーザーのスコープを選択できることに注目してください。

   > **注**: パスワード リセット機能は、このラボの後半で必要な構成手順が壊れるので、有効にしないでください。

#### <a name="task-2-enable-password-writeback-in-azure-ad-connect"></a>タスク 2: Azure AD Connect でパスワード ライトバックを有効にする

1. **SEA-ADM1** の **[スタート]** メニューで、 **[Azure AD Connect]** を展開し、 **[Azure AD Connect]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウで、 **[構成]** を選択します。
1. **[追加のタスク]** ページで **[同期オプションのカスタマイズ]** を選んで、 **[次へ]** を選びます。
1. **[Azure AD への接続]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力し、 **[次へ]** を選択します。
1. **[ディレクトリの接続]** ページで、 **[次へ]** を選択します。
1. **[ドメインと OU のフィルタリング]** ページで、 **[次へ]** を選択します。
1. **[オプション機能]** ページで、 **[パスワード ライトバック]** の横にあるチェック ボックスをオンにし、 **[次へ]** を選択します。

   > **注**: Active Directory ユーザーのセルフサービス パスワード リセットには、パスワード ライトバックが必要です。 これにより、Azure AD 内のユーザーによってパスワードが変更され、Active Directory に同期されます。

1. **[構成の準備完了]** ページで、実行するアクションの一覧を確認し、 **[構成]** を選択します。
1. **[構成が完了しました]** ページで、 **[終了]** を選択します。

#### <a name="task-3-enable-pass-through-authentication-in-azure-ad-connect"></a>タスク 3: Azure AD Connect でパススルー認証を有効にする

1. **SEA-ADM1** の **[スタート]** メニューで、 **[Azure AD Connect]** を展開し、 **[Azure AD Connect]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウで、 **[構成]** を選択します。
1. **[追加のタスク]** ページで、 **[ユーザー サインインの変更]** を選択し、 **[次へ]** を選択します。
1. **[Azure AD への接続]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力し、 **[次へ]** を選択します。
1. **[ユーザー サインイン]** ページで、 **[パススルー認証]** を選択します。
1. **[シングル サインオンを有効にする]** チェック ボックスがオンになっていることを確認し、 **[次へ]** を選択します。
1. **[シングル サインオンを有効にする]** ページで、 **[資格情報の入力]** を選択します。
1. **[フォレストの資格情報]** ダイアログ ボックスで、次の資格情報を入力し、 **[OK]** を選択します。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**

1. **[シングル サインオンを有効にする]** ページで、 **[資格情報の入力]** の横に緑色のチェック マークが表示されているのを確認し、 **[次へ]** を選択します。
1. **[構成の準備完了]** ページで、実行するアクションの一覧を確認し、 **[構成]** を選択します。
1. **[構成が完了しました]** ページで、 **[終了]** を選択します。

#### <a name="task-4-verify-pass-through-authentication-in-azure"></a>タスク 4: Azure でパススルー認証を検証する

1. **SEA-ADM1** で、Microsoft Edge ウィンドウに切り替えて Azure portal を表示し、 **[Azure Active Directory]** ページに戻ります。
1. Azure portal 内の **[Azure Active Directory]** ページで、 **[Azure AD Connect]** を選択します。
1. **[Azure AD Connect]** ページの、 **[ユーザー サインイン]** の下に表示されている情報を確認します。
1. **[ユーザー サインイン]** で、 **[シームレス シングル サインオン]** を選択します。
1. **[シームレス シングル サインオン]** ページで、オンプレミスのドメイン名に注目します。
1. Microsoft Edge で、 **[Azure AD Connect]** ページに戻ります。
1. **[Azure AD Connect]** ページの **[ユーザー サインイン]** の下の、 **[パススルー認証]** を選択します。
1. **[パススルー認証]** ページで、 **[認証エージェント]** の下の **SEA-ADM1** サーバー名に注目します。

   > **注**: Azure AD Authentication エージェントを環境内の複数のサーバーにインストールするには、Azure portal の **[パススルー認証]** ページからバイナリーをダウンロードします。 

#### <a name="task-5-install-and-register-the-azure-ad-password-protection-proxy-service-and-dc-agent"></a>タスク 5: Azure AD パスワード保護プロキシ サービスと DC エージェントをインストールして登録する

1. **SEA-ADM1** で Microsoft Edge を起動し、Microsoft ダウンロード Web サイトに移動し、インストーラーをダウンロードできる「**Windows Server Active Directory 用 Azure AD パスワード保護**」のページを参照して、 **[ダウンロード]** を選択します。
1. 「**Windows Server Active Directory 用 Azure AD パスワード保護**」のページで **AzureADPasswordProtectionProxySetup.msi** と **AzureADPasswordProtectionDCAgentSetup.msi** ファイルを選び、 **[次へ]** を選択します。
1. **[Download]** を選択します。
1. **[複数のファイルのダウンロード]** ダイアログ ボックスで、 **[許可]** を選択します。

   > **注**: ドメイン コントローラーではないサーバーにプロキシ サービスをインストールすることをお勧めします。 さらに、プロキシ サービスは、Azure AD Connect エージェントと同じサーバーにはインストールできません。 プロキシ サービスは **SEA-SVR1** に、パスワード保護 DC エージェントは **SEA-DC1** にインストールします。

1. **SEA-ADM1** 上で、**Windows PowerShell** コンソール ウィンドウに切り替えます。
1. **Windows PowerShell** コンソールで、次のコマンドを入力し、Enter キーを押して、インターネットからファイルがダウンロードされたことを示す Zone.Identifier 代替データ ストリームを削除します。

   ```powershell
   Get-ChildItem -Path "$env:USERPROFILE\Downloads" -File | Unblock-File
   ```
1. 次のコマンドを実行して **SEA-SVR1** に **C:\Temp** ディレクトリを作成し、**AzureADPasswordProtectionProxySetup.msi** インストーラーをそのディレクトリにコピーして、インストールを起動します。

   ```powershell
   New-Item -Type Directory -Path '\\SEA-SVR1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionProxySetup.msi" -Destination '\\SEA-SVR1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Temp\AzureADPasswordProtectionProxySetup.msi -ArgumentList '/quiet /log C:\Temp\AzureADPPProxyInstall.log' -Wait }
   ```
1. 次のコマンドを実行して **SEA-DC1** に **C:\Temp** ディレクトリを作成し、**AzureADPasswordProtectionDCAgentSetup.msi** インストーラーをそのディレクトリにコピーし、インストールを起動して、インストールが完了したらドメイン コントローラーを再起動します。

   ```powershell
   New-Item -Type Directory -Path '\\SEA-DC1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionDCAgentSetup.msi" -Destination '\\SEA-DC1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-DC1.contoso.com -ScriptBlock { Start-Process msiexec.exe -ArgumentList '/i C:\Temp\AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn /norestart /log C:\Temp\AzureADPPInstall.log' -Wait }
   Restart-Computer -ComputerName SEA-DC1.contoso.com -Force
   ```
1. 次のコマンドを実行して、インストールによって Azure AD パスワード保護を実装するために必要なサービスが作成されたことを確認します。

   ```powershell
   Get-Service -Computer SEA-SVR1 -Name AzureADPasswordProtectionProxy | fl
   Get-Service -Computer SEA-DC1 -Name AzureADPasswordProtectionDCAgent | fl
   ```

   > **注**: 各サービスの状態が **[実行中]** であることを確認します。

1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して、**SEA-SVR1** への PowerShell リモート処理セッションを開始します。

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1
   ```

1. **[SEA-SVR1]** プロンプトから次のコマンドを入力し、Enter キーを押してプロキシ サービスを Active Directory に登録します (プレースホルダー `<Azure_AD_Global_Admin>` を演習 1 で作成した AD グローバル管理者アカウントの完全修飾ユーザー プリンシパル名に置き換えます)。

   ```powershell
   Register-AzureADPasswordProtectionProxy -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. 指示に従って、別の Microsoft Edge ウィンドウを開き、 **https://microsoft.com/devicelogin** を参照し、ダイアログが表示されたら、PowerShell リモート処理セッションに表示されるメッセージに含まれるコードを入力します。 
1. ダイアログが表示されたら、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントを使用して認証し、 **[続行]** を選択します。
1. PowerShell リモート処理セッションに戻り、次のコマンドを入力し、Enter キーを押して PowerShell リモート処理セッションを終了して **SEA-SVR1** に移動します。

   ```powershell
   Exit-PSsession
   ```

1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して、**SEA-DC1** への PowerShell リモート処理セッションを開始します。

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```

1. **[SEA-DC1]** プロンプトから次のコマンドを入力し、Enter キーを押してプロキシ サービスを Active Directory に登録します (プレースホルダー `<Azure_AD_Global_Admin>` を演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントの完全修飾ユーザー プリンシパル名に置き換えます)。

   ```powershell
   Register-AzureADPasswordProtectionForest -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. 指示に従って、別の Microsoft Edge ウィンドウを開き、 **https://microsoft.com/devicelogin** を参照し、ダイアログが表示されたら、PowerShell リモート処理セッションに表示されるメッセージに含まれるコードを入力します。 
1. ダイアログが表示されたら、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントを使用して認証し、 **[続行]** を選択します。
1. PowerShell リモート処理セッションに戻り、次のコマンドを入力し、Enter キーを押して PowerShell リモート処理セッションを終了して **SEA-DC1** に移動します。

   ```powershell
   Exit-PSsession
   ```

#### <a name="task-6-enable-password-protection-in-azure"></a>タスク 6: Azure でパスワード保護を有効にする

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替え、 **[Azure Active Directory]** ページに戻り、 **[Azure Active Directory]** ページで **[セキュリティ]** を選択します。
1. **[セキュリティ]** ページで、 **[認証方法]** を選択します。
1. **[認証方法]** ページで、 **[パスワード保護]** を選択します。
1. **[パスワード保護]** ページで、 **[カスタム リストを適用する]** のスライダーを **[はい]** に変更します。
1. **[カスタム禁止パスワード リスト]** テキスト ボックスに、次の単語を (1 行に 1 語) 入力します。
 
   - **Contoso**
   - **ロンドン**

   > **注**: 禁止パスワードの一覧は、ご自分の組織に関連する単語である必要があります。

1. **[Windows Server Active Directory でパスワード保護を有効にする]** のスライダーが **[はい]** に設定されていることを確認します。
1. **[モード]** のスライダーが **[監査]** に設定されていることを確認し、 **[保存]** を選択します。

## <a name="exercise-6-cleaning-up"></a>演習 6: クリーンアップ

#### <a name="task-1-uninstall-azure-ad-connect"></a>タスク 1 Azure AD Connect のアンインストール

1. **SEA-ADM1** の **[スタート]** メニューで、 **[コントロール パネル]** を選択します。
1. **[コントロール パネル]** ウィンドウの **[プログラム]** で、 **[プログラムのアンインストール]** をクリックします。
1. **[プログラムのアンインストールまたは変更]** ウィンドウで、 **[Microsoft Azure AD Connect]** を選択し、 **[アンインストール]** を選択します。
1. **[プログラムと機能]** ダイアログ ボックスで **[はい]** をクリックします。
1. **[Azure AD Connect のアンインストール]** ウィンドウで、 **[削除]** を選択します。
1. Azure AD Connect がアンインストールされたら、 **[Azure AD Connect のアンインストール]** ウィンドウで **[終了]** を選択します。

#### <a name="task-2-disable-directory-synchronization-in-azure"></a>タスク 2: Azure でディレクトリ同期を無効にする

1. **SEA-ADM1** 上で、**Windows PowerShell** コンソール ウィンドウに切り替えます。
1. **Windows PowerShell** コンソールで、次のコマンドを入力し、Enter キーを押して、Azure AD 用の Microsoft Online モジュールをインストールします。

   ```powershell
   Install-Module -Name MSOnline
   ```
1. NuGet プロバイダーのインストールを求めるダイアログが表示されたら、「**Y**」と入力し、Enter キーを押します。
1. 信頼されていないリポジトリからモジュールをインストールするように求めるダイアログが表示されたら、「**A**」と入力し、Enter キーを押します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力し、Enter キーを押して、Azure AD の資格情報を変数に格納します。

   ```powershell
   $msolCred = Get-Credential
   ```
1. **[Windows PowerShell 資格情報の要求]** ダイアログボックスに、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントの資格情報を入力し、 **[OK]** を選択します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力し、Enter キーを押して、Azure AD テナントに対する認証を行います。

   ```powershell
   Connect-MsolService -Credential $msolCred
   ```
1. 次のコマンドを入力し、Enter キーを押してディレクトリ同期を無効にします。

   ```powershell
   Set-MsolDirSyncEnabled -EnableDirSync $false
   ```
1. 確認を求めるダイアログが表示されたら、「**Y**」と入力し、Enter キーを押します。
