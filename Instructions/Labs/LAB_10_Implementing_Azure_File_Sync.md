---
lab:
  title: 'ラボ : Azure File Sync の実装'
  module: 'Module 10: Implementing a hybrid file server infrastructure'
---

# <a name="lab-implementing-azure-file-sync"></a>ラボ: Azure File Sync の実装

## <a name="scenario"></a>シナリオ

Contoso のロンドン本社とシアトルを拠点とするブランチ オフィスとの間の分散ファイル システム (DFS) レプリケーションに関する懸念に対処するために、2 つのオンプレミス ファイル共有間で行うレプリケーションの代替メカニズムとして Azure File Sync をテストすることにしました。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- オンプレミス環境で DFS レプリケーションを実装する。
- 同期グループを作成して構成する。
- DFS レプリケーションを Azure File Sync ベースのレプリケーションに置き換える。
- レプリケーションを確認し、クラウドを使った階層化を有効にする。
- レプリケーションの競合のトラブルシューティングを行う。

## <a name="estimated-time-60-minutes"></a>予想所要時間: 60 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-SEA-SVR2**、**AZ-800T00A-ADM1** が稼働している必要があります。 

> **注**: **AZ-800T00A-SEA-DC1**、**AZ-800T00A-SEA-SVR1**、**AZ-800T00A-SEA-SVR2**、**AZ-800T00A-SEA-ADM1** 仮想マシンはそれぞれ、**SEA-DC1**、**SEA-SVR1**、**SEA-SVR2**、**SEA-ADM1**仮想マシンのインストールをホストしています。

1. **SEA-ADM1** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

For this lab, you'll use the available VM environment and an Azure subscription. Before you begin the lab, ensure that you have an Azure subscription and a user account with the Owner or Contributor role in that subscription.

## <a name="exercise-1-implementing-dfs-replication-in-your-on-premises-environment"></a>演習 1: オンプレミス環境での DFS レプリケーションの実装

### <a name="scenario"></a>シナリオ

演習シナリオ: オンプレミスの DFS レプリケーション移行のテストを開始する前に、まず、**SEA-SVR1** および **SEA-SVR2** の概念実証環境に DFS レプリケーションを実装する必要があります。

この演習の主なタスクは次のとおりです。

1. DFS をデプロイする。
1. DFS のデプロイをテストする。

#### <a name="task-1-deploy-dfs"></a>タスク 1: DFS をデプロイする

1. **SEA-ADM1**で、管理者として Windows PowerShell を起動し、次のコマンドを実行して分散ファイル システム (DFS) 管理ツールをインストールします。

   ```powershell
   Install-WindowsFeature -Name RSAT-DFS-Mgmt-Con -IncludeManagementTools
   ```
1. **SEA-ADM1**で、エクスプローラーを開き、**C:\\Labfiles\\Lab10** フォルダーにアクセスして、新しい **[Windows PowerShell ISE]** ウィンドウで **L10-DeployDFS.ps1** を開きます。
1. スクリプト ペインでスクリプトを確認し、それを実行して、サンプル DFS 名前空間と DFS レプリケーション グループを作成します。

#### <a name="task-2-test-dfs-deployment"></a>タスク 2: DFS のデプロイをテストする

