---
lab:
  title: 'ラボ: Windows Server での仮想化の実装と構成'
  module: 'Module 5: Hyper-V virtualization in Windows Server'
---

# <a name="lab-implementing-and-configuring-virtualization-in-windows-server"></a>ラボ: Windows Server での仮想化の実装と構成

## <a name="scenario"></a>シナリオ

Contoso は、米国シアトルに本社があるグローバルなエンジニアリングおよび製造会社です。 IT オフィスとデータ センターはシアトルにあり、シアトルの場所やその他の場所をサポートしています。 Contoso では最近、Windows Server サーバーとクライアント インフラストラクチャを展開しました。 

現在使用率が低い物理サーバーが多数あるため、仮想化を展開して環境を最適化する予定です。 このため、あなたは仮想マシン環境の管理に Hyper-V をどのように使用できるかを確認するための概念実証を行うことにしました。 また、Contoso DevOps チームはコンテナー テクノロジを調べ、新しいアプリケーションの展開時間を短縮できるかどうかを判断し、クラウドへのアプリケーションの移行を簡素化したいと考えています。 あなたはそのチームと協力して Windows Server コンテナーを評価し、コンテナーでのインターネット インフォメーション サービス (Web サービス) の提供を検討する予定です。

## <a name="objectives"></a>                **メモ:** このラボをご自分のペースでクリックして進めることができる、 **[ラボの対話型シミュレーション](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Implementing%20and%20configuring%20virtualization%20in%20Windows%20Server)** が用意されています。

対話型シミュレーションとホストされたラボの間に若干の違いがある場合がありますが、示されている主要な概念とアイデアは同じです。

- 目標
- このラボを完了すると、次のことができるようになります。

## <a name="estimated-time-60-minutes"></a>VM を作成して構成する。

## <a name="lab-setup"></a>コンテナーをインストールして構成する。

予想所要時間: 60 分 ラボのセットアップ

> 仮想マシン: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、および **AZ-800T00A-ADM1** が実行されている必要があります。

1. 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。
1. **注**: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、および **AZ-800T00A-SEA-ADM1** 仮想マシンによって、**SEA-DC1**、**SEA-SVR1** および **SEA-ADM1** のインストールがホストされます

   - **SEA-ADM1** を選択します。
   - 次の資格情報を使用してサインインします。
   - ユーザー名: **Administrator**

パスワード: **Pa55w.rd**

## <a name="exercise-1-creating-and-configuring-vms"></a>ドメイン: **CONTOSO**

### <a name="scenario"></a>このラボでは、利用可能な VM 環境を使用します。

演習 1: VM の作成と構成 シナリオ この演習では、Hyper-V マネージャーと Windows Admin Center を使用して、仮想マシンを作成および構成します。 まず、プライベート仮想ネットワーク スイッチを作成します。

次に、VM にインストールするオペレーティング システムで既に準備されている基本イメージの差分ドライブを作成することにします。

1. 最後に、概念実証のために準備した差分ドライブとプライベート スイッチを使用する第 1 世代 VM を作成します。
1. この演習の主なタスクは次のとおりです。
1. Hyper-V 仮想スイッチを作成します
1. 仮想ハード_ディスクを作成します。

#### <a name="task-1-create-a-hyper-v-virtual-switch"></a>仮想マシンの作成

1. Windows Admin Center を使用して仮想マシンを管理します 
1. タスク 1: Hyper-V 仮想スイッチを作成する 
1. **SEA-ADM1** で、**サーバー マネージャー**を開きます。 
1. サーバー マネージャーで、**[すべてのサーバー]** を選択します。

   - [サーバー] リストで **SEA-SVR1** を選択し、状況依存のメニューを使用して、そのサーバーをターゲットとする **Hyper-V マネージャー**を起動します。
   - **Hyper-V マネージャー**で、**仮想スイッチ マネージャー**を使用して、**SEA-SVR1** に次の設定で仮想スイッチを作成します。

#### <a name="task-2-create-a-virtual-hard-disk"></a>名前: **Contoso Private Switch**

1. 接続の種類: **プライベート ネットワーク** 

   - タスク 2: 仮想ハード ディスクを作成する
   - **SEA-ADM1** の Hyper-V マネージャーで、**[仮想ハード ディスクの新規作成ウィザード]** を使用して、**SEA-SVR1** に次の設定で新しい仮想ハード ディスクを作成します。
   - ディスク フォーマット: **VHD**
   - ディスクの種類: **差分**
   - 名前: **SEA-VM1**

