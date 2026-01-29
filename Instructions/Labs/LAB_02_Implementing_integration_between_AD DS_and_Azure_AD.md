---
lab:
  title: 'ラボ: AD DS と Microsoft Entra ID の統合の実装'
  module: 'Module 2: Implementing Identity in Hybrid Scenarios'
---

# ラボ: AD DS と Microsoft Entra ID の統合の実装

このラボは完了するまで、約 **60** 分かかります。

## シナリオ

Azure リソースへのアクセスの Microsoft Entra ID を使用した認証および承認する作業で生じる管理と監視のオーバーヘッドの懸念事項に対処するために、あなたは、オンプレミスの Active Directory Domain Services (AD DS) と Microsoft Entra ID の統合をテストし、複数のユーザー アカウントの管理に、オンプレミスとクラウドのリソースを組み合わせて使用することに関するビジネス上の懸念に対処できることを検証することにしました。

さらに、あなたは、自分のアプローチが情報セキュリティ チームの懸念事項に対応し、サインイン時間やパスワード ポリシーなどの、Active Directory ユーザーに適用される既存のコントロールを保持することを確認する必要があると考えています。 最後に、オンプレミスの Active Directory のセキュリティを一層強化し、管理オーバーヘッドを最小限に抑える Microsoft Entra ID 統合機能を特定したいと考えています。これには、Windows Server Active Directory 用の Microsoft Entra ID のパスワード保護や、パスワード ライトバックを使用したセルフサービス パスワード リセット (SSPR) が含まれます。

あなたの目標は、オンプレミスの AD DS と Microsoft Entra ID の間でパススルー認証を実装することです。

## 目標

このラボを完了すると、次のことができるようになります。

- カスタム ドメインの追加や検証を含め、Microsoft Entra ID とオンプレミス AD DS との統合の準備を行う。
- IdFix DirSync エラー修復ツールの実行を含め、オンプレミス AD DS と Microsoft Entra ID の統合の準備を行う。
- Microsoft Entra Connect をインストールして構成する。
- 同期プロセスをテストして、AD DS と Microsoft Entra ID の統合を検証する。
- Active Directory に Microsoft Entra ID 統合機能を実装する。これには Windows Server Active Directory 用の Microsoft Entra ID パスワード保護や、パスワード ライトバックを使用した SSPR が含まれます。

## ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-ADM1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。 

> **注**: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-SEA-ADM1** 仮想マシンによって、**SEA-DC1**、**SEA-SVR1**、**SEA-ADM1** のインストールがホストされます。

1. **SEA-ADM1** を選択します。
1. 講師から提供された資格情報を使用してサインインします。

このラボでは、使用可能な VM 環境と Microsoft Entra テナントを使用します。 ラボを開始する前に、Microsoft Entra テナントと、そのテナントのグローバル管理者ロールがあるユーザー アカウントを持っていることを確認してください。

> **重要**: このラボ用に新しい Microsoft Entra テナントを作成する場合は、演習 3 で Microsoft Entra Connect をインストールする前に、ディレクトリ同期を完全に初期化する必要があることに注意してください。 通常、この初期化プロセスは、テナント作成後 10 分から 30 分かかります。 ディレクトリ同期の準備が整うまで、演習 1 と 2 を進めることができます。 Microsoft Entra Connect のインストール時に "このディレクトリではディレクトリ同期が有効になっていますが、まだ実施されていません" というエラーが発生した場合は、さらに 15 分から 30 分待ってから再試行してください。

## 演習 1: AD DS 統合用に Microsoft Entra ID を準備する

### シナリオ

ご自分の Microsoft Entra ID の環境が、オンプレミスの AD DS と統合する準備ができていることを確認する必要があります。 そのため、カスタムの Microsoft Entra ID ドメイン名とグローバル管理者ロールのあるアカウントを作成して検証します。

この演習の主なタスクは次のとおりです。

1. Azure でカスタム ドメイン名を作成する。
1. グローバル管理者ロールを持つユーザーを作成する。
1. グローバル管理者ロールを持つユーザーのパスワードを変更する。