1. **SEA-ADM1**で、 **[DFS 管理]** コンソールを起動し、前のタスクで作成した **\\\\Contoso.com\\Root\\** 名前空間と **Branch1** レプリケーション グループの両方を追加します。 
1. Verify that the <bpt id="p1">**</bpt><ph id="ph1">\\</ph><ph id="ph2">\\</ph>Contoso.com<ph id="ph3">\\</ph>Root<ph id="ph4">\\</ph>Data<ept id="p1">**</ept> folder has targets on <bpt id="p2">**</bpt>SEA-SVR1<ept id="p2">**</ept> and <bpt id="p3">**</bpt>SEA-SVR2<ept id="p3">**</ept>. Note the folders configured as the targets.
1. Verify that the <bpt id="p1">**</bpt>Branch1<ept id="p1">**</ept> replication group has two members, <bpt id="p2">**</bpt>SEA-SVR1<ept id="p2">**</ept> and <bpt id="p3">**</bpt>SEA-SVR2<ept id="p3">**</ept>. Note the folders replicated on each server.
1. Open two instances of File Explorer. In the first instance, connect to <bpt id="p1">**</bpt><ph id="ph1">\\</ph><ph id="ph2">\\</ph>SEA-SVR1<ph id="ph3">\\</ph>Data<ept id="p1">**</ept>, and then in the second instance, connect to <bpt id="p2">**</bpt><ph id="ph4">\\</ph><ph id="ph5">\\</ph>SEA-SVR2<ph id="ph6">\\</ph>Data<ept id="p2">**</ept>.
1. Create a new file with your name in <bpt id="p1">**</bpt><ph id="ph1">\\</ph><ph id="ph2">\\</ph>SEA-SVR1<ph id="ph3">\\</ph>Data<ept id="p1">**</ept>, and then confirm that the file replicates to <bpt id="p2">**</bpt><ph id="ph4">\\</ph><ph id="ph5">\\</ph>SEA-SVR2<ph id="ph6">\\</ph>Data<ept id="p2">**</ept> after a few seconds. This confirms that DFS Replication is working.

   >**注:** ファイルがレプリケートされ、両方のエクスプローラー ウィンドウに同じ内容が記録されるのを待ちます。

### <a name="results"></a>結果

After completing this exercise, you'll have created a working DFS infrastructure. This includes DFS Replication, which replicates content between <bpt id="p1">**</bpt>SEA-SVR1<ept id="p1">**</ept> and <bpt id="p2">**</bpt>SEA-SVR2<ept id="p2">**</ept>.

## <a name="exercise-2-creating-and-configuring-a-sync-group"></a>演習 2: 同期グループの作成と構成

### <a name="scenario"></a>シナリオ

DFS レプリケーション環境を File Sync に移行するための準備として、最初に File Sync グループを作成して構成する必要があります。

この演習の主なタスクは次のとおりです。

1. Azure ファイル共有を作成します。
1. Azure ファイル共有を使用します。
1. ストレージ同期サービスと File Sync グループをデプロイします。

#### <a name="task-1-create-an-azure-file-share"></a>タスク 1: Azure ファイル共有を作成する

1. **SEA-ADM1** で Microsoft Edge を起動し、Azure portal にアクセスして、Azure 資格情報で認証します。
1. Azure portal で、**AZ800-L1001-RG** という名前のリソース グループ内に、**ローカル冗長ストレージ (LRS)** を指定して Azure ストレージ アカウントを作成します。

   >**注:** このラボでは、すべての Azure リソースのデプロイに同じリージョンを使用します。

1. ストレージ アカウントで、**share1** という名前のファイル共有を作成します。

#### <a name="task-2-use-an-azure-file-share"></a>タスク 2: Azure ファイル共有を使用する

1. **SEA-ADM1** で、**C:\\Labfiles\\Lab10\\File1.txt** ファイルを **share1** にアップロードします。
1. Azure portal で、**share1** のスナップショットを作成します。
1. **SEA-ADM1** で、Azure portal によって提供される接続スクリプトを使用して、**share1** をドライブ **Z** としてマウントします。
1. エクスプローラーで、マウントされたドライブ上の **File1.txt** という名前のファイルを開き、自分の名前を入力し、ファイルを保存します。
1. エクスプローラーの **[以前のバージョン]** を使用して、**File1.txt** の以前のバージョンを復元します。
1. **File1.txt** を開き、自分の名前が含まれていないことを確認します。

#### <a name="task-3-deploy-storage-sync-service-and-a-file-sync-group"></a>タスク 3: ストレージ同期サービスと File Sync グループをデプロイする

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, use the Azure portal to create an Azure File Sync resource named <bpt id="p2">**</bpt>FileSync1<ept id="p2">**</ept>. Use the same region and Resource Group as you used when deploying the storage account.

   >**注:** File Sync をデプロイすると、ストレージ同期サービス リソースが作成されます。

1. Create a sync group named <bpt id="p1">**</bpt>Sync1<ept id="p1">**</ept> in the <bpt id="p2">**</bpt>FileSync1<ept id="p2">**</ept> Storage Sync Service. Use the storage account that you created earlier and <bpt id="p1">**</bpt>share1<ept id="p1">**</ept> as the Azure file share when creating <bpt id="p2">**</bpt>Sync1<ept id="p2">**</ept>.
1. 現在、**FileSync1**に登録されているサーバーがないことを確認します。

