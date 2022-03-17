---
lab:
  title: 'ラボ : Windows Server でのネットワーク インフラストラクチャ サービスの実装と構成'
  module: 'Module 7: Network Infrastructure services in Windows Server'
ms.openlocfilehash: 3d4a37155a7609316235b4ec8c4fcb8360cfe2f7
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137906999"
---
# <a name="lab-implementing-and-configuring-network-infrastructure-services-in-windows-server"></a>ラボ : Windows Server でのネットワーク インフラストラクチャ サービスの実装と構成

## <a name="scenario"></a>シナリオ

Contoso, Ltd. は、ネットワーク サービスに関する複雑な要件がある大規模な組織です。 これらの要件を満たせるように、DHCP を展開および構成し、サービスの可用性を確保するための高可用性を実現します。 また、Contoso 内の部門である Trey Research がテスト領域で独自の DNS サーバーを持てるように、DNS を設定します。 最後に、Windows Admin Center へのリモート アクセスを提供し、Web アプリケーション プロキシを使用してセキュリティで保護します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- DHCP を展開および構成する。
- DNS を展開および構成する。

## <a name="estimated-time-60-minutes"></a>予想所要時間: 60 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、および **AZ-800T00A-ADM1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、および **AZ-800T00A-SEA-ADM1** 仮想マシンによって、**SEA-DC1**、**SEA-SVR1**、および **SEA-ADM1** のインストールがホストされます

1. **SEA-ADM1** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、利用可能な VM 環境を使用します。

## <a name="exercise-1-deploying-and-configuring-dhcp"></a>演習 1: DHCP の展開と構成

### <a name="scenario"></a>シナリオ

Contoso の Trey Research の下位部門には、約 50 人のユーザーのみが存在する別のオフィスがあります。 彼らはすべて自分のコンピューターで IP アドレスを手動で構成しており、代わりに DHCP の使用を開始したいと考えています。 DHCP は、Trey Research サイトのスコープを持つ **SEA-SVR1** にインストールします。 さらに、**SEA-DC1** での高可用性を実現するために新しい DHCP サーバーを使用して、DHCP フェールオーバーを構成します。

この演習の主なタスクは次のとおりです。

1. DHCP 役割をインストールします。
1. DHCP サーバーを承認します。
1. スコープを作成します。
1. DHCP フェールオーバーを構成します。
1. DHCP 機能を確認します。

### <a name="task-1-install-the-dhcp-role"></a>タスク 1: DHCP 役割をインストールする

1. **SEA-ADM1** で、管理者として Windows PowerShell を起動します。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。