#### <a name="task-3-create-a-virtual-machine"></a>場所: **C:\Base**

1. 親ディスク: **C:\Base\BaseImage.vhd** 

   - タスク 3: 仮想マシンを作成する
   - **SEA-ADM1** の Hyper-V マネージャーで、**SVR1** に、次の設定で新しい仮想マシンを作成します。
   - 名前: **SEA-VM1**
   - 場所: **C:\Base**
   - 世代: **第 1 世代**
   - メモリ: **4096**

1. ネットワーク: **Contoso Private Switch**
1. ハード ディスク: **C:\Base\SEA-VM1.vhd**

#### <a name="task-4-manage-virtual-machines-using-windows-admin-center"></a>**SEA-VM1** の **[設定]** ウィンドウを開き、最大 RAM 値が **4096** の**動的メモリ**を有効にします。

1. Hyper-V マネージャーを閉じます。

   >タスク 4: Windows Admin Center を使用して仮想マシンを管理する

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を起動します。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. **注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **Windows PowerShell** コンソールで、次のコマンドを実行してから Enter キーを押し、最新バージョンの Windows Admin Center をダウンロードします。 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。

1. **注**: インストールが完了するまで待ちます。 
1. これには 2 分ほどかかります。

   - **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` で Windows Admin Center のローカル インスタンスに接続します。
   - メッセージが表示されたら、**[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力し、**[OK]** を選択します。

1. ユーザー名: **CONTOSO\\Administrator** 
1. パスワード: **Pa55w.rd**
1. Windows Admin Center で、**sea-svr1.contoso.com** への接続を追加し、パスワード **Pa55w.rd** を使用して **CONTOSO\\Administrator** としてそれに接続します。
1. **[ツール]** リストで、**[仮想マシン]** を選択し、**[要約]** ペインを確認します。
1. **[インベントリ]** ペインで **SEA-VM1** を開き、 **[設定]** を確認します。
1. Windows Admin Center を使用して、サイズが 5 GB の新しいディスクを作成します。

### <a name="exercise-1-results"></a>Windows Admin Center を使用して **SEA-VM1** を起動してから、実行中の VM の統計情報を表示します。

Windows Admin Center を使用して、**SEA-VM1** をシャットダウンします。

## <a name="exercise-2-installing-and-configuring-containers"></a>**[ツール]** リストで、**[仮想スイッチ]** を選択し、既存のスイッチを特定します。

### <a name="scenario"></a>演習 1 の結果

この演習が完了すると、Hyper-V マネージャーと Windows Admin Center を使用して、仮想スイッチ、仮想ハード ディスク、仮想マシンを作成し、その仮想マシンを管理したことになります。 演習 2: コンテナーのインストールと構成

シナリオ

1. この演習では、Docker を使用して Windows コンテナーをインストールして実行します。
1. また、Windows Admin Center を使用してコンテナーを管理します。
1. この演習の主なタスクは次のとおりです。

#### <a name="task-1-install-docker-on-windows-server"></a>Windows Server に Docker をインストールします

1. Windows コンテナーをインストールして実行します 

   > Windows Admin Center を使用してコンテナーを管理します

1. タスク 1: Windows Server に Docker をインストールする 

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
   Install-PackageProvider -Name NuGet -Force
   Install-Module PowerShellGet -RequiredVersion 2.2.4 -SkipPublisherCheck
   ```
1. **SEA-ADM1** の Windows Admin Center で、**sea-svr1.contoso.com** に接続している間に、**[ツール]** メニューを使用して、そのサーバーへの PowerShell リモート処理セッションを確立します。

   ```powershell
   Restart-Computer -Force
   ```
1. **注**: Windows Admin Center での PowerShell 接続は、ラボで使用される入れ子になった仮想化が原因で比較的遅くなる可能性があります。そのため、**SEA-ADM1** の Windows Powershell コンソールから `Enter-PSSession -ComputerName SEA-SVR1` を実行する別の方法があります。

1. PowerShell コンソールで次のコマンドを実行して、TLS 1.2 を強制的に使用し、PowerShellGet モジュールをインストールします。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```
1. インストールが完了した後、次のコマンドを実行して **SEA-SVR1** を再起動します。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```
1. **SEA-SVR1** が再起動した後、**PowerShell** ツールを再度使用して、**SEA-SVR1** への新しい PowerShell リモート処理セッションを確立します。

   ```powershell
   Restart-Computer -Force
   ```