#### タスク 1: Azure でカスタム ドメイン名を作成する。

1. **SEA-ADM1** で Microsoft Edge を起動してから、Azure portal に移動します。
1. インストラクターが提供する資格情報を使用して、Azure portal にサインインします。
1. Azure portal で ** Microsoft Entra ID** に移動します。
1. **Microsoft Entra ID** ページで、左側のナビゲーション ウィンドウの **[管理]** セクションを展開し、**[カスタム ドメイン名]** を選択して、`contoso.com` を追加します。
1. ドメインの検証に使用する DNS レコードの種類を確認し、ドメイン名を確認せずにウィンドウを閉じます。

   > **注**: 一般には、DNS レコードを使用してドメインを確認しますが、このラボでは検証済みドメインを使用する必要はありません。

#### タスク 2: グローバル管理者ロールを持つユーザーを作成する

1. **SEA-ADM1** の Microsoft Edge ウィンドウで **[Microsoft Entra ID]** ページを表示し、**[すべてのユーザー]** ページを参照して、次のプロパティを持つユーザー アカウントを作成します。 

   - ユーザー名: `admin`

   > **注**: **[ユーザー名]** の [ドメイン名] ドロップダウンメニューに、`onmicrosoft.com` で終わる既定のドメイン名が表示されていることを確認します。 このラボの後半でサインインするときに必要になるので、完全なユーザー名 (たとえば、admin@contoso35501731.onmicrosoft.com) をメモしてください。

   - 名前: **admin**
   - ロール: **グローバル管理者**
   - パスワード: 自動生成されたパスワードを使用する

   > **注**: **[パスワードの表示]** オプションを使用して自動生成されたパスワードを表示し、これを記録しておいてください。このラボで後ほど使用します。

#### タスク 3: グローバル管理者ロールを持つユーザーのパスワードを変更する

1. **Azure portal** からサインアウトし、前のタスクで作成したユーザー アカウントでサインインします。 
1. ダイアログが表示されたら、パスワードを変更します。

   > **注**: 使用した複雑なパスワードは、このラボで後ほど使用するので、記録しておきます。

## 演習 2: Microsoft Entra ID 統合のためのオンプレミス AD DS の準備

### シナリオ

あなたは、既存の Active Directory 環境で Microsoft Entra ID との統合の準備ができていることを確認する必要があります。 そのため、IdFix ツールを実行し、Active Directory ユーザーの UPN が Microsoft Entra テナントのカスタム ドメイン名と一致することを確かめます。

この演習の主なタスクは次のとおりです。

1. IdFix をインストールする。
1. IdFix を実行する。

#### タスク 1: IdFix をインストールする

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://github.com/microsoft/idfix` にアクセスします。
1. **Github** ページの **[ClickOnce の起動]** で、**[起動]** を選択します。
1. **[IdFix プライバシーステートメント]** ダイアログ ボックスで、免責事項を確認し、**[OK]** を選択します。

#### タスク 2: IdFix を実行する

1. **IdFix** ウィンドウで、**[クエリ]** を選択します。
1. オンプレミスの Active Directory でオブジェクトの一覧を確認して、**[エラー]** および **[属性]** 列を確認します。 このシナリオでは、**ContosoAdmin** の **displayName** の値が空白で、ツールの推奨される新しい値が**更新**列に表示されます。
1. **IdFix** ウィンドウの、**[アクション] **ドロップダウン メニューで **[編集]** を選択し、**[適用]** を選択すると、推奨される変更が自動的に実装されます。
1. **[保留中の適用]** ダイアログ ボックスで **[はい]** を選択し、IdFix ツールを閉じます。

## 演習 3: Microsoft Entra Connect のダウンロード、インストール、構成

### シナリオ

演習のシナリオ: Microsoft Entra Connect をダウンロードし、**SEA-ADM1** にインストールし、統合の目標に合わせて設定を構成することで、統合を実装する準備ができました。

