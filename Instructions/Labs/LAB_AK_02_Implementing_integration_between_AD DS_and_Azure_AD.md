---
lab:
  title: 'ラボ: AD DS と Microsoft Entra ID の統合の実装'
  type: Answer Key
  module: 'Module 2: Implementing Identity in Hybrid Scenarios'
---

# ラボの回答キー: AD DS と Microsoft Entra ID の統合の実装

このラボは完了するまで、約 **60** 分かかります。

## ラボのセットアップ

1. **SEA-ADM1** に接続し、必要に応じて、講師から提供された資格情報でサインインします。
1. **SEA-ADM1** で Microsoft Edge を起動し、Azure portal を参照して、Azure 資格情報で認証します。
1. Azure portal で ** Microsoft Entra ID** に移動します。
1. 概要で **[テナントの管理]** を選択し、**[+ 作成]** を選択します。
1. Microsoft Entra ID を選択し、**[次へ: 構成]** を選択します。
1. [組織名] に、**「Contoso Organization」** などの一意の名前を入力します。
1. [イニシャル ドメイン名] に、**「Contoso35501731」** などの一意の名前を入力します。
1. **[確認および作成]** を選択し、次に **[作成]** を選択します。

   > **注**: Captcha を完了し、[送信] をクリックして、テナントの作成が完了するまで待ちます。

1. **「テナントの作成が成功しました。こちらをクリックして新しいテナントに移動してください (テナントリンクはこちら)。」** という通知を受けたら、メッセージにある作成されたテナントを選択します。

   > **重要**: 新しい Microsoft Entra テナントを作成した後、Microsoft Entra Connect をインストールする前に、ディレクトリ同期が完全に初期化されるまで待つ必要があります。 通常、このプロセスには 10 分から 30 分かかります。 すぐに演習 3 に進み、Microsoft Entra Connect をインストールしようとすると、次のエラーが発生する可能性があります:"このディレクトリではディレクトリ同期が有効になっていますが、まだ実施されていません。 ディレクトリ同期の準備が整うまでお待ちください。" この問題が発生した場合は、少なくとも 15 分から 30 分待ってから Microsoft Entra Connect を再インストールしてください。 待っている間に、演習 1 と 2 を進めることができます。

## 演習 1: AD DS 統合用に Microsoft Entra ID を準備する

#### タスク 1: Azure でカスタム ドメインを作成する

1. **SEA-ADM1** に接続し、必要に応じて、講師から提供された資格情報でサインインします。
1. **SEA-ADM1** で Microsoft Edge を起動し、Azure portal を参照して、Azure 資格情報で認証します。
1. Azure portal で ** Microsoft Entra ID** に移動します。
1. **Microsoft Entra ID** ページで、左側のナビゲーション ウィンドウの **[管理]** セクションを展開し、**[カスタム ドメイン名]** を選択します。
1. **[カスタムドメイン名]** ページで、**[カスタムドメインの追加]** を選択します。
1. **[カスタム ドメイン名]** ペインで、**[カスタム ドメイン名]** テキスト ボックスに「`contoso.com`」と入力し、**[ドメインの追加] **を選択します。
1. カスタム ドメイン名 **[contoso.com]** のページで、使用するドメイン ネーム システム (DNS) レコードの種類を確認してドメインを検証します。
1. ドメイン名を確認せずにウィンドウを閉じます。

   > **注**: 一般には、DNS レコードを使用してドメインを確認しますが、このラボでは検証済みドメインを使用する必要はありません。

#### タスク 2: グローバル管理者ロールを持つユーザーを作成する

1. **SEA-ADM1** で Azure portal の **[Microsoft Entra ID]** ページで、**[ユーザー]** を選択します。
1. **[すべてのユーザー]** ページで **[+ 新しいユーザー]** を選択し、ドロップダウン リストから **[新しいユーザーの作成]** を選択します。
1. **[新しいユーザーの作成]** ページの **[ID]** で、**[ユーザー プリンシパル名]** と **[表示名]** のテキスト ボックスに「**admin**」と入力します。

   > **注**: **[ユーザー プリンシパル名]** の [ドメイン名] ドロップダウン メニューに、`onmicrosoft.com` で終わる既定のドメイン名が表示されていることを確認します。 このラボの後半でサインインするときに必要になるので、完全なユーザー名 (たとえば、admin@contoso35501731.onmicrosoft.com) をメモしてください。

