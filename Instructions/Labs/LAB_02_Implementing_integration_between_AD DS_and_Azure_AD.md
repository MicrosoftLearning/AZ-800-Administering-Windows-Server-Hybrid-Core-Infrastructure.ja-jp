---
lab:
  title: 'ラボ: AD DS と Azure AD の統合の実装'
  module: 'Module 2: Implementing Identity in Hybrid Scenarios'
---

# <a name="lab-implementing-integration-between-ad-ds-and-azure-ad"></a>ラボ: AD DS と Azure AD の統合の実装

## <a name="scenario"></a>シナリオ

Microsoft Azure Active Directory (Azure AD) を使用して Azure リソースへのアクセスを認証および承認したことで生じる管理と監視のオーバーヘッドについての懸念に対応するために、あなたは、オンプレミスの Active Directory Domain Services (AD DS) と Azure AD の間の統合をテストし、複数のユーザー アカウントの管理に、オンプレミスとクラウド リソースを組み合わせて使用することに関するビジネス上の懸念に対処できることを検証することにしました。

さらに、あなたは、自分のアプローチが情報セキュリティ チームの懸念事項に対応し、サインイン時間やパスワード ポリシーなどの、Active Directory ユーザーに適用される既存のコントロールを保持することを確認する必要があると考えています。 最後に、オンプレミスの Active Directory のセキュリティを一層強化し、管理オーバーヘッドを最小限に抑える Azure AD 統合機能を特定する必要があります。これには、Windows Server Active Directory 用の Azure AD パスワード保護や、パスワード ライトバックを使用したセルフサービス パスワード リセット (SSPR) が含まれます。

あなたの目標は、オンプレミスの AD DS と Azure AD の間でパススルー認証を実装することです。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- カスタム ドメインの追加や検証を含め、Azure AD とオンプレミス AD DS との統合の準備を行う。
- IdFix DirSync エラー修復ツールの実行を含め、オンプレミス AD DS と Azure AD の統合の準備を行う。
- Azure AD Connect をインストールし、構成する。
- 同期プロセスをテストして、AD DS とAzure AD の統合を検証する。
- Active Directory に Azure AD 統合機能を実装する。これには Windows Server Active Directory 用の Azure AD パスワード保護や、パスワード ライトバックを使用した SSPR が含まれます。

## <a name="estimated-time-60-minutes"></a>予想所要時間: 60 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-ADM1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。 

> **注**: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-SEA-ADM1** 仮想マシンによって、**SEA-DC1**、**SEA-SVR1**、**SEA-ADM1** のインストールがホストされます。

1. **SEA-ADM1** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、使用可能な VM 環境と Azure AD テナントを使用します。 ラボを開始する前に、Azure AD テナントと、そのテナントのグローバル管理者ロールがあるユーザー アカウントを持っていることを確認します。

## <a name="exercise-1-preparing-azure-ad-for-ad-ds-integration"></a>演習 1: AD DS 統合のための Azure AD の準備

### <a name="scenario"></a>シナリオ

ご自分の Azure AD の環境で、オンプレミスの AD DS と統合する準備ができている必要があります。 そのため、カスタムの Azure AD ドメイン名とグローバル管理者ロールのあるアカウントを作成して検証します。

この演習の主なタスクは次のとおりです。

1. Azure でカスタム ドメイン名を作成する。
1. グローバル管理者ロールを持つユーザーを作成する。
1. グローバル管理者ロールを持つユーザーのパスワードを変更する。

#### <a name="task-1-create-a-custom-domain-name-in-azure"></a>タスク 1: Azure でカスタム ドメイン名を作成する。

1. ラボの仮想マシン **SEA-ADM1** で Microsoft Edge を起動してから、Azure portal に移動します。
1. インストラクターが提供する資格情報を使用して、Azure portal にサインインします。
1. Azure portal で、**[Azure Active Directory]** に移動します。
1. **[Azure Active Directory]** ページで、**[カスタム ドメイン名]** を選択して `contoso.com` を追加します。
1. ドメインの検証に使用する DNS レコードの種類を確認し、ドメイン名を確認せずにウィンドウを閉じます。

   > **注**: 一般には、DNS レコードを使用してドメインを確認しますが、このラボでは検証済みドメインを使用する必要はありません。