### <a name="results"></a>結果

After completing this exercise, you will have created a File Sync group. You also have created the cloud endpoint mapped on <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept> so that you can inspect the Azure file share content.

## <a name="exercise-3-replacing-dfs-replication-with-file-sync-based-replication"></a>演習 3: DFS レプリケーションを File Sync ベースのレプリケーションに置き換える

### <a name="scenario"></a>シナリオ

必要なコンポーネントがすべて準備できたので、DFS レプリケーションを File Sync ベースのレプリケーションに置き換えます。

この演習の主なタスクは次のとおりです。

1. **SEA-SVR1** をサーバー エンドポイントとして追加する。
1. **SEA-SVR2** を File Sync に登録する。
1. DFS レプリケーションを削除し、**SEA-SVR2** をサーバー エンドポイントとして追加する。

#### <a name="task-1-add-sea-svr1-as-a-server-endpoint"></a>タスク 1: SEA-SVR1 をサーバー エンドポイントとして追加する

1. **SEA-ADM1** の Azure portal で、File Sync agent for Windows Server 2022 (**StorageSyncAgent_WS2022.msi**) をダウンロードし、**C:\\\\Labfiles\\Lab10** フォルダーに保存します。
1. **SEA-ADM1** のエクスプローラーで、**C:\\Labfiles\\Lab10** フォルダーにアクセスし、[Windows PowerShell ISE] ウィンドウのスクリプト ペインで **Install-FileSyncServerCore.ps1** を開きます。
1. **[Windows PowerShell ISE]** のスクリプト ペインで、スクリプトを確認して実行し、File Sync エージェントを **SEA-SVR1**にインストールします。
1. メッセージが表示されたら、Azure サブスクリプションに対する認証を行います。 
1. Azure portal で、**FileSync1** ストレージ同期サービスの登録済みサーバーを更新し、**SEA-SVR1.Contoso.com** が登録されたことを示します。
1. エクスプローラーで **\\\\SEA-SVR1\\Data** を開き、フォルダーに **File1.txt** が含まれていないことを示します。
1. Azure portal を使用して、**SEA-SVR2.Contoso.com**  上の **S:\\Data** をサーバー エンドポイントとして **Sync1** に追加します。
1. エクスプローラーを使用して、**File1.txt** が **\\\\SEA-SVR1\\Data\\** で使用できることを確認します。

   >**注:** **File1.txt** を **File1.txtAzure** ファイル共有にアップロードしました。このファイルは、そこから File Sync によって **SEA-SVR2** に同期されました。

#### <a name="task-2-register-sea-svr2-with-file-sync"></a>タスク 2: SEA-SVR2 を File Sync に登録する

1. **SEA-ADM1** の **Install-FileSyncServerCore.ps1** スクリプトが表示されている[Windows PowerShell ISE] ウィンドウで、`SEA-SVR1` を `SEA-SVR2` に置き換えて変更を保存します。
1. **C:\\Labfiles\\Lab10\\Install-FileSyncServerCore.ps1** を実行して、File Sync エージェントを **SEA-SVR2** にインストールします。
1. メッセージが表示されたら、Azure サブスクリプションに対する認証を行います。 
1. Azure portal を使用して、**SEA-SVR2.Contoso.com** と **SEA-SVR1.Contoso.com** が **FileSync1** ストレージ同期サービスに登録されていることを確認します。

#### <a name="task-3-remove-dfs-replication-and-add-sea-svr2-as-a-server-endpoint"></a>タスク 3: DFS レプリケーションを削除し、SEA-SVR2 をサーバー エンドポイントとして追加する

1. **SEA-ADM1** で、**DFS 管理**を使用して **Branch1** レプリケーション グループを削除します。
1. Azure portal を使用して、**SEA-SVR2.Contoso.com**  上の **S:\\Data** をサーバー エンドポイントとして **Sync1** に追加します。

### <a name="results"></a>結果

この演習を完了すると、DFS レプリケーションが File Sync に置き換えられます。

## <a name="exercise-4-verifying-replication-and-enabling-cloud-tiering"></a>演習 4: レプリケーションの検証と、クラウドを使った階層化の有効化

### <a name="scenario"></a>シナリオ

