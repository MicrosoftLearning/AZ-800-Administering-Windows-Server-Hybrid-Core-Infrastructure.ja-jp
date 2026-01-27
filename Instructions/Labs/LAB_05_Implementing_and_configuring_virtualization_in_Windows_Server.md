---
lab:
  title: 'ラボ: Windows Server での仮想化の実装と構成'
  module: 'Module 5: Hyper-V virtualization in Windows Server'
---

# ラボ: Windows Server での仮想化の実装と構成

このラボは完了するまで、約 **60** 分かかります。

## シナリオ

Contoso は、米国シアトルに本社があるグローバルなエンジニアリングおよび製造会社です。 IT オフィスとデータ センターはシアトルにあり、シアトルの場所やその他の場所をサポートしています。 Contoso では最近、Windows Server サーバーとクライアント インフラストラクチャを展開しました。 

現在使用率が低い物理サーバーが多数あるため、仮想化を展開して環境を最適化する予定です。 このため、あなたは仮想マシン環境の管理に Hyper-V をどのように使用できるかを確認するための概念実証を行うことにしました。 また、Contoso DevOps チームはコンテナー テクノロジを調べ、新しいアプリケーションの展開時間を短縮できるかどうかを判断し、クラウドへのアプリケーションの移行を簡素化したいと考えています。 あなたはそのチームと協力して Windows Server コンテナーを評価し、コンテナーでのインターネット インフォメーション サービス (Web サービス) の提供を検討する予定です。

## 目標

このラボを完了すると、次のことができるようになります。

- VM を作成して構成する。
- コンテナーをインストールして構成する。

## ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-ADM1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、および **AZ-800T00A-SEA-ADM1** 仮想マシンによって、**SEA-DC1**、**SEA-SVR1** および **SEA-ADM1** のインストールがホストされます

1. **SEA-ADM1** を選択します。
1. 講師から提供された資格情報を使用してサインインします。

このラボでは、利用可能な VM 環境を使用します。

## 演習 1: VM の作成と構成

### シナリオ

この演習では、Hyper-V マネージャーと Windows Admin Center を使用して、仮想マシンを作成および構成します。 まず、プライベート仮想ネットワーク スイッチを作成します。 次に、VM にインストールするオペレーティング システムで既に準備されている基本イメージの差分ドライブを作成することにします。 最後に、概念実証のために準備した差分ドライブとプライベート スイッチを使用する第 1 世代 VM を作成します。

この演習の主なタスクは次のとおりです。

1. Hyper-V 仮想スイッチを作成します
1. 仮想ハード_ディスクを作成します。
1. 仮想マシンの作成
1. Windows Admin Center を使用して仮想マシンを管理します

#### タスク 1: Hyper-V 仮想スイッチを作成する

1. **SEA-ADM1** で、**サーバー マネージャー**を開きます。 
1. サーバー マネージャーで、**[すべてのサーバー]** を選択します。 
1. [サーバー] リストで **SEA-SVR1** を選択し、状況依存のメニューを使用して、そのサーバーをターゲットとする **Hyper-V マネージャー**を起動します。 
1. **Hyper-V マネージャー**で、**仮想スイッチ マネージャー**を使用して、**SEA-SVR1** に次の設定で仮想スイッチを作成します。

   - 名前: **Contoso Private Switch**
   - 接続の種類: **プライベート ネットワーク**

#### タスク 2: 仮想ハード ディスクを作成する

1. **SEA-ADM1** の Hyper-V マネージャーで、**[仮想ハード ディスクの新規作成ウィザード]** を使用して、**SEA-SVR1** に次の設定で新しい仮想ハード ディスクを作成します。 

   - ディスク フォーマット: **VHD**
   - ディスクの種類: **差分**
   - 名前: **SEA-VM1**
   - 場所: **C:\Base**
   - 親ディスク: **C:\Base\BaseImage.vhd**

#### タスク 3: 仮想マシンを作成する

1. **SEA-ADM1** の Hyper-V マネージャーで、**SVR1** に、次の設定で新しい仮想マシンを作成します。 

   - 名前: **SEA-VM1**
   - 場所: **C:\Base**
   - 世代: **第 1 世代**
   - メモリ: **4096**
   - ネットワーク: **Contoso Private Switch**
   - ハード ディスク: **C:\Base\SEA-VM1.vhd**