1. **Windows PowerShell** コンソールで、次のコマンドを実行して、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを実行して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` で Windows Admin Center のローカル インスタンスに接続します。

   >**注**: リンクが機能しない場合は、**SEA-ADM1** で **WindowsAdminCenter.msi** ファイルを参照し、コンテキスト メニューを開いて **[修復]** を選択します。 修復が完了した後、Microsoft Edge を更新します。 
   
1. メッセージが表示されたら、 **[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力し、 **[OK]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. Windows Admin Center で、**sea-svr1.contoso.com** への接続を追加し、パスワード **Pa55w.rd** を使用して **CONTOSO\\Administrator** としてそれに接続します。
1. **[ツール]** リストで、 **[役割と機能]** を使用して、**SEA-SVR1** に DHCP 役割をインストールします。
1. **[ツール]** リストで、**DHCP** ツールを参照し、**DHCP PowerShell** ツールをインストールします。 

   > **注**: **sea-svr1.contoso.com** の [ツール] ペインで **DHCP** エントリを利用できない場合は、**Microsoft Edge** ページを更新して、もう一度やり直してください。

### <a name="task-2-authorize-the-dhcp-server"></a>タスク 2: DHCP サーバーを承認する

1. **SEA-ADM1** で、サーバー マネージャーを開きます。
1. サーバー マネージャーで、 **[通知]** を開き、 **[DHCP 構成を完了する]** を開いてから、既定のオプションを使用して **[DHCP インストール後の構成ウィザード]** を完了します。

### <a name="task-3-create-a-scope"></a>タスク 3: スコープを作成する

1. **SEA-ADM1**  の Windows Admin Center で、**sea-svr1.contoso.com** に接続している間に、**DHCP** ツールを使用して、次の設定で新しいスコープを作成します。

   - プロトコル: **IPv4**
   - 名前: **ContosoClients**
   - 開始 IP アドレス: **10.100.150.50**
   - 終了 IP アドレス: **10.100.150.254**
   - DHCP クライアント サブネット マスク: **255.255.255.0**
   - ルーター (デフォルト ゲートウェイ): **10.100.150.1**
   - DHCP クライアントのリース期間: **4 日**

1. サーバー マネージャーの **[ツール]** メニューで、 **[DHCP 管理コンソール]** を開きます。
1. **[DHCP 管理コンソール]** で、すべての承認済みサーバー (**172.16.10.12** および **SEA-DC1.contoso.com**) を追加します。
1. **[DHCP 管理コンソール]** で、DHCP サーバー **172.16.10.12** ノードを参照し、**ContosoClients** スコープで、値が **172.16.10.10** のスコープ オプション **006 DNS サーバー** を追加します。

### <a name="task-4-configure-dhcp-failover"></a>タスク 4: DHCP フェールオーバーを構成する

1. **SEA-ADM1** の **DHCP** 管理コンソールで、DHCP サーバー **172.16.10.12** の **IPv4** ノードを参照し、次の設定を使用して **SEA-DC1.contoso.com** で **ContosoClients** スコープのフェールオーバーを構成します。

   - リレーションシップ名: **SEA-SVR1 から SEA-DC1**
   - クライアントの最大リード タイム: **1 時間**
   - モード: **ホット スタンバイ**
   - パートナー サーバーの役割: **スタンバイ**
   - スタンバイ サーバー用に予約されているアドレス: **5%**
   - 状態の切り替え間隔: **無効**
   - メッセージ認証を有効にする: **有効**
   - 共有シークレット: **DHCP フェールオーバー**

1. **SEA-SVR1** のスコープが 1 つだけであることを確認します。
1. **SEA-DC1** のスコープが 2 つになっていることを確認します。
1. **SEA-ADM1** の **DHCP** 管理コンソールで、DHCP サーバー **SEA-DC1.contoso.com** の **IPv4** ノードを参照し、既存のフェールオーバー リレーションシップの設定を使用して、**172.16.10.12** で **Contoso** スコープのフェールオーバーを構成します。
1. 両方のスコープが **172.16.10.12** (**SEA-SVR1**) の **IPV4** ノード内に表示されるようになったことを確認します。

### <a name="task-5-verify-dhcp-functionality"></a>タスク 5: DHCP 機能を確認する

1. **SEA-ADM1** で、IP 構成を静的から動的に割り当てられるように変更します。
1. 結果の IP 構成を調べ、DHCP リースが **SEA-SVR2 (172.16.10.12)** から取得されたことを確認します。
1. **SEA-ADM1** の **DHCP 管理コンソール** で、両方の DHCP サーバーの **Contoso** スコープに **SEA-ADM1** のリースが一覧表示されていることを確認します。
1. **SEA-ADM1** で、**DHCP 管理コンソール** を使用して、**SEA-SVR2 (172.16.10.12)** 上の **DHCP** サービスを停止します。
1. **SEA-ADM1** のイーサネット ネットワーク接続を無効にしてから再度有効にして、リースを強制的に更新します。
1. **SEA-ADM1** で、同じ DHCP リースが **SEA-DC1 (172.16.10.10)** から取得されていることを確認します。

## <a name="exercise-2-deploying-and-configuring-dns"></a>演習 2: DNS の展開と構成

### <a name="scenario"></a>シナリオ

Contoso 内の Trey Research の場所で働いているスタッフには、テスト環境にレコードを作成するための自身の DNS サーバーが必要です。 しかし、テスト環境では、引き続き Contoso のインターネット DNS 名とリソース レコードを解決できる必要があります。 これらのニーズを満たすために、インターネット サービス プロバイダー (ISP) への転送を構成し、**SEA-DC1** に **contoso.com** の条件付きフォワーダーを作成します。 また、ユーザーの場所に基づいて異なる IP アドレスの解決を必要とするテスト アプリケーションもあります。 DNS ポリシーを使用して、**testapp.treyresearch.net** を、本社のユーザーに対して異なる方法で解決するように構成します。

この演習の主なタスクは次のとおりです。

1. DNS 役割をインストールします。
1. DNS ゾーンの作成。
1. 転送を構成します。
1. 条件付き転送を構成します。
1. DNS ポリシーを構成します。
1. DNS ポリシーの機能を確認します。

### <a name="task-1-install-the-dns-role"></a>タスク 1: DNS 役割をインストールする

1. **SEA-ADM1** で、**sea-svr1.contoso.com** に接続されている Windows Admin Center の **[ツール]** リストの **[役割と機能]** を使用して、**SEA-SVR1** に DNS 役割をインストールします。
1. **[ツール]** リストで、**DNS** ツールを参照し、**DNS PowerShell** ツールをインストールします。 

   > **注**: **sea-svr1.contoso.com** の **[ツール]** リストで **DNS** エントリを利用できない場合は、**Microsoft Edge** ページを更新して、もう一度試してください。

### <a name="task-2-create-a-dns-zone"></a>タスク 2: DNS ゾーンを作成する

1. **SEA-ADM1**  の Windows Admin Center で、**sea-svr1.contoso.com** に接続している間に、**DNS** ツールを使用して、次の設定で新しい DNS ゾーンを作成します。

   - ゾーンの種類: **プライマリ**
   - ゾーン名: **TreyResearch.net**
   - ゾーン ファイル: **新しいファイルを作成**
   - ゾーン ファイル名: **TreyResearch.net.dns**
   - 動的更新: **動的更新を許可しない**

1. Windows Admin Center で、**sea-svr1.contoso.com** に接続している間に、**DNS** ツールを使用して、次の設定で **TreyResearch.net** ゾーンに新しい DNS レコードを作成します。

   - DNS レコードの種類: **ホスト (A)**
   - レコード名: **TestApp**
   - IP アドレス: **172.30.99.234**
   - Time to Live: **600**

1. **SEA-ADM1** の **Windows PowerShell** コンソールで、次のコマンドを実行して、新しく作成されたレコードが正しく解決されていることを確認します。

    ```powershell
    Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
    ```

### <a name="task-3-configure-forwarding"></a>タスク 3: 転送を構成する

1. **SEA-ADM1** で、サーバー マネージャーの **[ツール]** メニューから、**DNS マネージャー コンソール** を開きます。
1. **[DNS マネージャー]** で、**SEA-SVR1.contoso.com** に接続します。
1. **SEA-SVR1.contoso.com** のプロパティの **[フォワーダー]** タブで、フォワーダーとして **131.107.0.100** を構成します。

### <a name="task-4-configure-conditional-forwarding"></a>タスク 4: 条件付き転送を構成する

1. **SEA-ADM1** の **[DNS マネージャー]** で、**SEA-SVR1.contoso.com** に接続している間に、**SEA-DC1.contoso.com** (**172.16.10.10**) に要求を転送する **Contoso.com** の新しい条件付きフォワーダーを作成します。
1. **SEA-ADM1** の **Windows PowerShell** コンソールで、次のコマンドを実行して条件付きフォワーダーが動作していることを確認します。

    ```powershell
    Resolve-DnsName -Server sea-svr1.contoso.com -Name sea-dc1.contoso.com
    ```

### <a name="task-5-configure-dns-policies"></a>タスク 5: DNS ポリシーを構成する

1. **SEA-ADM1** の Windows Admin Center で、**sea-svr1.contoso.com** に接続している間に、 **[ツール]** リストの **PowerShell** を使用して PowerShell リモート処理セッションを確立します。
1. **Windows PowerShell** プロンプトで、次のコマンドを実行して本社のサブネットを作成します。

    ```powershell
    Add-DnsServerClientSubnet -Name "HeadOfficeSubnet" -IPv4Subnet "172.16.10.0/24"
    ```

1. 次のコマンドを実行して、本社のゾーン スコープを作成します。

    ```powershell
    Add-DnsServerZoneScope -ZoneName "TreyResearch.net" -Name "HeadOfficeScope"
    ```

1. 次のコマンドを実行して、本社のスコープの新しいリソース レコードを作成します。

    ```powershell
    Add-DnsServerResourceRecord -ZoneName "TreyResearch.net" -A -Name "testapp" -IPv4Address "172.30.99.100" -ZoneScope "HeadOfficeScope"
    ```

1. 次のコマンドを実行して、本社のサブネットとゾーン スコープをリンクする新しいポリシーを作成します。

    ```powershell
    Add-DnsServerQueryResolutionPolicy -Name "HeadOfficePolicy" -Action ALLOW -ClientSubnet "eq,HeadOfficeSubnet" -ZoneScope "HeadOfficeScope,1" -ZoneName "TreyResearch.net"
    ```

### <a name="task-6-verify-dns-policy-functionality"></a>タスク 6: DNS ポリシー機能を確認する

1. **SEA-ADM1** の **Windows PowerShell** コンソールで、`ipconfig` を実行して、**SEA-ADM1** が **HeadOfficeSubnet (172.16.10.0)** 上にあることを確認します。
1. **Windows PowerShell** コンソールで、次のコマンドを実行して DNS ポリシーをテストします。

    ```powershell
    Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
    ```

   > **注**: 名前が、**HeadOfficePolicy** で構成された IP アドレス **172.30.99.100** に解決されることを確認します。

1. **SEA-ADM1** に割り当てられた IP アドレスを **172.16.10.11** から、**HeadOfficeSubnet** の IP アドレス範囲外の IP アドレス (**172.16.11.11**) に変更します。
1. **Windows PowerShell** コンソールで、次のコマンドを実行して DNS ポリシーをテストします。

    ```powershell
    Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
    ```
   > **注**: 名前が **172.30.99.234** に解決されることを確認します。 **SEA-ADM1** の IP アドレスが **HeadOfficeSubnet** 内に存在しなくなったため、これは想定内です。 `testapp.treyresearch.net` をターゲットとする **(172.16.10.0/24)** の **HeadOfficeSubnet** からの DNS クエリは、**172.30.99.100** に解決されます。 `testapp.treyresearch.net` をターゲットとするこのサブネット外からの DNS クエリは、**172.30.99.234** に解決されます。

1. **SEA-ADM1** の IP アドレスを元の値に戻します。