演習シナリオ: 次に、DFS レプリケーションが File Sync に正常に置き換えられたことを確認し、それを確認した後、クラウドを使った階層化を有効にする必要があります。

この演習の主なタスクは次のとおりです。

1. File Sync を検証する。
1. クラウドを使った階層化を有効にする。

#### <a name="task-1-verify-file-sync"></a>タスク 1: File Sync を検証する

1. **SEA-ADM1** で、エクスプローラーの 2 つのインスタンスを使用して、 **\\\\SEA-SVR1\\Data** と **\\\\SEA-SVR2\\Data** の内容を表示します。
1. **\\\\SEA-SVR1\\Data** に、任意の名前を持つファイルを作成します。
1. 直後に、同じ名前を持つファイルが **\\\\SEA-SVR2\\Data** フォルダーに表示されることを確認します。

   >**注** 前の演習で DFS レプリケーションを削除しました。これは、File Sync によって、新しく作成されたファイルがレプリケートされたことを意味します。

#### <a name="task-2-enable-cloud-tiering"></a>タスク 2: クラウドを使った階層化を有効にする

1. **SEA-ADM1** で、Azure portal を使用して、**FileSync1** ストレージ同期サービス内の **Sync1** 同期グループにアクセスします。
1. In the Azure portal, enable cloud tiering for the <bpt id="p1">**</bpt>SEA-SVR1.Contoso.com<ept id="p1">**</ept> endpoint in <bpt id="p2">**</bpt>Sync1<ept id="p2">**</ept>. Set the <bpt id="p1">**</bpt>free disk space<ept id="p1">**</ept> policy to <bpt id="p2">**</bpt>80<ept id="p2">**</ept> percent and the <bpt id="p3">**</bpt>date policy<ept id="p3">**</ept> to cache files that were accessed in the last <bpt id="p4">**</bpt>7<ept id="p4">**</ept> days.
1. **\\\\SEA-SVR1\\Data** フォルダーに接続されているエクスプローラー インスタンスの詳細ウィンドウで、右クリックするか、**タイトル** 列のコンテキスト メニューにアクセスして、 **[属性]** 列を追加します。たとえば、 **[名前]** 列で、 **[その他]** 、 **[属性]** の順に選択します。

   ><bpt id="p1">**</bpt>Note:<ept id="p1">**</ept> After some time, files on <bpt id="p2">**</bpt>SEA-SVR2<ept id="p2">**</ept> would be automatically tiered. You will trigger this process by using PowerShell.

1. **SEA-ADM1** で、 **[Windows PowerShell ISE]** のコンソール ウィンドウから、次のコマンドを実行すると、階層化が即座にトリガーされます。

   ```powershell
   Enter-PSSession -computername SEA-SVR2
   fsutil file createnew S:\Data\report1.docx 254321098
   fsutil file createnew S:\Data\report2.docx 254321098
   fsutil file createnew S:\Data\report3.docx 254321098
   fsutil file createnew S:\Data\report4.docx 254321098
   Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
   Invoke-StorageSyncCloudTiering -Path S:\Data 
   ```
1. **SEA-ADM1** でエクスプローラーに切り替えて、 **\\\\SEA-SVR2\\Data** 共有で、属性が **[L]** 、 **[M]** 、 **[O]** であるファイルを識別します (これは、階層化が行われたことを示します)。

### <a name="results"></a>結果

この演習を完了すると、機能する File Sync レプリケーションが作成され、クラウドを使った階層化が構成されます。

## <a name="exercise-5-troubleshooting-replication-issues"></a>演習 5: レプリケーション問題のトラブルシューティング

### <a name="scenario"></a>シナリオ

Exercise scenario: Contoso relies heavily on its DFS Replication implementation. You must ensure that any replication issues, including replication conflicts, can be quickly identified and resolved. To do so, you'll simulate the most common replication issues in your proof-of-concept environment and test their resolutions.

この演習の主なタスクは次のとおりです。

1. File Sync レプリケーションを監視する。
1. レプリケーションの競合の解決をテストする。