#### <a name="task-2-create-a-user-with-the-global-administrator-role"></a>タスク 2: グローバル管理者ロールを持つユーザーを作成する

1. **SEA-ADM1** の [Microsoft Edge] ウィンドウで、**[Azure Active Directory]** ページを表示し、**[すべてのユーザー]** ページを参照して、次のプロパティを持つユーザー アカウントを作成します。 

   - ユーザー名: **admin**

   > **注**: **[ユーザー名]** の [ドメイン名] ドロップダウンメニューに、`onmicrosoft.com` で終わる既定のドメイン名が表示されていることを確認します。

   - 名前: **admin**
   - 役割: **グローバル管理者**
   - パスワード: 自動生成されたパスワードを使用する

   > **注**: **[パスワードの表示]** オプションを使用して自動生成されたパスワードを表示し、これを記録しておいてください。このラボで後ほど使用します。

#### <a name="task-3-change-the-password-for-the-user-with-the-global-administrator-role"></a>タスク 3: グローバル管理者ロールを持つユーザーのパスワードを変更する

1. **Azure portal** からサインアウトし、前のタスクで作成したユーザー アカウントでサインインします。 
1. ダイアログが表示されたら、パスワードを変更します。

   > **注**: 使用した複雑なパスワードは、このラボで後ほど使用するので、記録しておきます。

## <a name="exercise-2-preparing-on-premises-ad-ds-for-azure-ad-integration"></a>演習 2: Azure AD 統合のためのオンプレミス AD DS の準備

### <a name="scenario"></a>シナリオ

あなたは、既存の Active Directory 環境で Azure AD との統合の準備ができていることを確認する必要があります。 そのため、IdFix ツールを実行し、Active Directory ユーザーの UPN が Azure AD テナントのカスタム ドメイン名と一致することを確保します。

この演習の主なタスクは次のとおりです。

1. IdFix をインストールする。
1. IdFix を実行する。

#### <a name="task-1-install-idfix"></a>タスク 1: IdFix をインストールする

1. **SEA-ADM1** で Microsoft Edge を起動し、**https://github.com/microsoft/idfix** にアクセスします。
1. **Github** ページの **[ClickOnce Launch]** で、**[Launch]** を選択します。
1. ダウンロードした **setrup.exe** を実行します。
1. **Install** をクリックします4. 
1. **[IdFix Privacy Statement]** ダイアログ ボックスで、免責事項を確認し、**[OK]** を選択します。

#### <a name="task-2-run-idfix"></a>タスク 2: IdFix を実行する

1. **IdFix** ウィンドウで、**[Query]** を選択します。
1. **Schema Warning** が表示されたら **Yes** をクリックします
1. オンプレミスの Active Directory でオブジェクトの一覧を確認して、**[Error]** および **[Attribute]** 列を確認します。 このシナリオでは、**ContosoAdmin** の **displayName** の値が空白で、推奨される新しい値が**Update**列に表示されます。
1. **IdFix** ウィンドウの、**[Action] **ドロップダウン メニューで **[EDIT]** を選択し、上のメニューバーにある **[Apply]** を選択すると、推奨される変更が自動的に実装されます。
1. **[Accept All Updates]** ダイアログ ボックスで **[Yes]** を選択します。
1.**ACTION** が **COMPLETED** に代わったことを確認します。
1. IdFix ツールを閉じます。

## <a name="exercise-3-downloading-installing-and-configuring-azure-ad-connect"></a>演習 3: Azure AD Connect のダウンロード、インストール、構成

### <a name="scenario"></a>シナリオ

演習のシナリオ: Azure AD Connect をダウンロードし、**SEA-ADM1** にインストールし、統合の目標に合わせて設定を構成したので、統合を実装する準備ができました。

この演習の主なタスクは次のとおりです。

1. Azure AD Connect をインストールし、構成する。

