---
lab:
  title: 'ラボ: Windows Server での仮想化の実装と構成'
  type: Answer Key
  module: 'Module 5: Hyper-V virtualization in Windows Server'
ms.openlocfilehash: 9202bf8b30582e0c3d8aebfddf7ca0762f53024a
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137906950"
---
# <a name="lab-answer-key-implementing-and-configuring-virtualization-in-windows-server"></a>ラボの回答キー: Windows Server での仮想化の実装と構成

### <a name="exercise-1-creating-and-configuring-vms"></a>演習 1: VM の作成と構成

#### <a name="task-1-create-a-hyper-v-virtual-switch"></a>タスク 1: Hyper-V 仮想スイッチを作成する

1. **SEA-ADM1** に接続し、必要に応じて、パスワード **Pa55w.rd** を使用し、**Contoso\\Administrator** としてサインインします。
1. **SEA-ADM1** で、 **[スタート]** を選択してから、 **[サーバー マネージャー]** を選びます。
1. サーバー マネージャーで、 **[すべてのサーバー]** を選択します。
1. [サーバー] リストで **[SEA-SVR1]** エントリを選び、そのコンテキスト メニューを表示してから、 **[Hyper-V マネージャー]** を選択します。
1. Hyper-V マネージャーで、**SEA-SVR1.Contoso.com** が選択されていることを確かめます。
1. [操作] ウィンドウで **[仮想スイッチ マネージャー]** を選択します。
1. **[仮想スイッチ マネージャー]** の **[仮想スイッチの作成]** ペインで、 **[プライベート]** を選び、 **[仮想スイッチの作成]** を選択します。
1. **[仮想スイッチのプロパティ]** ボックスで、次の設定を指定してから、 **[OK]** を選択します。

   - 名前: **Contoso Private Switch**
   - 接続の種類: **プライベート ネットワーク**

#### <a name="task-2-create-a-virtual-hard-disk"></a>タスク 2: 仮想ハード ディスクを作成する

1. **SEA-ADM1** の **SEA-SVR1** に接続されている Hyper-V マネージャーで、 **[新規]** を選択してから、 **[ハード ディスク]** を選びます。 **仮想ハード ディスクの新規作成ウィザード** が開始されます。
1. **[開始する前に] ページ** で、 **[次へ]** を選択します。
1. **[ディスク フォーマットの選択]** ページで、 **[VHD]** を選んでから、 **[次へ]** を選択します。
1. **[ディスクの種類の選択]** ページで **[差分]** を選んでから、 **[次へ]** を選択します。
1. **[名前と場所の指定]** ページで、次の設定を指定してから、 **[次へ]** を選択します。

   - 名前: **SEA-VM1**
   - 場所: **C:\Base**

1. **[ディスクの構成]** ページの **[場所]** ボックスに、「**C:\Base\BaseImage.vhd**」と入力してから **[次へ]** を選択します。
1. **[要約]** ページで、 **[完了]** を選択します。

#### <a name="task-3-create-a-virtual-machine"></a>タスク 3: 仮想マシンを作成する

1. **SEA-ADM1** の Hyper-V マネージャーで、 **[新規]** を選択し、 **[仮想マシン]** を選びます。 **[仮想マシンの新規作成ウィザード]** が起動します。
1. **[開始する前に] ページ** で、 **[次へ]** を選択します。
1. **[名前と場所の指定]** ページで 「**SEA-VM1**」と入力し、 **[仮想マシンを別の場所に格納する]** の横にあるチェック ボックスをオンにします。
1. **[場所]** ボックスに「**C:\Base**」と入力し、 **[次へ]** を選択します。
1. **[世代の指定]** ページで **[第 1 世代]** を選んでから、 **[次へ]** を選択します。
1. **[メモリの割り当て]** ページで「**4096**」と入力し、 **[次へ]** を選択します。
1. **[ネットワークの構成]** ページで、[接続] ドロップダウン メニューを選択し、 **[Contoso Private Switch]** を選んでから、 **[次へ]** を選択します。
1. **[仮想ハード ディスクの接続]** ページで、 **[既存の仮想ハード ディスクを使用する]** を選んでから、 **[参照]** を選択します。
1. **C:\Base** に移動し、 **[SEA-VM1.vhd]** 、 **[開く]** 、 **[次へ]** の順に選択します。
1. **[要約]** ページで、 **[完了]** を選択します。 仮想マシン リストに **SEA-VM1** が表示されていることに注目してください。
1. **[SEA-VM1]** を選択し、[操作] ペインの **[SEA-VM1]** の下にある **[設定]** を選択します。
1. **[ハードウェア]** リストで、 **[メモリ]** を選択します。
1. **[動的メモリ]** セクションで、 **[動的メモリを有効にする]** の横にあるチェック ボックスをオンにします。
1. **[最大 RAM]** の横に「**4096**」と入力し、 **[OK]** を選択します。
1. Hyper-V マネージャーを閉じます。

#### <a name="task-4-manage-virtual-machines-using-windows-admin-center"></a>タスク 4: Windows Admin Center を使用して仮想マシンを管理する