1. **[パスワード]** で **[コピー] アイコン**を選択し、このラボの後半で使用するパスワードを記録します。
1. **[次へ: プロパティ]** を選択します。
1. [プロパティ] タブの **[使用場所]** で **[米国]** を選択し、**[次へ: 割り当て]** を選択します。
1. **[割り当て]** タブで **[+ ロールの追加]** を選択し、**[ディレクトリ ロール]** の一覧から **[グローバル管理者]** を選択して **[選択]** ボタンを選択します。
1. **[新しいユーザー]** ページで **[確認および作成]** を選択し、**[作成]** を選択します。

#### タスク 3: グローバル管理者ロールを持つユーザーのパスワードを変更する

1. Azure portal の右上でご自分のユーザー アカウントを選択して、**[サインアウト]** を選択します。
1. **[アカウントの選択]** ページで、**[別のアカウントを使用する]** を選択します。
1. **[サインイン]** ページで、前に作成したユーザー アカウントの完全修飾ユーザー名を入力し、**[次へ]** を選択します。
1. 現在のパスワードについては、前の手順で書き留めたパスワードを使用します。
1. 複雑なパスワードを 2 回入力し、**[サインイン]** を選択します。

   > **注**: 使用した複雑なパスワードは、このラボで後ほど使用するので、記録しておきます。

## 演習 2: Microsoft Entra ID 統合のためのオンプレミス AD DS の準備

#### タスク 1: IdFix をインストールする

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://github.com/microsoft/idfix` を参照します。
1. **Github** ページの **[ClickOnce の起動]** で、**[起動]** を選択します。
1. ステータスバーで、**[ファイルを開く]** を選択します。
1. **[アプリケーションのインストール-セキュリティの警告]** ダイアログ ボックスで、**[インストール]** を選択します。
1. **[IdFix プライバシーステートメント]** ダイアログ ボックスで、免責事項を確認し、**[OK]** を選択します。

#### タスク 2: IdFix を実行する

1. **IdFix** ウィンドウで、**[クエリ]** を選択します。
1. **[スキーマの警告]** ダイアログ ボックスが表示されたら、**[はい]** を選択します。
1. Active Directory でオブジェクトの一覧を確認し、**[エラー]** および **[属性]** 列に注目します。 このシナリオでは、**ContosoAdmin** の **displayName** の値が空白で、ツールの推奨される新しい値が**更新**列に表示されます。
1. **IdFix** ウィンドウで、**[アクション] **ドロップダウン メニューから **[編集]** を選択し、**[適用]** を選択すると、推奨される変更が自動的に実装されます。
1. **[保留の適用]** ダイアログ ボックスで、**[はい]** を選択します。
1. IdFix ツールを閉じます。

## 演習 3: Microsoft Entra Connect のダウンロード、インストール、構成

   > **重要**: ラボのセットアップ セクションで新しい Microsoft Entra テナントを作成した場合は、テナントの作成から少なくとも 15 分から 30 分経過してからこの演習を続行してください。 新しいテナントでディレクトリ同期の初期化が完了している必要があります。 エラー "このディレクトリではディレクトリ同期が有効になっていますが、まだ実施されていません。 ディレクトリ同期の準備が整うまでお待ちください" が Microsoft Entra Connect のインストール時に表示された場合は、さらに 15 分から 30 分待ってから再試行してください。

#### タスク 1: Microsoft Entra Connect をインストールして構成する

   > **注**: Microsoft Entra Connect アプリケーションをダウンロードしても、アプリケーションには前の名前 (Azure AD Connect) が表示されます。

1. **SEA-ADM1** で Azure portal を表示している Microsoft Edge ウィンドウで、**[Microsoft Entra ID]** に移動します。
1. **Microsoft Entra ID** ページで、左側のナビゲーション ウィンドウの **[管理]** セクションを展開し、下にスクロールして **[Microsoft Entra Connect]** を選択します。
1. **Microsoft Entra Connect | [作業の開始]** ページで、**[Connect 同期]** を選択します。
1. **Microsoft Entra Connect | [Connect 同期]** ページで、**[Microsoft Entra Connect のダウンロード]** を選択します。
1. 新しく開いたページの **[Azure AD Connect V2]** で、**[ダウンロード]** を選択します。
1. ステータスバーで、**[ファイルを開く]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ページで、**[ライセンス条項とプライバシーに関する声明に同意します]** チェックボックスをオンにし、**[続行]** を選択します。
1. **[簡易設定]** ページで、**[簡易設定を使用する]** を選択します。
1. **[Azure AD への接続]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力し、**[次へ]** を選択します。
1. [アカウントへのサインイン] ページで、新しく作成したユーザーでサインインし、**[次へ]** を選択します。