#### <a name="task-1-install-and-configure-azure-ad-connect"></a>タスク 1: Azure AD Connect のインストールと構成

1. **SEA-ADM1** 上の、Azure portal が表示されている Microsoft Edge ウィンドウで、**[Azure Active Directory]** ページから **[Azure AD Connect]** ページを参照します。
1. 左側のメニューから **[Connect Sync]** をクリックします。
1. **Download Azure AD Connect** をクリックして、Azure AD Connect をダウンロードします。
1. **AzureADConnect.msi** を起動して、インストールを開始します。
1. **[Microsoft Azure Active Directory Connect]** ページで、**[I Agree to license term....]** チェックボックスをオンにし、**[Continue]** を選択します。
1. **[Express Settings]** ページで、**[Use express settings]** を選択します。
1. **[Connect to Azure AD]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力します。
1. **[Connect to AD DS]** ページで、次の資格情報を入力します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. **[Azure AD Sign-in configuration]** ページで、追加した新しいドメイン(contoso.com)が Active Directory UPN サフィックスの一覧に含まれていることを確認します。

   > **注**: 指定されたドメイン名は、検証済みドメインである必要はありません。 通常は Azure AD Connect をインストールする前にドメインを検証しますが、このラボでは検証手順は必要ありません。

1. **[Continue without matching all UPN suggixes to verified domains/一部の UPN サフィックスが検証済みドメインに一致していなくても続行する]** チェックボックスをオンにして **Next** をクリックします。
1. **[Ready to configure/構成の準備完了]** ページが表示された後、アクションの一覧を確認し、インストールを開始します。

## <a name="exercise-4-verifying-integration-between-ad-ds-and-azure-ad"></a>演習 4: AD DS と Azure AD の統合の検証

### <a name="scenario"></a>シナリオ

あなたは、Azure AD Connect をインストールして構成したので、同期のメカニズムを確認する必要があります。 あなたは、同期をトリガーするオンプレミスのユーザー アカウントに変更を加える予定です。 次に、対応する Azure AD ユーザー オブジェクトに変更がレプリケートされていることを確認します。

この演習の主なタスクは次のとおりです。

1. Azure portal で同期を検証する。
1. Synchronization Service Manager で同期を検証する。
1. Active Directory でユーザー アカウントを更新する。
1. Active Directory でユーザー アカウントを作成する。
1. Azure AD に対する変更を同期する。
1. Azure AD での変更を検証する。

#### <a name="task-1-verify-synchronization-in-the-azure-portal"></a>タスク 1: Azure portal で同期を検証する

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。 
1. **Azure AD Connect** ページを更新し、**[Provision From Active Directory/Active Directory からのプロビジョニング]** 情報を確認し、**Sync Status** が **Enabled** になっていることを確認します。
1. **[Azure Active Directory]** の管理ポータルから **[Users/ユーザー]** ページを参照します。
1. Active Directory から同期されたユーザーの一覧を確認します。同期されたユーザーは、**On-premise sync enabled** が **Yes** に設定されています。

   > **注**: ディレクトリ同期が開始されると、Active Directory オブジェクトが Azure AD ポータルに表示されるまでに 15 分かかることがあります。

1. **[ユーザー]** ページで、**[グループ]** ページを参照します。
1. Active Directory から同期されたグループの一覧を確認します。

#### <a name="task-2-verify-synchronization-in-the-synchronization-service-manager"></a>タスク 2: Synchronization Service Manager で同期を検証する

1. **SEA-ADM1** の **[スタート]** メニューで、**[Azure AD Connect]** を展開し、**[Synchronization Service/同期サービス]** を選択します。
1. **[Synchronization Service Manager]** ウィンドウの **[Operations]** タブで、Active Directory オブジェクトを同期するために実行されたタスクを確認します。
1. **[Connectors/コネクタ]** タブを選択し、2 つのコネクタに注目します。

   > **注**: 1 つのコネクタが AD DS 用であり、もう 1 つは Azure AD テナント用です。 

1. **[Synchronization Service Manager]** ウィンドウを閉じます。