#### <a name="task-2-install-and-run-a-windows-container"></a>**Windows PowerShell** コンソールで次のコマンドを実行し、**SEA-SVR1** に Docker Microsoft PackageManagement プロバイダーをインストールします。

1. **Windows PowerShell** コンソールで次のコマンドを実行し、**SEA-SVR1** に Docker ランタイムをインストールします。
1. インストールが完了した後、次のコマンドを実行して **SEA-SVR1** を再起動します。

   ```powershell
   Get-Package -Name Docker -ProviderName DockerMsftProvider
   ```
1. タスク 2: Windows コンテナーをインストールして実行する 

   ```powershell
   docker images
   ```

   > **SEA-SVR1** が再起動した後、**PowerShell** ツールを再度使用して、**SEA-SVR1** への新しい PowerShell リモート処理セッションを確立します。

1. **Windows PowerShell** コンソールで次のコマンドを実行し、インストールされた Docker のバージョンを確認します。

   ```powershell
   docker pull nanoserver/iis
   ```

   > 次のコマンドを実行して、現在 **SEA-SVR1** に存在する Docker イメージを特定します。

1. **注**: ローカル リポジトリ ストアにイメージがないことを確認してください。

   ```powershell
   docker images
   ```
1. 次のコマンドを実行して、インターネット インフォメーション サービス (IIS) インストールを含む Nano Server イメージをダウンロードします。

   ```powershell
   docker run --isolation=hyperv -d -t --name nano -p 80:80 nanoserver/iis 
   ```

   > **注**: ダウンロードが完了するのにかかる時間は、ラボ VM から Microsoft コンテナー レジストリへのネットワーク接続の使用可能な帯域幅によって異なります。 

1. 次のコマンドを実行して、Docker イメージが正常にダウンロードされたことを確認します。

   ```powershell
   ipconfig
   ```

   > 次のコマンドを実行し、ダウンロードされたイメージに基づいてコンテナーを起動します。 **注**: docker コマンドでは、(ホスト オペレーティング システムの非互換性の問題に対処する) Hyper-V 分離モードのコンテナーをバックグラウンド サービスとして起動し (`-d`)、コンテナー ホストのポート 80 がコンテナーのポート 80 にマップされるようにネットワークが構成されます。 次のコマンドを実行して、コンテナー ホストの IP アドレス情報を取得します。 **注**: vEthernet (nat) という名前のイーサネット アダプターの IPv4 アドレスを特定します。

1. これは、新しいコンテナーのアドレスです。 次に、**Ethernet** という名前のイーサネット アダプターの IPv4 アドレスを特定します。
1. これはホスト (**SEA-SVR1**) の IP アドレスであり、**172.16.10.12** に設定されています。

   ```powershell
   docker ps
   ```
   > **SEA-ADM1** で、Microsoft Edge ウィンドウに切り替え、別のタブを開き、**http://172.16.10.12** に移動します。 ブラウザーに既定の IIS ページが表示されていることを確認します。 

1. **SEA-ADM1** で、**SEA-SVR1** への PowerShell リモート処理セッションに戻り、**Windows PowerShell** コンソールで次のコマンドを実行し、実行中のコンテナーを一覧表示します。

   ```powershell
   docker stop <ContainerID>
   ```
1. **注**: このコマンドは、**SEA-SVR1** で現在実行されているコンテナーに関する情報を提供するものです。

   ```powershell
   docker ps
   ```

#### <a name="task-3-use-windows-admin-center-to-manage-containers"></a>コンテナーを停止するために使用するため、コンテナー ID を記録します。

1. 次のコマンドを実行して、実行中のコンテナーを停止します (`<ContainerID>` プレースホルダーは、前の手順で特定したコンテナー ID に置き換えてください)。 次のコマンドを実行して、コンテナーが停止したことを確認します。
1. タスク 3: Windows Admin Center を使用してコンテナーを管理する

### <a name="exercise-2-results"></a>**SEA-ADM1** の Windows Admin Center で、**sea-svr1.contoso.com** の **[ツール]** メニューの **[コンテナー]** を選択します。

**PowerShell** セッションを閉じるように求められたら、 **[続行]** を選択します。
