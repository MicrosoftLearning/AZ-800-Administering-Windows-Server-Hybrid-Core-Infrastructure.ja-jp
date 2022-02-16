---
lab:
  title: 'ラボ: Windows Server での記憶域ソリューションの実装'
  module: 'Module 9: File servers and storage management in Windows Server'
ms.openlocfilehash: 26aa54dc19222f7dd15dd5e64db939abdc8fada3
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907034"
---
# <a name="lab-implementing-storage-solutions-in-windows-server"></a>ラボ: Windows Server での記憶域ソリューションの実装

## <a name="scenario"></a>シナリオ

Contoso, Ltd. では、ストレージ アクセスを簡略化し、ストレージ レベルで冗長性を提供するために、Windows Server サーバーに記憶域スペース機能を実装する必要があります。 あなたは経営陣から、記憶域を節約するためのデータ重複排除をテストするよう求められています。 また、Internet Small Computer System Interface (iSCSI) 記憶域を実装して、組織内に記憶域を展開するためのより簡単なソリューションを提供することも望んでいます。 さらに、組織は、記憶域を高可用性にするためのオプションと、高可用性のために満たす必要がある要件を調べています。 あなたは、高可用性記憶域 (特に記憶域スペース ダイレクト) を使用する実現可能性をテストする必要があります。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- データ重複除去の実装。
- iSCSI 記憶域を構成する。
- 記憶域スペースを構成する。
- 記憶域スペース ダイレクトを実装する。

## <a name="estimated-time-90-minutes"></a>予想所要時間: 90 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-SEA-SVR2**、**AZ-800T00A-SEA-SVR3**、**AZ-800T00A-ADM1** が稼働している必要があります。 

- 演習 1 - 3 の場合: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR3**、**AZ-800T00A-SEA-ADM1**
- 演習 4 の場合: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-SEA-SVR2**、**AZ-800T00A-SEA-SVR3**、**AZ-800T00A-SEA-ADM1**

> **注**: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-SEA-SVR2**、**AZ-800T00A-SEA-SVR3**、**AZ-800T00A-SEA-ADM1** の各仮想マシンは、それぞれ、**SEA-DC1**、**SEA-SVR1**、**SEA-SVR2**、**SEA-SVR3**、**SEA-ADM1** のインストールをホストしています。

1. **SEA-ADM1** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、利用可能な VM 環境を使用します。

## <a name="lab-exercise-1-implementing-data-deduplication"></a>ラボ演習 1: データ重複除去の実装

### <a name="scenario"></a>シナリオ

あなたは、サーバー マネージャーを使用してデータ重複除去役割サービスをインストールすることにします。 ドライブ **M** は使用量が多く、一部のフォルダーには重複するファイルが含まれる可能性があると判断します。 データ重複除去の役割を有効にして構成し、このボリュームで使用されているスペースを減らすことにします。

この演習の主なタスクは次のとおりです。

1. **SEA-SVR3** にデータ重複除去機能をインストールします。
1. **SEA-SVR3** のドライブ **M** でデータ重複除去を有効にして構成します。
1. ファイルを追加し、重複除去を監視して、データ重複除去をテストします。

#### <a name="task-1-install-the-data-deduplication-role-service"></a>タスク 1: データ重複除去役割サービスをインストールする

1. **SEA-ADM1** で、**サーバー マネージャー** を使用して **SEA-SVR3** にデータ重複除去役割サービスをインストールします。
1. **SEA-ADM1** で、**Users** グループに **読み取り** アクセス許可を付与して **C:\Labfiles** フォルダーを共有します。
1. **SEA-SVR3** のコンソール セッションに切り替え、必要に応じて、**Pa55w.rd** のパスワードを使用して **CONTOSO\\Administrator** としてサインインします。
1. **SEA-SVR3** で **Windows PowerShell** セッションを開始し、**Windows PowerShell** コンソールで次のコマンドを実行して、ReFS でフォーマットされ、ドライブ文字 **M** が割り当てられたボリュームを作成します。

   ```powershell
   Get-Disk
   Initialize-Disk -Number 1
   New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter M
   Format-Volume -DriveLetter M -FileSystem ReFS
   ```