#### <a name="task-3-update-a-user-account-in-active-directory"></a>タスク 3: Active Directory でユーザー アカウントを更新する

1. **SEA-ADM1** の **[サーバー マネージャー]** の **Tools** でメニューから **Active Directory Usera and Computers**を開きます。
1. **[Active Directory Users and Computers]** で、**Sales** 組織単位 (OU) を展開し、**Sumesh Rajan** のプロパティを開きます。
1. ユーザーのプロパティで、 **[Organizations/組織]** タブを選択します。
1. **[Job Title/役職]** テキストボックスに「**Manager**」と入力し、**[OK]** を選択します。

#### <a name="task-4-create-a-user-account-in-active-directory"></a>タスク 4: Active Directory でユーザー アカウントを作成する

- **Sales** OU で次のユーザー を作成します。
   - 名: **Jordan**
   - 姓: **Mitchell**
   - ユーザーログオン名: **Jordan**
   - パスワード: **Pa55w.rd**
   - Password never expires を **オン** にする

#### <a name="task-5-sync-changes-to-azure-ad"></a>タスク 5: Azure AD に対する変更を同期する

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を起動します。
1. **Windows PowerShell** コンソールで、次のコマンドを実行して同期をトリガーします。

   ```powershell
   Start-ADSyncSyncCycle
   ```

   > **注**: 同期サイクルの開始後、Active Directory オブジェクトが Azure AD ポータルに表示されるまでに 15 分かかることがあります。

#### <a name="task-6-verify-changes-in-azure-ad"></a>タスク 6: Azure AD での変更を検証する

1. **SEA-ADM1** で、Microsoft Edge ウィンドウに切り替えて Azure portal を表示し、**[Azure Active Directory]** ページに戻ります。
1. **[Azure Active Directory]** ページから **[Users/ユーザー]** ページを参照します。
1. ユーザー **Sumesh** を検索します。
1. ユーザー **Sumesh Rajan** の Properties ページを開き、**役職**の属性が Active Directory から同期されたことを確認します。
1. Microsoft Edge で、 **[すべてのユーザー]** ページに戻ります。
1. **[すべてのユーザー]** ページで、ユーザーを **Jordan** 検索します。
1. ユーザー **Jordan Mitchell** の [プロパティ] ページを開き、Active Directory から同期されたユーザー アカウントの属性を確認します。

## <a name="exercise-5-implementing-azure-ad-integration-features-in-ad-ds"></a>演習 5 AD DS での Azure AD 統合機能の実装

### <a name="scenario"></a>シナリオ

あなたは、オンプレミスの Azure Active Directory のセキュリティをさらに強化し、管理オーバーヘッドを最小限に抑えることができる Azure AD 統合機能を特定する必要があると考えています。 また、Windows Server Active Directory 用の Azure AD パスワード保護と、パスワード ライトバックを使用するセルフサービス パスワード リセットを実装することも希望しています。

この演習の主なタスクは次のとおりです。

1. Azure でセルフサービス パスワード リセットを有効にする。
1. Azure AD Connect でパスワード ライトバックを有効にする。
1. Azure AD Connect でパススルー認証を有効にする。
1. Azure でパススルー認証を検証する。
1. Azure AD パスワード保護プロキシ サービスと DC エージェントをインストールして登録する。
1. Azure でパスワード保護を有効にする。

#### <a name="task-1-enable-self-service-password-reset-in-azure"></a>タスク 1: Azure でセルフサービス パスワード リセットを有効にする

1. **SEA-ADM1** の、Azure portal が表示されている Microsoft Edge ウィンドウで、Azure AD の **[License]** ページを開きます。
1. **All Products** を開き、**Try/Buy** をクリックします。
1. **[Azure AD Premium P2]** 無料試用版を **Activate** します。 
1. **Azure Active Directory Premium P2** が表示されるまで、画面をリフレッシュします。
1. **Azure Active Directory Premium P2** をクリックして、**「+Assign」** をクリックし、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウント(**admin@** )に Azure AD Premium P2 ライセンスを割り当てます。
1. ライセンスを有効にするために、admin ユーザーでサインインしなおします。
3. Azure portal に接続し、Azure AD 管理画面のトップページにいき、 **[Password Reset/パスワードのリセット]** ページを参照します。
4. **[Self-service password reset/セルフサービスパスワードのリセット]** ページで、構成を適用するユーザーのスコープを選択できることに注目してください。

   > **注**: パスワード リセット機能は、このラボの後半で必要な構成手順が壊れるので、ここでは何もしないででください。