#### <a name="task-1-monitor-file-sync-replication"></a>タスク 1: File Sync レプリケーションを監視する

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, use File Explorer to copy the <bpt id="p2">**</bpt>C:<ph id="ph1">\\</ph>Windows<ph id="ph2">\\</ph>INF<ept id="p2">**</ept> folder to <bpt id="p3">**</bpt><ph id="ph3">\\</ph><ph id="ph4">\\</ph>SEA-SVR1\Data<ph id="ph5">\\</ph><ept id="p3">**</ept>. The folder will sync to the cloud endpoint, causing sync traffic.
1. Azure portal で、**FileSync1** ストレージ同期サービスの **Sync1** 同期グループにアクセスします。
1. **[サーバー エンドポイント]** セクションで、両方のエンドポイントの **[正常性]** を確認します。
1. **SEA-SVR1.Contoso.com** エンドポイントを選択し、[サーバー エンドポイントのプロパティ] ウィンドウで、 **[同期アクティビティ]** を確認します。
1. **[同期されたファイル数]** グラフを選択し、フィルターを使用してグラフをカスタマイズする方法を調べます。
1. **INF** フォルダーがドライブ **Z**に同期中であることを確認します。
1. In the Azure portal, verify that the <bpt id="p1">**</bpt>INF<ept id="p1">**</ept> sync traffic is reflected in the <bpt id="p2">**</bpt>Files Synced<ept id="p2">**</ept> and <bpt id="p3">**</bpt>Bytes Synced<ept id="p3">**</ept> graphs. The <bpt id="p1">**</bpt>INF<ept id="p1">**</ept> folder has more than 800 files, and its size is more than 40 megabytes (MB).

   >**注:** 更新された統計を確認するには、Azure portal が表示されているページを更新することが必要な場合があります。

#### <a name="task-2-test-replication-conflict-resolution"></a>タスク 2: レプリケーションの競合の解決をテストする

1. **SEA-ADM1** で、**\\\\SEA-SVR1\Data\\** と **\\\\SEA-SVR2\Data\\** の内容が並べて表示されているエクスプローラー ウィンドウにアクセスします。
1. **\\\\SEA-SVR1\Data\\** の内容が表示されているエクスプローラー ウィンドウで、**Demo.txt** という名前のファイルを作成します。 
1. **\\\\SEA-SVR2\Data\\** の内容が表示されているエクスプローラー ウィンドウで、**Demo.txt** という名前のファイルを作成します。 
1. 最初の **Demo.txt** ファイルに任意のテキストを追加し、変更を保存します。
1. その後すぐに、2 番目の **Demo.txt** ファイルに任意のテキスト (前の手順で使用したものとは異なるテキスト) を追加し、変更を保存します。

   ><bpt id="p1">**</bpt>Note:<ept id="p1">**</ept> Make sure to save the change to the second file as soon as possible. You're creating files with the same name but different content to intentionally trigger a sync conflict.

1. 各エクスプローラー ウィンドウで、内容を確認し、**Demo.txt** ファイルだけでなく、**Demo-SEA-SVR2.txt** ファイル (および場合によっては **Demo-Cloud.txt**) も含まれていることを確認します。 

   >**注:** これは、File Sync によって同期の競合が検出され、エンドポイント名を表すサフィックス (**SEA-SVR2**) または **Cloud** が、競合が発生したファイルに追加されたためです。

### <a name="results"></a>結果

この演習を完了すると、File Sync レプリケーションを監視し、レプリケーションの問題が解決されます。

## <a name="exercise-6-cleaning-up-the-azure-subscription"></a>演習 6: Azure サブスクリプションのクリーンアップ

演習シナリオ: Azure 関連の料金を最小限に抑えるために、Azure サブスクリプションをクリーンアップします。

#### <a name="task-1-delete-the-azure-resources-that-were-created-in-the-lab"></a>タスク 1: ラボで作成された Azure リソースを削除する

1. **SEA-ADM1** で、Azure portal を使用して、 **[FileSync1 ストレージ同期サービス]** ページにアクセスします。
1. 登録済みサーバーとしての **SEA-SVR1.Contoso.com** および **SEA-SVR2.Contoso.com** を削除します。
1. **Sync1** 同期グループ内の **share1** クラウド エンドポイントを削除します。
1. **Sync1** 同期グループを削除します。
1. **FileSync1** ストレージ同期サービスと、ラボで作成した Azure ストレージ アカウントを削除します。
1. **AZ800-L1001-RG** リソース グループを削除します。

### <a name="results"></a>結果

この演習を完了すると、このラボで作成した Azure リソースがクリーンアップされます。