1. **[AD DS への接続]** ページで、講師から提供された資格情報を入力します。
1. **[Azure AD のサインイン構成]** ページで、追加した新しいドメインが Active Directory UPN サフィックスの一覧に表示されていますが、その状態は **[未検証]** と表示されていることに注目してください。

   > **注**: 指定されたドメイン名は、検証済みドメインである必要はありません。 通常は Microsoft Entra Connect をインストールする前にドメインを検証しますが、このラボではその検証手順は不要とします。

1. **[一部の UPN サフィックスが検証済みドメインに一致していなくても続行する]** チェックボックスをオンにし、**[次へ]** を選択します。
1. **[構成の準備完了]** ページで、アクションの一覧を確認し、**[インストール]** を選択します。
1. **[構成が完了しました]** ページで、 **[終了]** を選択します。

## 演習 4: AD DS と Microsoft Entra ID の統合の検証

#### タスク 1: Azure portal で同期を検証する

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。 
1. **[Microsoft Entra Connect]** ページを更新し、**[Active Directory からのプロビジョニング]** の情報を確認します。
1. **[Microsoft Entra ID]** ページで、**[ユーザー]** を選択します。
1. ユーザー一覧には Active Directory から同期されたユーザーが含まれていることにご注意ください。

   > **注**: ディレクトリ同期が始まると、Active Directory オブジェクトが Microsoft Entra ID ポータルに表示されるまでに 15 分かかることがあります。

1. Microsoft Edge で、**[Microsoft Entra ID]** ページに戻ります。
1. **[Microsoft Entra ID]** ページで、**[グループ]** を選択します。
1. Active Directory から同期されたグループの一覧に注目します。 


#### タスク 2: Active Directory でユーザー アカウントを更新する

1. **SEA-ADM1** で、サーバー マネージャーの **[ツール]** メニューから **[Active Directory ユーザーとコンピューター]** を選択します。
1. **[Active Directory ユーザーとコンピューター]** で、**Sales** 組織単位 (OU) を展開し、**Sumesh Rajan** のプロパティを開きます。
1. ユーザーのプロパティで、 **[組織]** タブを選択します。
1. **[役職]** テキストボックスに「**マネージャー**」と入力し、**[OK]** を選択します。

#### タスク 3: Active Directory でユーザー アカウントを作成する

1. **[Active Directory ユーザーとコンピューター]** で、**Sales** OU のコンテキスト メニューを右クリックするか、またはこれにアクセスして、**[新規作成]**、**[ユーザー]** の順に選択します。
1. **[新しいオブジェクト - ユーザー]** ウィンドウで、各フィールドについて次のユーザーの詳細を入力し、**[次へ]** を選択します。

   - 名: **Jordan**
   - 姓: **Mitchell**
   - ユーザーログオン名: **Jordan**

1. **"パスワード"** および **"パスワードの確認"** フィールドに「**Pa55w.rd**」と入力し、**[次へ]** を選択します。
1. **[完了]** を選択します。

#### タスク 4: 変更を Microsoft Entra ID に同期する

1. **SEA-ADM1** の **[スタート]** メニューで、**Windows PowerShell** を選択します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力しから Enter キーを押して同期をトリガーします。

   ```powershell
   Start-ADSyncSyncCycle
   ```

   > **注**: 同期サイクルが開始されると、Active Directory オブジェクトが Microsoft Entra ID ポータルに表示されるまでに 15 分かかることがあります。

#### タスク 5: Microsoft Entra ID の変更を確認する

1. **SEA-ADM1** で Azure portal を表示している Microsoft Edge ウィンドウに切り替え、**[Microsoft Entra ID]** ページに戻ります。
1. **[Microsoft Entra ID]** ページで、**[ユーザー]** を選択します。
1. **[すべてのユーザー]** ページで、ユーザー **Sumesh** を検索します。
1. ユーザー **Sumesh Rajan** のプロパティ ページを開き、**役職**の属性が Active Directory から同期されたことを確認します。
1. Microsoft Edge で、 **[すべてのユーザー]** ページに戻ります。
1. **[すべてのユーザー]** ページで、ユーザーを **Jordan** 検索します。
1. ユーザー **Jordan Mitchell** の [プロパティ] ページを開き、Active Directory から同期されたユーザー アカウントの属性を確認します。