1. **SEA-SVR3** の **Windows PowerShell** コンソールで、次のコマンドを入力して、**SEA-ADM1** から重複除去するサンプル ファイルを作成するスクリプトをコピーして実行し、結果を確認します。

   ```powershell
   New-PSDrive –Name 'X' –PSProvider FileSystem –Root '\\SEA-ADM1\Labfiles'
   New-Item -Type Directory -Path 'M:\Data' -Force
   Copy-Item -Path X:\Lab09\CreateLabFiles.cmd -Destination M:\Data\ -PassThru
   Start-Process -FilePath M:\Data\CreateLabFiles.cmd -PassThru
   Set-Location -Path M:\Data
   Get-ChildItem -Path .
   Get-PSDrive -Name M
   ```

   > **注**: ドライブ **M** の空き領域を記録しておきます。

#### <a name="task-2-enable-and-configure-data-deduplication"></a>タスク 2: データ重複除去を有効にして構成する

1. **SEA-ADM1** へのコンソール セッションに切り替えます。
1. **サーバー マネージャー** の **[ファイル サービスと記憶域サービス]** インターフェイスを使用して、**SEA-SVR3** 上のディスクを表示します。
1. 次の設定を使用して、**SEA-SVR3** 上のディスク番号 **1** の **M** ボリュームでデータ重複除去を有効にします。

   - 重複除去オプション: **汎用ファイル サーバー**
   - 次の期間経過したファイルを重複除去の対象とする (日数): **0**
   - スループットの最適化を有効にする: オン

#### <a name="task-3-test-data-deduplication"></a>タスク 3: データ重複除去をテストする

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を開始します。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。

1. **Windows PowerShell** コンソールで、次のコマンドを実行して Windows Admin Center の最新バージョンをダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを実行して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

1. **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` で Windows Admin Center のローカル インスタンスに接続します。 
1. ダイアログが表示されたら、 **[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力し、 **[OK]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. Windows Admin Center で、**sea-svr3.contoso.com** への接続を追加し、パスワード **Pa55w.rd** を使用して **CONTOSO\\Administrator** としてそれに接続します。
1. **sea-svr3.contoso.com** に接続している間に、 **[ツール]** の一覧の **PowerShell** ツールを使用して、重複除去をトリガーする次のコマンドを実行します。

   ```powershell
   Start-DedupJob -Volume M: -Type Optimization –Memory 50
   ```
1. **SEA-SVR3** へのコンソール セッションに切り替えます。
1. **SEA-SVR3** の **Windows PowerShell** プロンプトで次のコマンドを実行して、重複除去されているボリューム上で使用可能なスペースを確認します。

   ```powershell
   Get-PSDrive -Name M
   ```

   > **注**: 前に表示された値と現在の値を比較します。 

1. 5 から 10 分待って重複除去ジョブが完了したら、前の手順を繰り返します。
1. **SEA-ADM1** へのコンソール セッションに切り替えます。
1. **SEA-ADM1** の **sea-svr3.contoso.com** に接続されている Windows Admin Center で、**PowerShell** ツールを使用して、重複除去ジョブの状態を確認する次のコマンドを実行します。

   ```powershell
   Get-DedupStatus –Volume M: | fl
   Get-DedupVolume –Volume M: |fl
   Get-DedupMetadata –Volume M: |fl
   ```
1. **SEA-ADM1** で、**サーバー マネージャー** の [ディスク] ペインを更新し、**M:** ボリュームのプロパティを表示します。
1. **[Volume (M:\) Deduplication Properties]\(ボリューム (M:) の重複除去のプロパティ\)** ウィンドウで、 **[重複除去率]** と **[重複除去による節約量]** の値を確認します。

## <a name="lab-exercise-2-configuring-iscsi-storage"></a>ラボ演習 2: iSCSI 記憶域の構成

### <a name="scenario"></a>シナリオ

Contoso の経営陣は、iSCSI を使用して、一元化された記憶域を構成するコストと複雑さを軽減するオプションを探っています。 これをテストするため、あなたは iSCSI ターゲットをインストールして構成し、ターゲットへのアクセスを提供する iSCSI イニシエーターを構成する必要があります。

この演習の主なタスクは次のとおりです。

1. **SEA-SVR3** に iSCSI をインストールしてターゲットを構成します。
1. **SEA-DC1** (イニシエーター) から iSCSI ターゲットに接続して構成します。
1. iSCSI ディスクの構成を確認します。
1. ディスクの構成を元に戻します。

#### <a name="task-1-install-iscsi-and-configure-targets"></a>タスク 1: iSCSI をインストールしてターゲットを構成する

1. **SEA-ADM1** の **Windows PowerShell** コンソールで次のコマンドを実行して、**SEA-SVR3** への PowerShell リモート処理セッションを確立します。

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR3
   ```