> **重要**: (新しいテナントを作成した場合は) この演習を進める前に、Microsoft Entra テナントの作成後から十分な時間 (少なくとも 15 分から 30 分) が経過するまで待ちます。 Microsoft Entra Connect をインストールする前に、ディレクトリ同期の初期化が完了している必要があります。 エラー "このディレクトリではディレクトリ同期が有効になっていますが、まだ実施されていません。 ディレクトリ同期の準備が整うまでお待ちください" が表示される場合は、もう少し待ってからインストールを再試行する必要があります。

この演習の主なタスクは次のとおりです。

1. Microsoft Entra Connect をインストールして構成する。

#### タスク 1: Microsoft Entra Connect をインストールして構成する

1. **SEA-ADM1** の場合、Azure portal が表示されている Microsoft Edge ウィンドウの **Microsoft Entra ID** ページで、左側のナビゲーション ウィンドウの **[管理]** セクションを展開し、下にスクロールして **[Microsoft Entra Connect]** を選択し、**Microsoft Entra Connect** ページを参照します。
1. **[Microsoft Entra Connect]** ページから、**[ダウンロード]** を選択します。
1. Microsoft Entra Connect のインストール バイナリをダウンロードし、インストールを開始します。
1. **[Microsoft Entra Connect]** ページで、**[ライセンス条項とプライバシーに関する声明に同意します]** チェックボックスをオンにし、**[続行]** を選択します。
1. **[簡易設定]** ページで、**[簡易設定を使用する]** を選択します。
1. **[Microsoft Entra ID に接続]** ページで、演習 1 で作成した Microsoft Entra ID のグローバル管理者ユーザー アカウントのユーザー名とパスワードを入力します。
1. **[AD DS への接続]** ページで、講師から提供された資格情報を入力します。

1. **[Microsoft Entra ID サインイン構成]** ページで、追加した新しいドメインが Active Directory UPN サフィックスの一覧に含まれていることを確認します。

   > **注**: 指定されたドメイン名は、検証済みドメインである必要はありません。 通常は Microsoft Entra Connect をインストールする前にドメインを検証しますが、このラボではその検証手順は不要とします。

1. **[一部の UPN サフィックスが検証済みドメインに一致していなくても続行する]** チェックボックスをオンにします。
1. **[構成の準備完了]** ページが表示された後、アクションの一覧を確認し、インストールを開始します。

## 演習 4: AD DS と Microsoft Entra ID の統合の検証

### シナリオ

Microsoft Entra Connect をインストールして構成したので、同期のメカニズムを検証する必要があります。 あなたは、同期をトリガーするオンプレミスのユーザー アカウントに変更を加える予定です。 次に、対応する Microsoft Entra ID ユーザー オブジェクトに変更がレプリケートされていることを確認します。

この演習の主なタスクは次のとおりです。

1. Azure portal で同期を検証する。
1. Synchronization Service Manager で同期を検証する。
1. Active Directory でユーザー アカウントを更新する。
1. Active Directory でユーザー アカウントを作成する。
1. 変更を Microsoft Entra ID に同期します。
1. Microsoft Entra ID の変更を検証します。

#### タスク 1: Azure portal で同期を検証する

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。 
1. **[Microsoft Entra Connect]** ページを更新し、**[Active Directory からのプロビジョニング]** の情報を確認します。
1. **[Microsoft Entra ID]** ページから、**[ユーザー]** ページへ移動します。
1. Active Directory から同期されたユーザーの一覧を確認します。

   > **注**: ディレクトリ同期が始まると、Active Directory オブジェクトが Microsoft Entra ID ポータルに表示されるまでに 15 分かかることがあります。

1. **[ユーザー]** ページで、**[グループ]** ページを参照します。
1. Active Directory から同期されたグループの一覧を確認します。

#### タスク 2: Synchronization Service Manager で同期を検証する