## 演習 5 AD DS での Microsoft Entra ID 統合機能の実装

#### タスク 1: Azure でセルフサービス パスワード リセットを有効にする

1. **SEA-ADM1** 上の Azure portal を表示する Microsoft Edge ウィンドウで、**[Microsoft Entra ID]** ページを参照します。
1. **Microsoft Entra ID** ページで、左側のナビゲーション ウィンドウの **[管理]** セクションを展開し、下にスクロールして **[ライセンス]** を選択します。
1. **[ライセンス]** ページで、左側のナビゲーション ウィンドウの **[管理]** セクションを展開し、下にスクロールして **[すべての製品]** を選択します。
1. **[すべての製品]** ページで 、**[+ 試してみる/購入]** を選択します。
1. **[アクティブ化]** ページの **[Microsoft Entra ID P2]** で **[無料試用版]** を選択して、**[アクティブ化]** を選択します。

   > **注**:Microsoft Entra ID P2 無料試用版のライセンス認証を行う際に問題が発生し、支払いオプションの設定を求めるエラー メッセージが表示される場合は、テナントの組織の要件が原因である可能性があります。 このような場合は、P2 ライセンスの機能を特に必要としないラボ演習を続行するか、管理者に問い合わせてサポートを受けることができます。

1. **[すべての製品]** ページを参照し、**[Microsoft Entra ID P2]** を選択します。
1. **[Microsoft Entra ID P2 \| ライセンスを付与されたユーザー]** ページで、**[+ 割り当て]** を選択します。
1. **[ライセンスの割り当て]** ページで、**[+ ユーザーとグループの追加]** を選択します。
1. **[ユーザーとグループの追加]** ページで、**管理者**を検索し、結果の一覧から**管理者**アカウントを選択して、**[選択]** を選択します。
1. **[ライセンスの割り当て]** ページに戻り、**[確認と割り当て]** を選択し、**[割り当て]** を選択します。

   > **注**: これは、このラボの後半で Microsoft Entra ID パスワード保護を実装するために必要です。

1. **[Microsoft Entra ID]** ページに戻ります。
1. **[Microsoft Entra ID]** ページで、**[パスワードのリセット]** を選択します。
1. **[パスワードのリセット]** ページで、構成を適用するユーザーのスコープを選択できることに注目してください。

   > **注**: パスワード リセット機能は、このラボの後半で必要な構成手順が壊れるので、有効にしないでください。

#### タスク 2: Microsoft Entra Connect でパスワード ライトバックを有効にする

1. **SEA-ADM1** の **[スタート]** メニューで、**[Azure AD Connect]** を検索して選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウで、**[構成]** を選択します。
1. **[追加のタスク]** ページで **[同期オプションのカスタマイズ]** を選んで、 **[次へ]** を選びます。
1. **[Azure AD への接続]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力し、**[次へ]** を選択します。
1. **[ディレクトリの接続]** ページで、 **[次へ]** を選択します。
1. **[ドメインと OU のフィルタリング]** ページで、 **[次へ]** を選択します。
1. **[オプション機能]** ページで、**[パスワード ライトバック]** の横にあるチェック ボックスをオンにし、**[次へ]** を選択します。

   > **注**: Active Directory ユーザーのセルフサービス パスワード リセットには、パスワード ライトバックが必要です。 これにより、Microsoft Entra ID でユーザーが変更したパスワードが Active Directory に同期されます。

1. **[構成の準備完了]** ページで、実行するアクションの一覧を確認し、**[構成]** を選択します。
1. **[構成が完了しました]** ページで、 **[終了]** を選択します。

#### タスク 3: Microsoft Entra Connect のパススルー認証を有効にする

1. **SEA-ADM1** の **[スタート]** メニューで、**[Azure AD Connect]** を検索して選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウで、**[構成]** を選択します。
1. **[追加のタスク]** ページで、**[ユーザー サインインの変更]** を選択し、**[次へ]** を選択します。
1. **[Azure AD への接続]** ページで、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントのユーザー名とパスワードを入力し、**[次へ]** を選択します。
1. **[ユーザー サインイン]** ページで、**[パススルー認証]** を選択します。
1. **[シングル サインオンを有効にする]** チェック ボックスがオンになっていることを確認し、**[次へ]** を選択します。
1. **[シングル サインオンを有効にする]** ページで、**[資格情報の入力]** を選択します。
1. **[フォレストの資格情報]** ダイアログ ボックスに講師から提供された資格情報を入力してから、**[OK]** を選択します。