1. 次のコマンドを実行して、**SEA-SVR3** に iSCSI ターゲットをインストールします。

   ```powershell
   Install-WindowsFeature –Name FS-iSCSITarget-Server –IncludeManagementTools
   ```
1. 次のコマンドを実行して、ReFS でフォーマットされた新しいボリュームをディスク 2 に作成します。

   ```powershell
   Initialize-Disk -Number 2
   $partition2 = New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter $partition2.DriveLetter -FileSystem ReFS
   ```
1. 次のコマンドを実行して、ReFS でフォーマットされた新しいボリュームをディスク 3 に作成します。

   ```powershell
   Initialize-Disk -Number 3
   $partition3 = New-Partition -DiskNumber 3 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter $partition3.DriveLetter -FileSystem ReFS
   ```
1. 次のコマンドを実行して、iSCSI トラフィックを許可するセキュリティが強化された Windows Defender ファイアウォール規則を構成します。

   ```powershell
   New-NetFirewallRule -DisplayName "iSCSITargetIn" -Profile "Any" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 3260
   New-NetFirewallRule -DisplayName "iSCSITargetOut" -Profile "Any" -Direction Outbound -Action Allow -Protocol TCP -LocalPort 3260
   ```
1. 次のコマンドを実行して、新しく作成したボリュームに割り当てられたドライブ文字を表示します。

   ```powershell
   $partition2.DriveLetter
   $partition3.DriveLetter
   ```

   > **注**: この手順では、ドライブ文字がそれぞれ **E** および **F** であるとしています。 ドライブ文字の割り当てが異なる場合は、この演習の手順に従うときにそれを考慮してください。

#### <a name="task-2-connect-to-and-configure-iscsi-targets"></a>タスク 2: iSCSI ターゲットに接続して構成する

1. **SEA-ADM1** で、**サーバー マネージャー** の [ディスク] ペインを更新し、**SEA-DC1** のディスク構成を表示します。 ブート ボリュームとシステム ボリュームのドライブ **C** だけが含まれていることに注意してください。
1. **サーバー マネージャー** の **[ファイル サービスと記憶域サービス]** で、[iSCSI] ペインに切り替えます。 
1. [iSCSI] ペインで、次の設定を使用して iSCSI 仮想ディスクを作成します。

   - 記憶域の場所: **E:**
   - 名前: **iSCSIDisk1**
   - ディスク サイズ: **5 GB**、**可変容量**
   - iSCSI ターゲット: **新規**
   - ターゲット名: **iSCSIFarm**
   - アクセス サーバー: **SEA-DC1**

1. 次の設定で 2 つ目の iSCSI 仮想ディスクを作成します。

   - 記憶域の場所: **F:**
   - 名前: **iSCSIDisk2**
   - ディスク サイズ: **5 GB**、**可変容量**
   - iSCSI ターゲット: **iSCSIFarm**

1. **SEA-DC1** のコンソール セッションに切り替え、必要に応じて、**Pa55w.rd** のパスワードを使用して **CONTOSO\\Administrator** としてサインインします。
1. **SEA-SVR3** で **Windows PowerShell** セッションを開始した後、**Windows PowerShell** コンソールで次のコマンドを実行して、iSCSI イニシエーター サービスを開始し、iSCSI イニシエーターの構成を表示します。

   ```powershell
   Start-Service msiscsi
   iscsicpl
   ```

   > **注**: **iscsicpl** コマンドを実行すると、 **[iSCSI イニシエーターのプロパティ]** ウィンドウが開きます。

1. **[iSCSI イニシエーターのプロパティ]** インターフェイスを使用して、次の iSCSI ターゲットに接続します。

   - 名前: **SEA-SVR3**
   - ターゲット名: **iqn.1991-05.com.microsoft:SEA-SVR3-fileserver-target**