1. **SEA-ADM1** で **[スタート]** を選択してから、 **[Windows PowerShell (管理者)]** を選びます。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。

1. **Windows PowerShell** コンソールで、以下のコマンドを入力します。 その後、Enter キーを押して、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

1. **SEA-ADM1** で Microsoft Edge を起動してから、`https://SEA-ADM1.contoso.com` に移動します。 
1. メッセージが表示されたら、 **[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力し、 **[OK]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. **[すべての接続]** ペインで、 **[+ 追加]** を選択します。
1. **リソースの追加または作成** ペインの **[サーバー]** タイルで、 **[追加]** を選択します
1. **[サーバー名]** テキスト ボックスに、「**sea-svr1.contoso.com**」と入力します。
1. **[Use another account for this connection]\(この接続に別のアカウントを使用する\)** オプションが選択されていることを確かめ、次の資格情報を入力してから、 **[Add with credentials]\(資格情報を使用して追加\)** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

   > **注**: 手順 8 を実行した後、**お客様はこのサーバーを自分の接続リストに追加できますが、Microsoft 側でそれが利用可能であることを確認できません** という内容のエラー メッセージが 表示された場合は、 **[追加]** を選択します。 [すべての接続] ペインで、 **[sea-svr1.contoso.com]** を選んでから、 **[管理に使用する資格情報]** を選択します。 **[資格情報の指定]** ダイアログ ボックスで、 **[この接続に別のアカウントを使用する]** オプションが選択されていることを確かめ、管理者の資格情報を入力してから **[続行]** を選択します。

   > **注**: シングル サインオンを実行するには、Kerberos の制約付き委任を設定する必要があります。

1. **[sea-svr1.contoso.com]** ページの **[ツール]** リストで、 **[仮想マシン]** を選び、 **[要約]** タブを選択してから、その内容を確認します。
1. **[インベントリ]** タブを選択し、2 台の仮想マシンのリストが含まれていることを確認します。
1. **[SEA-VM1]** を選択し、その [プロパティ] ペインを確認します。
1. **[設定]** 、 **[ディスク]** の順に選択します。
1. ペインの下部までスクロールし、 **[+ ディスクの追加]** を選択します。
1. **[新しい仮想ハード ディスク]** を選択します。
1. [新しい仮想ハード ディスク] ペインの **[サイズ (GB)]** テキスト ボックスに「**5**」と入力し、他の設定は既定値のままにして、 **[作成]** を選択します。
1. **[ディスク設定の保存]** 、 **[閉じる]** の順に選択します。
1. **[SEA-VM1]** の **[プロパティ]** ペインに戻り、 **[電源]** を選んでから、 **[起動]** を選択して **SEA-VM1** を起動します。
1. 下にスクロールし、実行中の VM の統計情報を表示します。
1. ページを更新し、 **[電源]** を選び、 **[シャットダウン]** を選択してから、 **[はい]** を選んで確定します。
1. **[ツール]** リストで **[仮想スイッチ]** を選択し、ペインに 2 つのスイッチが表示されることを確認します。

### <a name="exercise-1-results"></a>演習 1 の結果

この演習が完了すると、Hyper-V マネージャーと Windows Admin Center を使用して、仮想スイッチ、仮想ハード ディスク、仮想マシンを作成し、その仮想マシンを管理したことになります。

### <a name="exercise-2-installing-and-configuring-containers"></a>演習 2: コンテナーのインストールと構成

#### <a name="task-1-install-docker-on-windows-server"></a>タスク 1: Windows Server に Docker をインストールする

1. **SEA-ADM1** で、**SEA-SVR1** の **[ツール]** リストの **[PowerShell]** を選択します。 メッセージが表示されたら、「**Pa55w.rd**」と入力して **CONTOSO\\Administrator** ユーザー アカウントを使用して認証してから、Enter キーを押します。 

   > **注**: これにより、SEA-SVR1 への PowerShell リモート処理接続が確立されます。

   > **注**: Windows Admin Center での PowerShell 接続は、ラボで使用される入れ子になった仮想化が原因で比較的遅くなる可能性があります。そのため、**SEA-ADM1** の Windows Powershell コンソールから `Enter-PSSession -ComputerName SEA-SVR1` を実行する別の方法があります。

1. **Windows PowerShell** コンソールで、次のコマンドを入力してから、Enter キーを押して TLS 1.2 を強制的に使用し、PowerShellGet モジュールをインストールします。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
   Install-PackageProvider -Name NuGet -Force
   Install-Module PowerShellGet -RequiredVersion 2.2.4 -SkipPublisherCheck
   ```
1. 信頼されていないリポジトリからのモジュールのインストールを確認するように求められたら、**A** キーを押して Enter キーを押します。
1. インストールが完了した後、次のコマンドを入力し、Enter キーを押して **SEA-SVR1** を再起動します。

   ```powershell
   Restart-Computer -Force
   ```
1. **SEA-SVR1** が再起動した後、**PowerShell** ツールを再度使用して、**SEA-SVR1** への新しい PowerShell リモート処理セッションを確立します。
1. **Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押して **SEA-SVR1** に Docker Microsoft PackageManagement プロバイダーをインストールします。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```
1. NuGet プロバイダーのプロンプトで、**Y** キーを押してから、Enter キーを押します。
1. **Windows PowerShell** コンソールで次のコマンドを入力してから、Enter キーを押して **SEA-SVR1** に Docker ランタイムをインストールします。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```
1. 確認を求められたら、**A** キーを押 してから Enter キーを押します。
1. インストールが完了した後、次のコマンドを入力し、Enter キーを押して **SEA-SVR1** を再起動します。

   ```powershell
   Restart-Computer -Force
   ```

#### <a name="task-2-install-and-run-a-windows-container"></a>タスク 2: Windows コンテナーをインストールして実行する

1. **SEA-SVR1** が再起動した後、PowerShell ツールを再度使用して、**SEA-SVR1** への新しい PowerShell リモート処理セッションを確立します。
1. **Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押して、インストールされている Docker のバージョンを確認します。

   ```powershell
   Get-Package -Name Docker -ProviderName DockerMsftProvider
   ```
1. 次のコマンドを入力し、Enter キーを押して **SEA-SVR1** に現在存在する Docker イメージを特定します。 

   ```powershell
   docker images
   ```

   > **注**: ローカル リポジトリ ストアにイメージがないことを確認してください。

1. 次のコマンドを入力し、Enter キーを押して、オンラインの Microsoft リポジトリから Docker の基本イメージを一覧表示します。

   ```powershell
   docker search Microsoft
   ```
1. 次のコマンドを入力し、Enter キーを押して、インターネット インフォメーション サービス (IIS) インストールが含まれている Nano Server イメージをダウンロードします。

   ```powershell
   docker pull nanoserver/iis
   ```

   > **注**: ダウンロードが完了するのにかかる時間は、ラボ VM から Microsoft コンテナー レジストリへのネットワーク接続の使用可能な帯域幅によって異なります。

1. 次のコマンドを入力し、Enter キーを押して、Docker イメージが正常にダウンロードされたことを確認します。

   ```powershell
   docker images
   ```
1. 次のコマンドを入力し、Enter キーを押して、ダウンロードされたイメージに基づいてコンテナーを起動します。

   ```powershell
   docker run --isolation=hyperv -d -t --name nano -p 80:80 nanoserver/iis 
   ```

   > **注**: docker コマンドでは、(ホスト オペレーティング システムの非互換性の問題に対処する) Hyper-V 分離モードのコンテナーをバックグラウンド サービスとして起動し (`-d`)、コンテナー ホストのポート 80 がコンテナーのポート 80 にマップされるようにネットワークが構成されます。 

1. 次のコマンドを入力し、Enter キーを押して、コンテナー ホストの IP アドレス情報を取得します。

   ```powershell
   ipconfig
   ```

   > **注**: vEthernet (nat) という名前のイーサネット アダプターの IPv4 アドレスを特定します。 これは、新しいコンテナーのアドレスです。 次に、**Ethernet** という名前のイーサネット アダプターの IPv4 アドレスを特定します。 これはホスト (**SEA-SVR1**) の IP アドレスであり、**172.16.10.12** に設定されています。

1. **SEA-ADM1** で、Microsoft Edge ウィンドウに切り替え、別のタブを開き、 **http://172.16.10.12** に移動します。 ブラウザーに既定の IIS ページが表示されていることを確認します。
1. **SEA-ADM1** で、**SEA-SVR1** への PowerShell リモート処理セッションに戻り、**Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押して実行中のコンテナーを一覧表示します。

   ```powershell
   docker ps
   ```
   > **注**: このコマンドは、**SEA-SVR1** で現在実行されているコンテナーに関する情報を提供するものです。 コンテナーを停止するために使用するため、コンテナー ID を記録します。 

1. 次のコマンドを入力し、Enter キーを押して、実行中のコンテナーを停止します (`<ContainerID>` プレースホルダーは、前の手順で特定したコンテナー ID に置き換えてください)。 

   ```powershell
   docker stop <ContainerID>
   ```
1. 次のコマンドを入力し、Enter キーを押して、コンテナーが停止したことを確認します。

   ```powershell
   docker ps
   ```

#### <a name="task-3-use-windows-admin-center-to-manage-containers"></a>タスク 3: Windows Admin Center を使用してコンテナーを管理する

1. **SEA-ADM1** の Windows Admin Center で、**sea-svr1.contoso.com** の [ツール] メニューの **[コンテナー]** を選択します。 **PowerShell** セッションを閉じるように求められたら、 **[続行]** を選択します。
1. コンテナー ペインで、 **[要約]** 、 **[コンテナー]** 、 **[イメージ]** 、 **[ネットワーク]** 、 **[ボリューム]** の各タブを参照します。

### <a name="exercise-2-results"></a>演習 2 の結果

この演習が完了すると、Windows Server に Docker をインストールし、Web サービスを含む Windows コンテナー イメージをダウンロードして、その機能を確認したことになります。