1. **SEA-VM1** の **[設定]** ウィンドウを開き、最大 RAM 値が **4096** の**動的メモリ**を有効にします。
1. Hyper-V マネージャーを閉じます。

#### タスク 4: Windows Admin Center を使用して仮想マシンを管理する

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を起動します。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。

1. **Windows PowerShell** コンソールで、次のコマンドを実行してから Enter キーを押し、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   $parameters = @{
     Source = "https://aka.ms/WACdownload"
     Destination = ".\WindowsAdminCenter.exe"
     }
   Start-BitsTransfer @parameters
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process -FilePath '.\WindowsAdminCenter.exe' -ArgumentList '/VERYSILENT' -Wait
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` で Windows Admin Center のローカル インスタンスに接続します。 
1. メッセージが表示されたら、**[Windows セキュリティ]** ダイアログ ボックスに講師から提供された資格情報を入力してから、**[OK]** を選択します。

1. Windows Admin Center で、**sea-svr1.contoso.com** への接続を追加し、講師から提供された資格情報で接続します。
1. **[ツール]** リストで、**[仮想マシン]** を選択し、**[要約]** ペインを確認します。
1. **[インベントリ]** ペインで **SEA-VM1** を開き、 **[設定]** を確認します。
1. Windows Admin Center を使用して、サイズが 5 GB の新しいディスクを作成します。
1. Windows Admin Center を使用して **SEA-VM1** を起動してから、実行中の VM の統計情報を表示します。
1. Windows Admin Center を使用して、**SEA-VM1** をシャットダウンします。
1. **[ツール]** リストで、**[仮想スイッチ]** を選択し、既存のスイッチを特定します。

### 演習 1 の結果

この演習が完了すると、Hyper-V マネージャーと Windows Admin Center を使用して、仮想スイッチ、仮想ハード ディスク、仮想マシンを作成し、その仮想マシンを管理したことになります。

## 演習 2: コンテナーのインストールと構成

### シナリオ

この演習では、Docker を使用して Windows コンテナーをインストールして実行します。 また、Windows Admin Center を使用してコンテナーを管理します。

この演習の主なタスクは次のとおりです。

1. Windows Server に Docker をインストールします
1. Windows コンテナーをインストールして実行します
1. Windows Admin Center を使用してコンテナーを管理します

#### タスク 1: Windows Server に Docker をインストールする

1. **SEA-ADM1** で、**SEA-SVR1** の **[ツール]** リストの **[PowerShell]** を選択します。 メッセージが表示されたら、講師から提供された資格情報で認証してから、Enter キーを押します。 

   > **注**: これにより、SEA-SVR1 への PowerShell リモート処理接続が確立されます。

   > **注**: Windows Admin Center での PowerShell 接続は、ラボで使用される入れ子になった仮想化が原因で比較的遅くなる可能性があります。そのため、**SEA-ADM1** の Windows Powershell コンソールから `Enter-PSSession -ComputerName SEA-SVR1` を実行する別の方法があります。

1. **Windows PowerShell** コンソールで次のコマンドを入力して Enter キーを押し、**SEA-SVR1** に Docker CE (Community Edition) をインストールします。

   ```powershell
   Invoke-WebRequest -UseBasicParsing "https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1" -o install-docker-ce.ps1

   .\install-docker-ce.ps1
   ```
 
1. インストールが完了した後、次のコマンドを入力し、Enter キーを押して **SEA-SVR1** を再起動します。

   ```powershell
   Restart-Computer -Force
   ```

#### タスク 2: Windows コンテナーをインストールして実行する


1. **SEA-SVR1** が再起動した後、PowerShell ツールを再度使用して、**SEA-SVR1** への新しい PowerShell リモート処理セッションを確立します。

   > **注**:このタスクの残りの手順では、TTY 対応のターミナルを必要とする対話型 Docker コマンドを実行する必要があります。 Windows Admin Center の PowerShell コンソールは TTY をサポートしていません。 そのため、別の方法を使用することをお勧めします。つまり、**SEA-ADM1** 上で管理者権限で **Windows PowerShell** を開き、`Enter-PSSession -ComputerName SEA-SVR1` を実行して PowerShell リモート セッションを確立します。

1. **Windows PowerShell** コンソールで次のコマンドを入力して Enter キーを押し、現在 **SEA-SVR1** に存在する Docker イメージを確認します。 

   ```powershell
   docker images
   ```

   > **注**: ローカル リポジトリ ストアにイメージがないことを確認してください。

1. 次のコマンドを入力して Enter キーを押し、Nano Server イメージをダウンロードします。

   ```powershell
   docker pull mcr.microsoft.com/windows/nanoserver:ltsc2022
   ```

   > **注**: ダウンロードが完了するのにかかる時間は、ラボ VM から Microsoft コンテナー レジストリへのネットワーク接続の使用可能な帯域幅によって異なります。

1. 次のコマンドを入力し、Enter キーを押して、Docker イメージが正常にダウンロードされたことを確認します。

   ```powershell
   docker images
   ```

1. 次のコマンドを入力し、Enter キーを押して、ダウンロードされたイメージに基づいてコンテナーを起動します。

   ```powershell
   docker run -it mcr.microsoft.com/windows/nanoserver:ltsc2022 cmd.exe 
   ```

   > **注**:この docker コマンドはコンテナーを起動し、コンテナーのコマンド ライン インターフェイスに接続します。 

1. 次のコマンドを入力し、Enter キーを押して、コンテナー ホストの IP アドレス情報を取得します。

   ```powershell
   hostname
   ```
    > **注**:これは、**SEA-SVR1** ではなく、コンテナー インスタンスのホスト名であることを確認します。

1. 次のコマンドを入力して Enter キーを押し、コンテナーにテキスト ファイルを作成します。

   ```powershell
   echo "Hello World!" > C:\Users\Public\Hello.txt
   ```

1. 次のコマンドを入力して Enter キーを押し、コンテナーのコマンド ライン インターフェイスを終了して、**SEA-SVR1** の PowerShell プロンプトに戻ります。

   ```powershell
   exit
   ```

1. 次のコマンドを入力して Enter キーを押し、docker ps コマンドを実行して終了したばかりのコンテナーのコンテナー ID を取得します。

   ```powershell
   docker ps -a
   ```
   > **注**:`-a` スイッチを指定すると、現在実行されていないものを含むすべてのコンテナーの一覧が表示されます。

1. 最初に実行したコンテナーでの変更を含む新しい helloworld イメージを作成します。 これを行うには、docker commit コマンドを実行し、\<containerID\> をコンテナーの ID に置き換えます。

   ```powershell
   docker commit <containerID> helloworld
   ```

1. これで、Hello.txt ファイルを含むカスタム イメージが作成されます。 docker images コマンドを使って、新しいイメージを確認できます。

   ```powershell
   docker images
   ```

1. docker run コマンドと --rm オプションを使って、新しいコンテナーを実行します。 このオプションを使うと、コマンド (この場合は cmd.exe) が停止すると、Docker によってコンテナーが自動的に削除されます。

   ```powershell
   docker run --rm helloworld cmd.exe /s /c type C:\Users\Public\Hello.txt
   ```
   > **注**:このコマンド ラインは、前に作成したファイルの内容を出力して、コンテナーを再び停止します。

1. 次のコマンドを入力して Enter キーを押し、元のイメージの新しいコンテナー インスタンスを起動して、作成したファイルが存在するかどうかを確認します。

   ```powershell
   docker run --rm mcr.microsoft.com/windows/nanoserver:ltsc2022 cmd.exe /s /c type C:\Users\Public\Hello.txt
   ```
   > **注**:元のイメージは、ファイルを追加しても変更されず、停止後に元の状態に戻されました。

#### タスク 3: Windows Admin Center を使用してコンテナーを管理する

1. **SEA-ADM1** の Windows Admin Center で、左上隅にある設定アイコンに移動し、**[Extensions]** を選びます。

1. **[Extensions]** ペインの **[Installed Extension]** で、**Containers** 拡張機能がインストールされて更新されていることを確認します。 この拡張機能がインストールされていない場合は、**[Available Extensions]** ペインから追加します。

1. **SEA-ADM1** の Windows Admin Center で、**sea-svr1.contoso.com**の [ツール] メニューの **[コンテナー]** を選択します。 **PowerShell** セッションを閉じるように求められたら、 **[続行]** を選択します。

1. コンテナー ペインで、 **[要約]** 、 **[コンテナー]** 、 **[イメージ]** 、 **[ネットワーク]** 、 **[ボリューム]** の各タブを参照します。

### 演習 2 の結果

この演習が完了すると、Windows Server に Docker をインストールし、Web サービスを含む Windows コンテナー イメージをダウンロードして、その機能を確認したことになります。