#### <a name="task-3-verify-iscsi-disk-configuration"></a>タスク 3: iSCSI ディスクの構成を確認する 

1. **SEA-ADM1** へのコンソール セッションに切り替えます。
1. **サーバー マネージャー** で、 **[ファイル サービスと記憶域サービス]** の [ディスク] ペインを参照し、その表示を更新します。 
1. **SEA-DC1** ディスクの構成を確認し、2 つの **5 GB** ディスクが含まれ、**オフライン** 状態であり、バスの種類が **iSCSI** であることを確認します。
1. **SEA-DC1** のコンソール セッションに切り替えます。 
1. **Windows PowerShell** のプロンプトで次のコマンドを実行して、ディスクの構成を表示します。

   ```powershell
   Get-Disk
   ```
   > **注**: 両方のディスクが存在し、正常ですが、オフラインです。 それらを使用するには、初期化してフォーマットする必要があります。

1. **SEA-DC1** で、**Windows PowerShell** のプロンプトから次のコマンドを実行して、ReFS でフォーマットされたドライブ文字 **E** のボリュームを作成します。

   ```powershell
   Initialize-Disk -Number 1
   New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
   Format-Volume -DriveLetter E -FileSystem ReFS
   ```
1. 前のステップを繰り返して、ReFS でフォーマットされた新しいドライブを作成しますが、今回はディスク番号 **2** とドライブ文字 **F** を使用します。
1. **[サーバー マネージャー]** ウィンドウをアクティブにして **SEA-ADM1** へのコンソール セッションに戻ります。
1. **サーバー マネージャー** で、 **[ファイル サービスと記憶域サービス]** の [ディスク] ペインを更新します。
1. **SEA-DC1** ディスクの構成を確認し、両方のドライブが **オンライン** になっていることを確認します。

#### <a name="task-4-revert-disk-configuration"></a>タスク 4: ディスクの構成を元に戻す 

1. **SEA-SVR3** へのコンソール セッションに切り替えます。
1. **Windows PowerShell** のプロンプトで次のコマンドを実行して、**SEA-SVR3** のディスクを元の状態にリセットします。

   ```powershell
   for ($num = 1;$num -le 4; $num++) {Clear-Disk -Number $num -RemoveData -RemoveOEM -ErrorAction SilentlyContinue}
   for ($num = 1;$num -le 4; $num++) {Set-Disk -Number $num -IsOffline $true}
   ```

   > **注**: これは、次の演習に備えるために必要です。

## <a name="lab-exercise-3-configuring-redundant-storage-spaces"></a>ラボ演習 3: 冗長記憶域スペースの構成

### <a name="scenario"></a>シナリオ

高可用性のいくつかの要件を満たすため、あなたは記憶域スペースでの冗長性オプションを評価することにします。 さらに、記憶域プールへの新しいディスクのプロビジョニングをテストする必要があります。

この演習の主なタスクは次のとおりです。

1. 記憶域プールを作成します。
1. 3 方向ミラー ディスクを基にしてボリュームを作成します。
1. エクスプローラーでボリュームを管理します。
1. 記憶域プールからディスクを切断し、ボリュームの可用性を確認します。
1. 記憶域プールにディスクを追加し、ボリュームの可用性を確認します。
1. ディスクの構成を元に戻します。

> **注:** Windows Server では、記憶域プール内のディスクを切断できません。 削除することだけができます。 また、先に新しいディスクを追加してからでないと、3 方向ミラーからディスクを削除することはできません。 

#### <a name="task-1-create-a-storage-pool"></a>タスク 1: 記憶域プールを作成する 

1. **SEA-ADM1** に切り替えて、**サーバー マネージャー** の **[ファイル サービスと記憶域サービス]** の [ディスク] ペインを更新します。
1. **SEA-SVR3** のディスク 1 から 4 の状態を **オンライン** に設定します。
1. **SEA-SVR3** の記憶域を構成するための **サーバー マネージャー** で、サイズが **127 GB** の 3 つのディスクで構成される **SP1** という名前の新しい記憶域プールを作成します。

#### <a name="task-2-create-a-volume-based-on-a-three-way-mirrored-disk"></a>タスク 2: 3 方向ミラー ディスクを基にしてボリュームを作成する 