1. **SEA-ADM1** の **[スタート]** メニューで **[Microsoft Entra Connect]** を展開し、**[同期サービス]** を選択します。
1. **[Synchronization Service Manager]** ウィンドウの **[操作]** タブで、Active Directory オブジェクトを同期するために実行されたタスクを確認します。
1. **[コネクタ]** タブを選択し、2 つのコネクタに注目します。

   > **注**: コネクタの 1 つは AD DS 用であり、もう 1 つは Microsoft Entra テナント用です。 

1. **[Synchronization Service Manager]** ウィンドウを閉じます。

#### タスク 3: Active Directory でユーザー アカウントを更新する

1. **SEA-ADM1** の **[サーバー マネージャー]** で **Active Directory ユーザーとコンピューター**を開きます。
1. **[Active Directory ユーザーとコンピューター]** で、**Sales** 組織単位 (OU) を展開し、**Sumesh Rajan** のプロパティを開きます。
1. ユーザーのプロパティで、 **[組織]** タブを選択します。
1. **[役職]** テキストボックスに「**マネージャー**」と入力し、**[OK]** を選択します。

#### タスク 4: Active Directory でユーザー アカウントを作成する

- **Sales** OU で次のユーザー を作成します。
   - 名: **Jordan**
   - 姓: **Mitchell**
   - ユーザーログオン名: **Jordan**
   - パスワード: **Pa55w.rd**

#### タスク 5: 変更を Microsoft Entra ID に同期する

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を起動します。
1. **Windows PowerShell** コンソールで、次のコマンドを実行して同期をトリガーします。

   ```powershell
   Start-ADSyncSyncCycle
   ```

   > **注**: 同期サイクルの開始後、Active Directory オブジェクトが Microsoft Entra ID ポータルに表示されるまでに 15 分かかることがあります。

#### タスク 6: Microsoft Entra ID の変更を検証する

1. **SEA-ADM1** で Azure portal を表示している Microsoft Edge ウィンドウに切り替え、**[Microsoft Entra ID]** ページに戻ります。
1. **[Microsoft Entra ID]** ページから、**[ユーザー]** ページへ移動します。
1. **[すべてのユーザー]** ページで、ユーザー **Sumesh** を検索します。
1. ユーザー **Sumesh Rajan** のプロパティ ページを開き、**役職**の属性が Active Directory から同期されたことを確認します。
1. Microsoft Edge で、 **[すべてのユーザー]** ページに戻ります。
1. **[すべてのユーザー]** ページで、ユーザーを **Jordan** 検索します。
1. ユーザー **Jordan Mitchell** の [プロパティ] ページを開き、Active Directory から同期されたユーザー アカウントの属性を確認します。

## 演習 5 AD DS での Microsoft Entra ID 統合機能の実装

### シナリオ

あなたは、オンプレミスの Active Directory のセキュリティをさらに強化し、管理オーバーヘッドを最小限に抑えることができる Microsoft Entra ID 統合機能を特定する必要があると考えています。 また、Windows Server Active Directory 用の Microsoft Entra ID パスワード保護と、パスワード ライトバックを使用するセルフサービス パスワード リセットを実装することも希望しています。

この演習の主なタスクは次のとおりです。

1. Azure でセルフサービス パスワード リセットを有効にする。
1. Microsoft Entra Connect でパスワード ライトバックを有効にする。
1. Microsoft Entra Connect でパススルー認証を有効にする。
1. Azure でパススルー認証を検証する。
1. Microsoft Entra ID パスワード保護プロキシ サービスと DC エージェントをインストールして登録する。
1. Azure でパスワード保護を有効にする。

#### タスク 1: Azure でセルフサービス パスワード リセットを有効にする

1. **SEA-ADM1** の場合、Azure portal が表示されている Microsoft Edge ウィンドウの Microsoft Entra ID ページに移動し、左側のナビゲーション ウィンドウの **[管理]** セクションを展開し、下にスクロールして **[ライセンス]** を選択し、**Microsoft Entra ID P2** 無料試用版のライセンス認証を行います。
   
   > **注**:Microsoft Entra ID P2 無料試用版のライセンス認証を行う際に問題が発生し、支払いオプションの設定を求めるエラー メッセージが表示される場合は、テナントの組織の要件が原因である可能性があります。 このような場合は、P2 ライセンスの機能を特に必要としないラボ演習を続行するか、管理者に問い合わせてサポートを受けることができます。
   