1. **[シングル サインオンを有効にする]** ページで、**[資格情報の入力]** の横に緑色のチェック マークが表示されているのを確認し、**[次へ]** を選択します。
1. **[構成の準備完了]** ページで、実行するアクションの一覧を確認し、**[構成]** を選択します。
1. **[構成が完了しました]** ページで、 **[終了]** を選択します。

#### タスク 4: Azure でパススルー認証を検証する

1. **SEA-ADM1** で Azure portal を表示している Microsoft Edge ウィンドウに切り替え、**[Microsoft Entra ID]** ページに戻ります。
1. Azure portal の **[Microsoft Entra ID]** ページで、**[Microsoft Entra Connect]** を選択します。
1. **Microsoft Entra Connect | [作業の開始]** ページで、**[Connect 同期]** を選択します。
1. **Microsoft Entra Connect | [Connect 同期]** ページで、**[ユーザー サインイン]** の情報を確認します。
1. **[ユーザー サインイン]** で、**[シームレス シングル サインオン]** を選択します。
1. **[シームレス シングル サインオン]** ページで、オンプレミスのドメイン名に注目します。
1. Microsoft Edge で、**[Microsoft Entra Connect]** ページに戻ります。
1. **[Microsoft Entra Connect]** ページの **[ユーザー サインイン]** で、**[パススルー認証]** を選択します。
1. **[パススルー認証]** ページで、 **[認証エージェント]** の下の **SEA-ADM1** サーバー名に注目します。

   > **注**: Microsoft Entra ID 認証エージェントを環境内の複数のサーバーにインストールするには、Azure portal の **[パススルー認証]** ページからエージェントのバイナリをダウンロードします。 

#### タスク 5: Azure AD パスワード保護プロキシ サービスと DC エージェントをインストールして登録する

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://www.microsoft.com/en-us/download/details.aspx?id=57071` に移動します。
1. **[Azure AD パスワード保護]** ページで、**[ダウンロード]** を選択します。
1. **[ダウンロードの選択]** ページで、**AzureADPasswordProtectionProxySetup.exe** ファイルと **AzureADPasswordProtectionDCAgentSetup.msi** ファイルを選択し、**[ダウンロード]** を選択します。
1. **[複数のファイルのダウンロード]** ダイアログ ボックスで、**[許可]** を選択します。

   > **注**: ドメイン コントローラーではないサーバーにプロキシ サービスをインストールすることをお勧めします。 また、プロキシ サービスは、Microsoft Entra Connect エージェントと同じサーバーにはインストールできません。 プロキシ サービスは **SEA-SVR1** に、パスワード保護 DC エージェントは **SEA-DC1** にインストールします。

1. **SEA-ADM1** 上で、**Windows PowerShell** コンソール ウィンドウに切り替えます。
1. **Windows PowerShell** コンソールで、次のコマンドを入力し、Enter キーを押して、インターネットからファイルがダウンロードされたことを示す Zone.Identifier 代替データ ストリームを削除します。

   ```powershell
   Get-ChildItem -Path "$env:USERPROFILE\Downloads" -File | Unblock-File
   ```
1. 次のコマンドを実行して **SEA-SVR1** に **C:\Temp** ディレクトリを作成し、**AzureADPasswordProtectionProxySetup.exe** インストーラーをそのディレクトリにコピーして、インストールを起動します。

   ```powershell
   New-Item -Type Directory -Path '\\SEA-SVR1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionProxySetup.exe" -Destination '\\SEA-SVR1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Temp\AzureADPasswordProtectionProxySetup.exe -ArgumentList '/quiet /log C:\Temp\AzureADPPProxyInstall.log' -Wait }
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

1. 指示に従って、別の Microsoft Edge ウィンドウを開き、`https://microsoft.com/devicelogin` を参照し、ダイアログが表示されたら、PowerShell リモート処理セッションに表示されるメッセージに含まれるコードを入力します。 
1. ダイアログが表示されたら、演習 1 で作成した Azure AD グローバル管理者ユーザー アカウントを使用して認証し、**[続行]** を選択します。
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