#### <a name="task-2-enable-password-writeback-in-azure-ad-connect"></a>タスク 2: Azure AD Connect でパスワード ライトバックを有効にする

1. **SEA-ADM1** の **スタート** メニューで **Azure AD Connect** を展開し、**Azure AD Connect** を起動します。
1. **[Welcome to Azure AD Connect]** ウィンドウで、**[Configure]** を選択します。
1. **[Additional taksks/追加のタスク]** ページで、**[Customize synchronization options/同期オプションのカスタマイズ]** を選択して **Next** をクリックします。
1. **[Connect Azure AD]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力して **Next** をクリックします。
1. **Connect your directories** ページでは何もせずに **Next** をクリックします
1. **Domain and OU filtering** ページでは何もせずに **Next** をクリックします。
1. **[Optional features/オプション機能]** ページで、**[Password writeback/パスワード ライトバック]** を選択し、**Next** をクリックします。

   > **注**: Active Directory ユーザーのセルフサービス パスワード リセットには、パスワード ライトバックが必要です。 これにより、Azure AD 内のユーザーによってパスワードが変更され、オンプレミス側の Active Directory に同期されます。

1. **[Ready to configure/構成の準備完了]** ページで、実行するアクションの一覧を確認し、**[Configure]** を選択します。
1. 構成が完了したら、**Exit** をクリックして、**[Azure Active Directory Connect]** ウィンドウを閉じます。

#### <a name="task-3-enable-pass-through-authentication-in-azure-ad-connect"></a>タスク 3: Azure AD Connect でパススルー認証を有効にする

1. **SEA-ADM1** の **[スタート]** メニューで、**[Azure AD Connect]** を展開し、**[Azure AD Connect]** を選択します。

  > **Cannot change configuration** が表示された場合は、前のステップの構成が完了するまで少し待ってからやり直してください

1. **[Welcome to Azure AD Connect]** ウィンドウで、**[Configure]** を選択します。
4. **[Additional Tasks/追加のタスク]** ページで、**[Change user sign-in/ユーザー サインインの変更]** を選択して **Next** をクリックします。
5. **[Azure AD への接続]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力して **Next** をクリックします。
6. **[User sign-in/ユーザー サインイン]** ページで、**[Pass-through authentication/パススルー認証]** を選択します。
7. **[Enable single sign-on/シングル サインオンを有効にする]** チェック ボックスがオンになっていることを確認します。
8. **[Enable single sign-on/シングル サインオンを有効にする]** ページで、**[Enter credentials/資格情報の入力]** を選択します。
9. **[Forest Credentials/フォレストの資格情報]** ダイアログ ボックスで、次の資格情報を使用して認証します。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**

1. **[Enable single sign-on/シングル サインオンを有効にする]** ページで、**[Enter credentials/資格情報の入力]** の横に緑色のチェック マークが表示されているのを確認し、**Next** をクリックます。
1. **[Ready to configure/構成の準備完了]** ページで、実行するアクションの一覧を確認し、**[Configure/構成]** を選択します。
1. 構成が完了したら、**Exit** をクリックして **[Azure Active Directory Connect]** ウィンドウを閉じます。

#### <a name="task-4-verify-pass-through-authentication-in-azure"></a>タスク 4: Azure でパススルー認証を検証する