1. 演習 1 で作成した Microsoft Entra ID グローバル管理者ユーザー アカウントに Microsoft Entra ID P2 ライセンスを割り当てます。
   
   > **注**:ライセンスを割り当てるには、**[ライセンス]** ページで、左側のナビゲーション ウィンドウの **[管理]** セクションを展開し、下にスクロールして **[すべての製品]** を選択します。
   
1. Azure portal で、Microsoft Entra ID **[パスワードのリセット]** ページを参照します。
1. **[パスワードのリセット]** ページで、構成を適用するユーザーのスコープを選択できることに注目してください。

   > **注**: パスワード リセット機能は、このラボの後半で必要な構成手順が壊れるので、有効にしないでください。

#### タスク 2: Microsoft Entra Connect でパスワード ライトバックを有効にする

1. **SEA-ADM1** で **Microsoft Entra Connect** を開きます。
1. **Microsoft Entra Connect** ウィンドウで **[構成]** を選択します。
1. **[追加のタスク]** ページで、**[同期オプションのカスタマイズ]** を選択します。
1. **[Microsoft Entra ID に接続]** ページで、演習 1 で作成した Microsoft Entra ID のグローバル管理者ユーザー アカウントのユーザー名とパスワードを入力します。
1. **[オプション機能]** ページで、**[パスワード ライトバック]** を選択します。

   > **注**: Active Directory ユーザーのセルフサービス パスワード リセットには、パスワード ライトバックが必要です。 これにより、Microsoft Entra ID でユーザーが変更したパスワードが Active Directory に同期されます。

1. **[構成の準備完了]** ページで、実行するアクションの一覧を確認し、**[構成]** を選択します。
1. 構成が完了したら、**Microsoft Entra Connect** ウィンドウを閉じます。

#### タスク 3: Microsoft Entra Connect のパススルー認証を有効にする

1. **SEA-ADM1** の **[スタート]** メニューで **[Microsoft Entra Connect]** を展開し、**[Microsoft Entra Connect]** を選択します。
1. **Microsoft Entra Connect** ウィンドウで **[構成]** を選択します。
1. **[追加のタスク]** ページで、**[ユーザー サインインの変更]** を選択します。
1. **[Microsoft Entra ID に接続]** ページで、演習 1 で作成した Microsoft Entra ID のグローバル管理者ユーザー アカウントのユーザー名とパスワードを入力します。
1. **[ユーザー サインイン]** ページで、**[パススルー認証]** を選択します。
1. **[シングル サインオンを有効にする]** チェック ボックスがオンになっていることを確認します。
1. **[シングル サインオンを有効にする]** ページで、**[資格情報の入力]** を選択します。
1. **[フォレストの資格情報]** ダイアログ ボックスで、講師から提供された資格情報を使用して認証します。

2. **[シングル サインオンを有効にする]** ページで、**[資格情報の入力]** の横に緑色のチェック マークが表示されているのを確認します。
3. **[構成の準備完了]** ページで、実行するアクションの一覧を確認し、**[構成]** を選択します。
4. 構成が完了したら、**Microsoft Entra Connect** ウィンドウを閉じます。

#### タスク 4: Azure でパススルー認証を検証する

1. **SEA-ADM1** で Azure portal の **[Microsoft Entra ID]** ページから **[Microsoft Entra Connect]** ページに移動します。
1. **[Microsoft Entra Connect]** ページで **[ユーザー サインイン]** に表示されている情報を確認します。
1. **[ユーザー サインイン]** で、**[シームレス シングル サインオン]** を選択します。
1. **[シームレス シングル サインオン]** ページで、オンプレミスのドメイン名を確認します。
1. **[シームレス シングル サインオン]** ページから、 **[パススルー認証]** ページを参照します。
1. **[パススルー認証]** ページで、 **[認証エージェント]** の下にあるサーバーのリストを確認します。

   > **注**: Microsoft Entra ID 認証エージェントを環境内の複数のサーバーにインストールするには、Azure portal の **[パススルー認証]** ページからエージェントのバイナリをダウンロードします。