1. **SEA-ADM1** の **サーバー マネージャー** から、新しく作成した記憶域プール **SP1** を使用して、ミラー記憶域レイアウトと仮想プロビジョニングを使用し、サイズが **25 GB** の、**Three-Mirror** という名前の仮想ディスクを作成します。
1. 新しくプロビジョニングした仮想ディスクを使用して、**TestData** という名前の ReFS ボリュームを作成し、そのサイズをすべての使用可能なディスク領域に設定して、ドライブ文字 **T** を割り当てます。

#### <a name="task-3-manage-a-volume-in-file-explorer"></a>タスク 3: エクスプローラーでボリュームを管理する 

1. **SEA-ADM1** で、**SEA-SVR3** への PowerShell リモート処理セッションをホストしている **Windows PowerShell** に切り替えます。
1. PowerShell リモート処理セッションを使用して次のコマンドを実行し、セキュリティが強化された Windows Defender ファイアウォールのすべてのファイルとプリンターの共有規則を有効にします。

   ```
   Enable-NetFirewallRule -Group "@FirewallAPI.dll,-28502"
   ```

1. **SEA-ADM1** で **エクスプローラー** を開始し、 **\\\\SEA-SVR3.contoso.com\\t$** 共有を参照します。
1. **TestData** という名前のフォルダーを作成した後、そのフォルダー内に **TestDocument.txt** という名前のドキュメントを作成します。

#### <a name="task-4-disconnect-a-disk-from-the-storage-pool-and-verify-volume-availability"></a>タスク 4: 記憶域プールからディスクを切断し、ボリュームの可用性を確認する 

1. **SEA-ADM1** で **サーバー マネージャー** を使用して、**SEA-SVR3** にアタッチされている残りの使用可能なディスクを記憶域プール **SP1** に追加します。 ディスクで自動割り当てが使用されていることを確認します。
1. **サーバー マネージャー** を使用して、**SP1** プールに割り当てられた最初の 3 つのディスクのいずれかを削除します。
1. **SEA-ADM1** でエクスプローラーを使用して、**TestDocument.txt** がまだ使用可能であることを確認します。 

#### <a name="task-5-add-a-disk-to-the-storage-pool-and-verify-volume-availability"></a>タスク 5: 記憶域プールにディスクを追加し、ボリュームの可用性を確認する 

1. **SEA-ADM1** の **サーバー マネージャー** で、**SP1** 記憶域プールを再スキャンします。
1. 前のタスクで削除したディスクを追加して戻し、自動割り当てが使用されることを確認します。
1. **SEA-ADM1** でエクスプローラーを使用して、**TestDocument.txt** がまだ使用可能であることを確認します。 
1. **SEA-SVR3** に切り替えて戻ります。

#### <a name="task-6-revert-disk-configuration"></a>タスク 6: ディスクの構成を元に戻す 

1. **SEA-SVR3** へのコンソール セッションに切り替えます。
1. **Windows PowerShell** のプロンプトで次のコマンドを実行して、**SEA-SVR3** のディスクを元の状態にリセットします。

   ```powershell
   Get-VirtualDisk -FriendlyName 'Three-Mirror' | Remove-VirtualDisk
   Get-StoragePool -FriendlyName 'SP1' | Remove-StoragePool
   for ($num = 1;$num -le 4; $num++) {Clear-Disk -Number $num -RemoveData -RemoveOEM -ErrorAction SilentlyContinue}
   for ($num = 1;$num -le 4; $num++) {Set-Disk -Number $num -IsOffline $true}
   ```

   > **注**: これは、次の演習に備えるために必要です。

## <a name="lab-exercise-4-implementing-storage-spaces-direct"></a>ラボ演習 4: 記憶域スペース ダイレクトの実装

### <a name="scenario"></a>シナリオ

あなたは、ローカル記憶域を高可用性記憶域として使用する方法が、組織で実行可能なソリューションかどうかをテストする必要があります。 これまで、組織は VM の格納に記憶域ネットワーク (SAN) のみを使用してきました。 Windows Server の機能ではローカル記憶域しか使用できないので、テスト用の実装として記憶域スペース ダイレクトを実装する必要があります。