1. **SEA-ADM1** の、Azure portal で **[Azure Active Directory]** ページから **[Azure AD Connect]** ページを参照します。
1. **[Azure AD Connect]** ページの、**Connect Sync** ページを開き、**[USER SIGN-IN/ユーザー サインイン]** の下で、**Seamless single sign-on** と **Pass-through authentication** が **Enabled** に設定されていることを確認します。
1. **[Seamless single sign-on/シームレス シングル サインオン]** を選択します。
1. **オンプレミスのドメイン名として **contoso.com** が表示されていることを確認します。
1. 前のページに戻り、**[Pass-through authentication]** をクリックします。
3. **[Authentication Agent/認証エージェント]** の下にあるサーバーのリストに、SEA-ADM1 が表示されていることを確認します。

   > **注**: Azure AD Authentication エージェントを環境内の複数のサーバーにインストールするには、Azure portal の **[パススルー認証]** ページからバイナリーをダウンロードします。

#### <a name="task-5-install-and-register-the-azure-ad-password-protection-proxy-service-and-dc-agent"></a>タスク 5: Azure AD パスワード保護プロキシ サービスと DC エージェントをインストールして登録する

1. **SEA-ADM1** で Microsoft Edge を起動し、https://www.microsoft.com/en-us/download/details.aspx?id=57071 から「**Windows Server Active Directory 用 Azure AD パスワード保護**」を表示します。
1. **AzureADPasswordProtectionProxySetup.exe** と **AzureADPasswordProtectionDCAgentSetup.msi** を **SEA-ADM1** にダウンロードします。

   > **注**: ドメイン コントローラーではないサーバーにプロキシ サービスをインストールすることをお勧めします。 さらに、**プロキシ サービスは、Azure AD Connect エージェントと同じサーバーにはインストールできません。** プロキシ サービスは **SEA-SVR1** に、パスワード保護 DC エージェントは **SEA-DC1** にインストールします。

1. **SEA-ADM1** の **Windows PowerShell** コンソールで、次のコマンドを入力し、インターネットからファイルがダウンロードされたことを示す Zone.Identifier 代替データ ストリームを削除します。

   ```powershell
   Get-ChildItem -Path "$env:USERPROFILE\Downloads" -File | Unblock-File
   ```
1. 次のコマンドを実行して **SEA-SVR1** に **C:\Temp** ディレクトリを作成し、**AzureADPasswordProtectionProxySetup.exe** インストーラーをそのディレクトリにコピーして、インストールを起動します。

   ```powershell
   New-Item -Type Directory -Path '\\SEA-SVR1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionProxySetup.exe" -Destination '\\SEA-SVR1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Temp\AzureADPasswordProtectionProxySetup.exe -ArgumentList "/quiet /log C:\Temp\AzureADPPProxyInstall.log" -Wait }
   ```
1. 次のコマンドを実行して **SEA-DC1** に **C:\Temp** ディレクトリを作成し、**AzureADPasswordProtectionDCAgentSetup.msi** インストーラーをそのディレクトリにコピーし、インストールを起動して、インストールが完了したらドメイン コントローラーを再起動します。

   ```powershell
   New-Item -Type Directory -Path '\\SEA-DC1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionDCAgentSetup.msi" -Destination '\\SEA-DC1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-DC1.contoso.com -ScriptBlock { Start-Process msiexec.exe -ArgumentList "/i C:\Temp\AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn /norestart /log C:\Temp\AzureADPPInstall.log" -Wait }
   Restart-Computer -ComputerName SEA-DC1.contoso.com -Force
   ```
1. 次のコマンドを実行して、インストールによって Azure AD パスワード保護を実装するために必要なサービスが作成されたことを確認します。

   ```powershell
   Get-Service -Computer SEA-SVR1 -Name AzureADPasswordProtectionProxy | fl
   Get-Service -Computer SEA-DC1 -Name AzureADPasswordProtectionDCAgent | fl
   ```

   > **注**: 各サービスの **Status** が **[Running]** であることを確認します。

1. **SEA-SVR1** への PowerShell リモート処理セッションを確立するには、次のコマンドを実行します。

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1
   ```

1. PowerShell リモート処理セッション内で次のコマンドを入力し、プロキシ サービスを Active Directory に登録します (プレースホルダー `<Azure_AD_Global_Admin>` を演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントの完全修飾ユーザー プリンシパル名に置き換えます)。

   ```powershell
   Register-AzureADPasswordProtectionProxy -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. プロンプトに従い、**https://microsoft.com/devicelogin/** を別のタブで表示して、画面に表示されているコードを入力した後、admin@... で認証します。 