#### タスク 5: Microsoft Entra ID のパスワード保護プロキシ サービスと DC エージェントをインストールして登録する

1. **SEA-ADM1** で Microsoft Edge を起動し、Microsoft ダウンロード Web サイトを参照し、インストーラーをダウンロードできる **[Windows Server Active Directory 用の Microsoft Entra ID パスワード保護]** のページを参照して、**[ダウンロード]** を選択します。
1. **AzureADPasswordProtectionProxySetup.exe** と **AzureADPasswordProtectionDCAgentSetup.msi** を **SEA-ADM1** にダウンロードします。

   > **注**: ドメイン コントローラーではないサーバーにプロキシ サービスをインストールすることをお勧めします。 また、プロキシ サービスは、Microsoft Entra Connect エージェントと同じサーバーにはインストールできません。 プロキシ サービスは **SEA-SVR1** に、パスワード保護 DC エージェントは **SEA-DC1** にインストールします。

1. **SEA-ADM1** の **Windows PowerShell** コンソールで、次のコマンドを入力し、インターネットからファイルがダウンロードされたことを示す Zone.Identifier 代替データ ストリームを削除します。

   ```powershell
   Get-ChildItem -Path "$env:USERPROFILE\Downloads -File | Unblock-File
   ```
1. 次のコマンドを実行して **SEA-SVR1** に **C:\Temp** ディレクトリを作成し、**AzureADPasswordProtectionProxySetup.exe** インストーラーをそのディレクトリにコピーして、インストールを起動します。

   ```powershell
   New-Item -Type Directory -Path '\\SEA-SVR1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionProxySetup.exe" -Destination '\\SEA-SVR1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Temp\AzureADPasswordProtectionProxySetup.exe -ArgumentList `/quiet /log C:\Temp\AzureADPPProxyInstall.log` -Wait }
   ```
1. 次のコマンドを実行して **SEA-DC1** に **C:\Temp** ディレクトリを作成し、**AzureADPasswordProtectionDCAgentSetup.msi** インストーラーをそのディレクトリにコピーし、インストールを起動して、インストールが完了したらドメイン コントローラーを再起動します。

   ```powershell
   New-Item -Type Directory -Path '\\SEA-DC1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionDCAgentSetup.msi" -Destination '\\SEA-DC1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-DC1.contoso.com -ScriptBlock { Start-Process msiexec.exe -ArgumentList '/i C:\Temp\AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn /norestart /log C:\Temp\AzureADPPInstall.log' -Wait }
   Restart-Computer -ComputerName SEA-DC1.contoso.com -Force
   ```
1. 次のコマンドを実行して、インストールによって Microsoft Entra ID パスワード保護を実装するために必要なサービスが作成されたことを確認します。

   ```powershell
   Get-Service -Computer SEA-SVR1 -Name AzureADPasswordProtectionProxy | fl
   Get-Service -Computer SEA-DC1 -Name AzureADPasswordProtectionDCAgent | fl
   ```

   > **注**: 各サービスの状態が **[実行中]** であることを確認します。

1. **SEA-SVR1** への PowerShell リモート処理セッションを確立するには、次のコマンドを実行します。

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1
   ```

1. PowerShell リモート処理セッション内で次のコマンドを実行し、プロキシ サービスを Active Directory に登録します (プレースホルダー `<Azure_AD_Global_Admin>` を、演習 1 で作成した Microsoft Entra ID グローバル管理者ユーザー アカウントの完全修飾ユーザー プリンシパル名に置き換えます)。

   ```powershell
   Register-AzureADPasswordProtectionProxy -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. プロンプトに従い、演習 1 で作成した Microsoft Entra ID グローバル管理者ユーザー アカウントを使用して認証を行います。 