この演習の主なタスクは次のとおりです。

1. 記憶域スペース ダイレクトのインストールを準備します。
1. フェールオーバー クラスターを作成して検証します。
1. 記憶域スペースを直接有効にします。
1. 記憶域プール、仮想ディスク、共有を作成します。
1. 記憶域スペース ダイレクトの機能を確認します。

#### <a name="task-1-prepare-for-installation-of-storage-spaces-direct"></a>タスク 1: 記憶域スペース ダイレクトのインストールを準備する 

1. 先に進む前に、**SEA-ADM1** の **サーバー マネージャー** で、**SEA-SVR1**、**SEA-SVR2**、**SEA-SVR3** の **[管理状態]** が **[オンライン - パフォーマンス カウンターが開始されていません]** 状態であることを確認します。
1. **サーバー マネージャー** で、 **[ファイル サービスと記憶域サービス]** の [ディスク] ペインを参照します。
1. [ディスク] ペインで **SEA-SVR3** ノードを参照し、ディスク 1 から 4 が **[不明]** と表示されていることを確認します。
1. **サーバー マネージャー** の [ディスク] ペインで、**SEA-SVR1**、**SEA-SVR2**、**SEA-SVR3** にアタッチされているすべてのディスクをオンラインにします。
1. **SEA-ADM1** で、**Windows PowerShell ISE** を開始し、スクリプト ペインで **C:\\Labfiles\\Lab09\\Implement-StorageSpacesDirect.ps1** を開きます。

   > **注**: スクリプトは、番号付きのステップに分かれています。 8 つのステップがあり、各ステップには多数のコマンドがあります。 個々の行を実行するには、その行のどこかにカーソルを置いて、F8 キーを押すか、**Windows PowerShell ISE** ウィンドウのツール バーで **[選択項目を実行]** を選択します。 複数の行を実行するには、それらすべての全体を選択して、F8 キーまたは **[選択項目を実行]** ツール バー アイコンを使用します。 一連の手順については、この演習の手順で説明されています。 各ステップが完了したことを確認してから、次のステップを始めます。

1. ステップ 1 の最初のコマンドを実行して、**SEA-SVR1**、**SEA-SVR2**、**SEA-SVR3** にファイル サーバーの役割とフェールオーバー クラスタリング機能をインストールします。

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。 各コマンドの出力で、**Success** プロパティが **True** に設定されていることを確認します。

1. ステップ 1 の 2 番目のコマンドを実行して、**SEA-SVR1**、**SEA-SVR2**、**SEA-SVR3** を再起動します。

   > **注**: 2 番目のコマンドを呼び出してサーバーを再起動した後、再起動が完了するのを待たずに、3 番目のコマンドを実行して、フェールオーバー クラスタリング管理ツールをインストールできます。

1. ステップ 1 の 3 番目のコマンドを実行して、**フェールオーバー クラスター マネージャー** ツールを **SEA-ADM1** にインストールします。

   > **注**: サーバーが再起動し、**フェールオーバー クラスター マネージャー** ツールが **SEA-ADM1** にインストールされるまで、数分待ちます。

#### <a name="task-2-create-and-validate-the-failover-cluster"></a>タスク 2: フェールオーバー クラスターを作成して検証する 

1. **SEA-ADM1** で、**フェールオーバー クラスター マネージャー** コンソールを開始します。
1. **SEA-ADM1** の **Windows PowerShell ISE** で、ステップ 2 のコマンドを実行してクラスター検証テストを呼び出します。

   > **注**: テストが完了するまで待ちます。 これには 2 分ほどかかります。 テストが失敗していないことを確認します。 警告は、想定されているため無視します。

1. **Windows PowerShell ISE** で、ステップ 3 のコマンドを実行してクラスターを作成します。

   > **注**: ステップが完了するまで待ちます。 これには 2 分ほどかかります。 

1. コマンドが完了したら、**フェールオーバー クラスター マネージャー** に切り替えて、新しく作成した **S2DCluster.Contoso.com** という名前のクラスターを追加します。

#### <a name="task-3-enable-storage-spaces-direct"></a>タスク 3: 記憶域スペース ダイレクトを有効にする

