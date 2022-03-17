---
lab:
  title: 'ラボ: Windows Server の管理'
  type: Answer Key
  module: 'Module 3: Windows Server administration'
ms.openlocfilehash: 617789fc8fcaf6ef6019c2bb65de0f7366742eac
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137906958"
---
# <a name="lab-managing-windows-server"></a>ラボ: Windows Server の管理

## <a name="exercise-1-implementing-and-using-remote-server-administration"></a>演習 1: リモート サーバー管理の実装と使用

#### <a name="task-1-install-windows-admin-center"></a>タスク 1: Windows Admin Center をインストールする

1. **SEA-ADM1** に接続し、必要に応じて、パスワード **Pa55w.rd** を使用し、**Contoso\\Administrator** としてサインインします。
1. **SEA-ADM1** 上で **[スタート]** を選択し、 **[Windows PowerShell (管理者)]** を選択します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力してから Enter キーを押して、Windows Admin Center の最新バージョンをダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

#### <a name="task-2-add-servers-for-remote-administration"></a>タスク 2: リモート管理用のサーバーを追加する

1. **SEA-ADM1** で Microsoft Edge を開始して、`https://SEA-ADM1.contoso.com` にアクセスします。 
1. ダイアログが表示されたら、 **[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力して、 **[OK]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. **[このリリースの新機能]** ポップアップ ウィンドウを確認し、右上隅の **[閉じる]** を選択します。
1. **[すべての接続]** ページを確認します。**sea-adm1.contoso.com** エントリが含まれていることに注目します。 
1. **[すべての接続]** ページで、 **[+ 追加]** を選択します。 
1. リソースの追加または作成ペインの **[サーバー]** タイルで、 **[追加]** を選択します。
1. **[サーバー名]** テキスト ボックスに、「**sea-svr1.contoso.com**」と入力します。
1. **[Use another account for this connection]\(この接続に別のアカウントを使用する\)** オプションが選択されていることを確認し、次の資格情報を入力して、 **[Add with credentials]\(資格情報を使用して追加\)** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

   > **注**: 手順 7 を実行した後、**お客様はこのサーバーを自分の接続リストに追加できますが、Microsoft 側でそれが利用可能であることを確認できません。** という内容のエラー メッセージが表示された場合、 **[追加]** を選択します。 [すべての接続] ウィンドウで、 **[sea-svr1.contoso.com]** を選択してから、 **[管理に使用する資格情報]** を選択します。 **[資格情報の指定]** ダイアログ ボックスで、 **[この接続に別のアカウントを使用する]** オプションを確実に選択し、管理者の資格情報を入力してから、 **[続行]** を選択します。

   > **注**: シングル サインオンを実行するには、Kerberos の制約付き委任を設定する必要があります。

#### <a name="task-3-configure-windows-admin-center-extensions"></a>タスク 3: Windows Admin Center 拡張機能を構成する

1. **SEA-ADM1** 上の、Windows Admin Center を表示している Microsoft Edge ウィンドウの右上隅で、 **[設定]** アイコン (歯車ホイール) を選択します。
1. 左側のウィンドウで、**[拡張機能]** を選択します。 使用可能な拡張機能を確認します。
1. **[セキュリティ (プレビュー)]** 拡張機能を選択して、 **[インストール]** を選択します。 拡張機能がインストールされ Windows Admin Center が更新されます。

   > **注**: **セキュリティ (プレビュー)** 拡張機能を使用できない場合は、別の Microsoft 拡張機能を選択します。

1. 詳細ウィンドウで、 **[インストール済みの拡張機能]** を選択し、一覧に DNS (プレビュー) 拡張機能が含まれていることを確認します。
1. 上部メニューの **[設定]** の横にあるドロップダウン矢印を選択してから、 **[サーバー マネージャー]** を選択します。
1. **[サーバーの接続]** ページで、 **[sea-dc1.contoso.com]** リンクを選択します。
1. **[この接続に別のアカウントを使用する]** オプションを確実に選択し、 **[すべての接続にこれらの資格情報を使用する]** を選択し、次の資格情報を入力してから、 **[続行]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. DNS PowerShell ツールをインストールするには、左側のウィンドウの **[ツール]** の一覧で **[DNS]** を選択してから、 **[インストール]** を選択します。 ツールのインストールには 1 分もかかりません。
1. **Contoso.com** ゾーンを選択し、その DNS レコードの一覧を確認します。

#### <a name="task-4-verify-remote-administration"></a>タスク 4: リモート管理を確認する

1. **SEA-ADM1** 上の Windows Admin Center の左側のウィンドウにある **[ツール]** の一覧で、 **[概要]** を選択します。 Windows Admin Center の詳細ウィンドウには、基本的なサーバー情報とパフォーマンスの監視が表示されます。
1. 左側のウィンドウの **[ツール]** の一覧を下にスクロールし、使用可能な基本的な管理ツールを確認します。 **[ロールと機能]** を選択し、どのロールと機能がインストール済みとして表示されているか、またはインストール可能であるかを確認します。 下にスクロールし、 **[Telnet クライアント]** チェック ボックスをオンにしてから、ウィンドウの上部にある **[+ インストール]** を選択します。
1. [役割と機能のインストール] ウィンドウで **[はい]** を選択し、Telnet クライアントが正常にインストールされたことを確認するメッセージが表示されるのを待ちます。
1. 左側のウィンドウの一番下にある、 **[ツール]** の一覧の下で、 **[設定]** を選択します。
1. 右側の **[設定]** セクションで、 **[リモート デスクトップ]** を選択します。
1. **[リモート デスクトップ]** セクションで、オプションの **[このコンピューターへのリモート接続を許可する]** チェック ボックスをオンにして、 **[保存]** を選択します。
1. 左側のウィンドウにある **[ツール]** の一覧で、 **[リモート デスクトップ]** を選択します。
1. [リモート デスクトップ] ウィンドウで、 **[このコンピューターへの接続について今後確認しない]** チェックボックスをオンにして、 **[接続]** を選択します。
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

この演習を完了すると、Windows Admin Center がインストールされ、ラボ環境内のサーバーに接続されます。 機能のインストールや、リモート デスクトップ接続の有効化およびテストなどのリモート管理タスクを複数実行しました。 最後に、PowerShell リモート処理を使用してサービスの状態を確認して、開始しました。