1. 指示に従って、別の Microsoft Edge ウィンドウを開き、`https://microsoft.com/devicelogin` を参照し、ダイアログが表示されたら、PowerShell リモート処理セッションに表示されるメッセージに含まれるコードを入力します。 
1. プロンプトが表示されたら、演習 1 で作成した Microsoft Entra ID グローバル管理者ユーザー アカウントを使用して認証し、**[続行]** を選択します。
1. PowerShell リモート処理セッションに戻り、次のコマンドを入力し、Enter キーを押して PowerShell リモート処理セッションを終了して **SEA-DC1** に移動します。

   ```powershell
   Exit-PSsession
   ```

#### タスク 6: Azure でパスワード保護を有効にする

1. **SEA-ADM1** で、Azure portal を表示する Microsoft Edge ウィンドウに切り替え、**[Microsoft Entra ID]** ページに戻ります。 
1. **[Microsoft Entra ID]** ページで、**[セキュリティ]** を選択します。
1. **[セキュリティ]** ページで、**[認証方法]** を選択します。
1. **[認証方法]** ページで、**[パスワード保護]** を選択します。
1. **[パスワード保護]** ページで、**[カスタム リストの適用]** を **[はい]** に変更します。
1. **[カスタム禁止パスワード リスト]** テキスト ボックスに、次の単語を (1 行に 1 語) 入力します。
 
   - **Contoso**
   - **ロンドン**

   > **注**: 禁止パスワードの一覧は、ご自分の組織に関連する単語である必要があります。

1. **[Windows Server Active Directory でパスワード保護を有効にする]** が **[はい]** に設定されていることを確認します。
1. **[モード]** が **[監査]** に設定されていることを確認し、**[保存]** を選択します。

## 演習 6: クリーンアップ

#### タスク 1 Azure AD Connect のアンインストール

1. **SEA-ADM1** の **[スタート]** メニューで、**[コントロール パネル]** を選択します。
1. **[コントロール パネル]** ウィンドウの **[プログラム]** で、**[プログラムのアンインストール]** をクリックします。
1. **[プログラムのアンインストールまたは変更]** ウィンドウで **[Azure AD Connect]** を選択し、**[アンインストール]** を選択します。
1. **[プログラムと機能]** ダイアログ ボックスで **[はい]** をクリックします。
1. **[Azure AD Connect のアンインストール]** ウィンドウで、**[削除]** を選択します。
1. Azure AD Connect がアンインストールされたら、**[Azure AD Connect のアンインストール]** ウィンドウで **[終了]** を選択します。

#### タスク 2: Azure でディレクトリ同期を無効にする

1. **SEA-ADM1** 上で、**Windows PowerShell** コンソール ウィンドウに切り替えます。
1. **Windows PowerShell** コンソールで、次のコマンドを入力し、Enter キーを押して、Microsoft Graph PowerShell SDK をインストールします。

   ```powershell
   Install-Module -Name Microsoft.Graph -Force
   ```
1. NuGet プロバイダーのインストールを求めるダイアログが表示されたら、「**Y**」と入力し、Enter キーを押します。
1. 信頼されていないリポジトリからモジュールをインストールするように求めるダイアログが表示されたら、「**A**」と入力し、Enter キーを押します。
1. **Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押して、必要なアクセス許可を持つ Microsoft Entra ID に接続します。

   ```powershell
   Connect-MgGraph -Scopes "Organization.ReadWrite.All"
   ```
1. プロンプトが表示されたら、演習 1 で作成した Microsoft Entra ID 全体管理者ユーザー アカウントの資格情報を使用してサインインします。
1. **Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押してディレクトリ同期を無効にします。

   ```powershell
   $OrgID = (Get-MgOrganization).Id
   $params = @{ onPremisesSyncEnabled = $false }
   Update-MgOrganization -OrganizationId $OrgID -BodyParameter $params
   ```
1. **Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押して、ディレクトリ同期が無効になっていることを確認します。

   ```powershell
   Get-MgOrganization | Select-Object DisplayName, OnPremisesSyncEnabled
   ```

   > **注**:**OnPremisesSyncEnabled** プロパティは **False** になっているはずです。 同期されたすべてのユーザーがクラウド専用アカウントに完全に変換されるまで、最長 72 時間かかる場合があります。