1. PowerShell リモート処理セッションを終了します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して、**SEA-DC1** への PowerShell リモート処理セッションを開始します。

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```

1. PowerShell リモート処理セッション内で次のコマンドを実行し、プロキシ サービスを Active Directory に登録します (プレースホルダー `<Azure_AD_Global_Admin>` を、演習 1 で作成した Microsoft Entra ID グローバル管理者ユーザー アカウントの完全修飾ユーザー プリンシパル名に置き換えます)。

   ```powershell
   Register-AzureADPasswordProtectionForest -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. プロンプトに従い、演習 1 で作成した Microsoft Entra ID グローバル管理者ユーザー アカウントを使用して認証を行います。 
1. PowerShell リモート処理セッションを終了します。

#### タスク 6: Azure でパスワード保護を有効にする

1. **SEA-ADM1** で Azure portal が表示されている Microsoft Edge ウィンドウに切り替え、**[Microsoft Entra ID]** ページに戻り **[セキュリティ]** ページに移動します。
1. **[セキュリティ]** ページで、**[認証方法]** を選択します。
1. **[認証方法]** ページで、**[パスワード保護]** を選択します。
1. **[パスワード保護]** ページで、**[カスタム リストの適用]** を有効にします。
1. **[カスタム禁止パスワード リスト]** テキスト ボックスに、次の単語を (1 行に 1 語) 入力します。
 
   - **Contoso**
   - **ロンドン**

   > **注**: 禁止パスワードの一覧は、ご自分の組織に関連する単語である必要があります。

1. **[Windows Server Active Directory でパスワード保護を有効にする]** の設定が有効になっていることを確認します。 
1. **[モード]** が **[監査]** に設定されていることを確認し、変更を保存します。

## 演習 6: クリーンアップ

### シナリオ

オンプレミスの Active Directory から Azure への同期を無効にします。 これには、Microsoft Entra Connect の削除と Azure との同期の無効化が含まれます。

この演習の主なタスクは次のとおりです。

1. Microsoft Entra Connect をアンインストールする。
1. Azure でディレクトリ同期を無効にする。

#### タスク 1: Microsoft Entra Connect をアンインストールする

1. **SEA-ADM1** で、 **[コントロール パネル]** を開きます。
2. **[プログラムのアンインストールまたは変更]** 機能を使用して、**[Microsoft Entra Connect]** をアンインストールします。

#### タスク 2: Azure でディレクトリ同期を無効にする

1. **SEA-ADM1** で、**Windows PowerShell** ウィンドウに切り替えます。
1. **Windows PowerShell** コンソールで次のコマンドを実行して Microsoft Graph PowerShell SDK をインストールします。

   ```powershell
   Install-Module -Name Microsoft.Graph -Force
   ```
1. 次のコマンドを実行して、必要なアクセス許可を持つ Microsoft Entra ID に接続します。

   ```powershell
   Connect-MgGraph -Scopes "Organization.ReadWrite.All"
   ```
1. プロンプトが表示されたら、演習 1 で作成した全体管理者ユーザー アカウントの資格情報を使用してサインインします。
1. 次のコマンドを実行して、Azure でディレクトリ同期を無効にします。

   ```powershell
   $OrgID = (Get-MgOrganization).Id
   $params = @{ onPremisesSyncEnabled = $false }
   Update-MgOrganization -OrganizationId $OrgID -BodyParameter $params
   ```
1. 次のコマンドを実行して、ディレクトリ同期が無効であることを確認します。

   ```powershell
   Get-MgOrganization | Select-Object DisplayName, OnPremisesSyncEnabled
   ```

   > **注**:**OnPremisesSyncEnabled** プロパティは **False** になっているはずです。 同期されたすべてのユーザーがクラウド専用アカウントに完全に変換されるまで、最長 72 時間かかる場合があります。

### 次のモジュールの準備

次のモジュールの準備が完了したら、ラボを終了します。