1. **SEA-ADM1** の **Windows PowerShell ISE** で、ステップ 4 のコマンドを実行して、新しくインストールしたクラスターで記憶域スペース ダイレクトを有効にします。

   > **注**: ステップが完了するまで待ちます。 これには 1 分ほどかかります。

1. **Windows PowerShell ISE** で、ステップ 5 のコマンドを実行して、**S2DStoragePool** という名前の記憶域プールを作成します。

   > **注**: ステップが完了するまで待ちます。 これにかかる時間は 1 分未満です。 コマンドの出力で、**FriendlyName** 属性の値が **S2DStoragePool** であることを確認します。

1. **フェールオーバー クラスター マネージャー** に切り替えて、クラスターに **クラスター プール 1** という名前の記憶域プールが含まれていることを確認します。
1. **Windows PowerShell ISE** に切り替えて、ステップ 6 のコマンドを実行して仮想ディスクを作成します。

   > **注**: ステップが完了するまで待ちます。 これにかかる時間は 1 分未満です。 

1. **フェールオーバー クラスター マネージャー** に切り替えて、 **[クラスター仮想ディスク (CSV)]** オブジェクトが [ディスク] ペインに表示されることを確認します。

#### <a name="task-4-create-a-storage-pool-a-virtual-disk-and-a-share"></a>タスク 4: 記憶域プール、仮想ディスク、共有を作成する

1. **SEA-ADM1** の **Windows PowerShell ISE** で、ステップ 7 のコマンドを実行して S2D-SOFS の役割を作成します。

   > **注**: ステップが完了するまで待ちます。 これにかかる時間は 1 分未満です。 

1. **フェールオーバー クラスター マネージャー** に切り替えて、**S2D-SOFS** オブジェクトが [役割] ペインに表示されることを確認します。
1. **Windows PowerShell ISE** に切り替えて、ステップ 8 の 3 つのコマンドをすべて実行して **VM01** 共有を作成します。 
1. **フェールオーバー クラスター マネージャー** に切り替えて、**VM01** 共有が [共有] ペインに表示されることを確認します。

#### <a name="task-5-verify-storage-spaces-direct-functionality"></a>タスク 5: 記憶域スペース ダイレクトの機能を確認する

1. **SEA-ADM1** のエクスプローラーで、 **\\\\s2d-sofs\\VM01** 共有を開き、**VMFolder** という名前のフォルダーを作成します。
1. **SEA-ADM1** で、**Windows PowerShell ISE** のコンソール ペインから次のコマンドを実行して、**SEA-SVR3** をシャットダウンします。

   ```powershell
   Stop-Computer -ComputerName SEA-SVR3 -Force
   ```
1. **サーバー マネージャー** に切り替えて、 **[すべてのサーバー]** ビューを更新し、**SEA-SVR3** にアクセスできなくなったことを確認します。
1. **フェールオーバー クラスター マネージャー** に切り替えて、 **[ディスク]** ノードで **[クラスター仮想ディスク (CSV)]** の情報を確認します。 
1. **[クラスター仮想ディスク (CSV)]** で、 **[正常性状態]** が **[警告]** に設定され、 **[動作状態]** が **[低下]** に設定されていることを確認します ( **[動作状態]** は **[不完全]** になっている場合もあります)。
1. **SEA-ADM1** で、Windows Admin Center が表示されている Microsoft Edge ウィンドウに切り替えます。 
1. Windows Admin Center で、**S2DCluster.Contoso.com** クラスターへの接続を追加します。 

   > **注**: クラスター ノードは、Windows Admin Center で既に使用できるので追加しないでください。 

1. Windows Admin Center のクラスターの [ダッシュボード] ペインで、**SEA-SVR3** に到達できないことを示す警告を確認します。 
1. **SEA-SVR3** へのコンソール セッションに切り替えて、それを開始します。 
1. 数分後に警告が自動的に消えることを確認します。
1. Windows Admin Center が表示されているブラウザー ページを更新し、すべてのサーバーが正常であることを確認します。

### <a name="results"></a>結果

このラボを終えるまでに、次のことを行いました。

- データ重複除去の実装をテストしました。
- iSCSI 記憶域をインストールして構成しました。
- 冗長記憶域スペースを構成しました。
- 記憶域スペース ダイレクトの実装をテストしました。