1. **Exit** を入力して、PowerShell リモート管理セッションを終了します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して、**SEA-DC1** への PowerShell リモート処理セッションを開始します。

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```

1. PowerShell リモート処理セッション内で次のコマンドを入力し、プロキシ サービスを Active Directory に登録します (プレースホルダー `<Azure_AD_Global_Admin>` を演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントの完全修飾ユーザー プリンシパル名に置き換えます)。

   ```powershell
   Register-AzureADPasswordProtectionForest -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. プロンプトに従い、**https://microsoft.com/devicelogin/** を別のタブで表示して、画面に表示されているコードを入力した後、admin@... で認証します。 
1. **Exit** コマンドでPowerShell リモート管理セッションを終了します。

#### <a name="task-6-enable-password-protection-in-azure"></a>タスク 6: Azure でパスワード保護を有効にする

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替え、**[Azure Active Directory]** 管理ポータルのトップページに戻り、**[Security/セキュリティ]** ページを閲覧します。
1. **[Security/セキュリティ]** ページで、**[Authentication Method/認証方法]** を選択します。
1. **[Password Protection/パスワード保護]** を選択します。
1. **[Enforce custom list]** を **Yes** にします。
1. **[カスタム禁止パスワード リスト]** テキスト ボックスに、次の単語を (1 行に 1 語) 入力します。
 
   - **Contoso**
   L **London**

   > **注**: 禁止パスワードの一覧は、ご自分の組織に関連する単語である必要があります。

1. **[Password protection for Windows Servver Active Directory]** で **Enable Password protection on Windows Server Active Directory**が **Yes** になっていることを確認します。 
1. **[モード]** が **[監査]** に設定されていることを確認し、**Save** をクリックして保存します。

## <a name="exercise-6-cleaning-up"></a>演習 6: クリーンアップ

### <a name="scenario"></a>シナリオ

オンプレミスの Active Directory から Azure への同期を無効にします。 これには、Azure AD Connect の削除と Azure との同期の無効化が含まれます。

この演習の主なタスクは次のとおりです。

1. Azure AD Connect をアンインストールする。
1. Azure でディレクトリ同期を無効にする。

#### <a name="task-1-uninstall-azure-ad-connect"></a>タスク 1 Azure AD Connect のアンインストール

1. **SEA-ADM1** で、**[コントロール パネル]** を開きます。
1. **[プログラムのアンインストールまたは変更]** 機能を使用して、**[Microsoft Azure AD Connect]** をアンインストールします。

#### <a name="task-2-disable-directory-synchronization-in-azure"></a>タスク 2: Azure でディレクトリ同期を無効にする

1. **SEA-ADM1** で、**Windows PowerShell** ウィンドウに切り替えます。
1. **Windows PowerShell** コンソールで、次のコマンドを実行し、Azure AD 用の Microsoft Online モジュールをインストールします。プロンプトが表示されたら「A」で回答してください。

   ```powershell
   Install-Module -Name MSOnline
   ```
1. 次のコマンドを実行して、資格情報を Azure に提供します。

   ```powershell
   $msolcred=Get-Credential
   ```
1. ダイアログが表示されたら、**[Windows PowerShell 資格情報要求]** ダイアログ ボックスで、演習 1 で作成した管理者アカウントの資格情報を入力します。
1. 次のコマンドを実行して Azure に接続します。

   ```powershell
   Connect-MsolService -Credential $msolcred
   ```
1. 次のコマンドを実行して、Azure でディレクトリ同期を無効にします。プロンプトが表示されたら「Y」で回答します。

   ```powershell
   Set-MsolDirSyncEnabled -EnableDirSync $false
   ```

### <a name="prepare-for-the-next-module"></a>次のモジュールの準備

次のモジュールの準備が完了したら、ラボを終了します。
