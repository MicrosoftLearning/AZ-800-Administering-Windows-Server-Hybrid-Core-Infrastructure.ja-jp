---
lab:
  title: 'ラボ : Azure File Sync の実装'
  type: Answer Key
  module: 'Module 10: Implementing a hybrid file server infrastructure'
ms.openlocfilehash: 63f6e93a55997fb915a2bd89d6cb7b8a56058092
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137906953"
---
# <a name="lab-implementing-azure-file-sync"></a>ラボ : Azure File Sync の実装

## <a name="exercise-1-implementing-distributed-file-system-dfs-replication-in-your-on-premises-environment"></a>演習 1: オンプレミス環境への分散ファイル システム (DFS) レプリケーションの実装

### <a name="task-1-deploy-dfs"></a>タスク 1: DFS をデプロイする

1. **SEA-ADM1** に接続し、必要に応じて、パスワード **Pa55w.rd** を使用し、**CONTOSO\\Administrator** としてサインインします。
1. **SEA-ADM1** の **[スタート]** メニューで、 **[Windows PowerShell]** を選択します。
1. **[Windows PowerShell]** コンソールに次のように入力し、Enter キーを押して、分散ファイル システム (DFS) 管理ツールをインストールします。

   ```powershell
   Install-WindowsFeature -Name RSAT-DFS-Mgmt-Con -IncludeManagementTools
   ```
1. タスク バーで **[エクスプローラー]** を選択します。
1. エクスプローラーで、 **[C:\\Labfiles\\Lab10]** フォルダーにアクセスします。
1. エクスプローラーの詳細ウィンドウで、ファイル **[L10_DeployDFS.ps1]** を選択して状況依存メニューを表示し、そのメニューで **[編集]** を選択します。

   >**注:** これにより、[Windows PowerShell ISE] のスクリプト ペインでファイル **L10_DeployDFS.ps1** が自動的に開きます。

1. **[Windows PowerShell ISE]** のスクリプト ペインでスクリプトを確認した後、ツール バーで **[スクリプトの実行]** アイコンを選択するか、F5 キーを押してスクリプトを実行します。 

### <a name="task-2-test-dfs-deployment"></a>タスク 2: DFS のデプロイをテストする

1. **SEA-ADM1** で **[スタート]** を選択し、「**DFS**」と入力して、 **[DFS 管理]** を選択します。
1. **[DFS 管理]** のナビゲーション ウィンドウで、右クリックするか、 **[名前空間]** のコンテキスト メニューにアクセスして、 **[名前空間の表示]** を選択します。
1. **[名前空間の表示]** ダイアログ ボックスの名前空間の一覧で、 **\\[Contoso.com\Root]** を選択し、 **[OK]** を選択します。
1. ナビゲーション ウィンドウで右クリックするか、 **[レプリケーション]** のコンテキスト メニューにアクセスして、 **[レプリケーション グループの表示]** を選択します。
1. **[レプリケーション グループの表示]** ダイアログ ボックスの **[レプリケーション グループ]** セクションで、 **[Branch1]** を選択し、 **[OK]** を選択します。
1. ナビゲーション ウィンドウで、 **\\[Contoso.com\Root]** 名前空間を展開し、 **[Data]** フォルダーを選択します。
1. 詳細ウィンドウで、**Data** フォルダーに、**SEA-SVR1** と **SEA-SVR2** 上の **Data** フォルダーへの 2 つの参照が含まれていることを確認します。
1. ナビゲーション ウィンドウで、 **[Branch1]** を選択します。
1. 詳細ウィンドウで、**SEA-SVR1** 上と **SEA-SVR2** 上の **S:\\Data** フォルダーが **Branch1** レプリケーション グループのメンバーであることを確認します。

   >**注:** DFS レプリケーションにより、**SEA-SVR1** と **SEA-SVR2** 上の **S:\\Data** フォルダー間で内容がレプリケートされます。

1. エクスプローラーの 2 つのインスタンスを開きます。 最初のエクスプローラー インスタンスで、 **\\\\SEA-SVR1\\Data** に接続し、2 番目のエクスプローラー インスタンスで、 **\\\\SEA-SVR2\\Data** に接続します。
1. **\\\\SEA-SVR1\\Data** に自分の名前を付けた新しいファイルを作成します。
1. 数秒後、自分の名前が付いたファイルが **\\\\SEA-SVR2\\Data** にレプリケートされることを確認します。 これにより、DFS レプリケーションが機能していることが確認されます。

   >**注:** ファイルがレプリケートされ、両方のエクスプローラー ウィンドウに同じ内容が記録されるのを待ちます。

## <a name="exercise-2-creating-and-configuring-a-sync-group"></a>演習 2: 同期グループの作成と構成

### <a name="task-1-create-an-azure-file-share"></a>タスク 1: Azure ファイル共有を作成する

1. **SEA-ADM1** で Microsoft Edge を起動して、 **[Azure portal](https://portal.azure.com)** にアクセスし、このラボで使用するサブスクリプション内の所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。
1. Azure portal で、ツール バーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**ストレージ アカウント**」を検索して選択します。
1. **[ストレージ アカウント]** ページで **[+ 作成]** を選択します。
1. **[ストレージ アカウントの作成]** ページの **[基本]** タブで、次の設定を指定します。

   - [リソース グループ]: **[新規作成]** を選択し、リソース グループ名として「**AZ800-L1001-RG**」と入力し、 **[OK]** を選択します。
   - [ストレージ アカウント名]: 小文字で始まり、小文字と数字の任意の組み合わせが続く文字列を入力します。合計長は 3 から 24 文字です。 

   >**注:** グローバルに一意である可能性が高い名前を選択してください。 たとえば、ストレージ アカウント名を、\<*yourlowercaseinitials*>*DDMMYY* の形式で指定できます。また、たとえば、名前が Devon Torres で、2022 年 1 月 30 日にストレージ アカウントを作成する場合、ストレージ アカウント名を **dt013022** にします。 その名前が既に使用されている場合は、名前が使用できるようになるまで名前に別の文字を追加します。 

   - [リージョン]: ストレージ アカウントを作成できる地理的地域内の Azure リージョン。

   >**注:** このラボでは、すべてのリソースのデプロイに同じリージョンを使用します。

   - [冗長]: **ローカル冗長ストレージ (LRS)**

1. 他のすべての設定については既定値をそのまま使用し、 **[確認と作成]** 、 **[作成]** の順に選択します。
1. ストレージ アカウントが作成された後、 **[デプロイ]** ページで、 **[リソースに移動]** を選択します。
1. **[ストレージ アカウント]** ページで、 **[ファイル共有]** 、 **[+ ファイル共有]** の順に選択します。
1. **[新しいファイル共有]** タブで、 **[名前]** テキスト ボックスに「**share1**」と入力し、 **[作成]** を選択します。

### <a name="task-2-use-an-azure-file-share"></a>タスク 2: Azure ファイル共有を使用する

1. **SEA-ADM1** 上の Azure portal の詳細ウィンドウで、 **[share1]** を選択します。
1. 詳細ウィンドウで、 **[アップロード]** を選択します。
1. **[ファイルのアップロード]** タブで、**C:\\Labfiles\\Lab10\\File1.txt** にアクセスし、 **[アップロード]** を選択します。アップロードが完了したら、 **[ファイルのアップロード]** タブを閉じます。
1. **[share1]** ページで、 **[スナップショット]** 、 **[スナップショットの追加]** 、 **[OK]** の順に選択します。
1. **[share1]** ページで、 **[概要]** 、 **[接続]** の順に選択し、 **[クリップボードにコピー]** ボタンを使用してスクリプトをコピーし、 **[接続]** タブを閉じます。
1. **SEA-ADM1** で、 **[Windows PowerShell ISE]** ウィンドウに切り替え、スクリプト ペインで別のタブを開き、コピーしたスクリプトを貼り付けます。
1. スクリプトの内容を確認し、ツール バーで **[スクリプトの実行]** アイコンを選択するか、F5 キーを押してスクリプトを実行します。 

   >**注:** スクリプトによって、Azure ファイル共有がドライブ文字 **Z** にマウントされました。

1. タスク バーで、右クリックするか、エクスプローラーのコンテキスト メニューにアクセスして、 **[エクスプローラー]** を選択し、 **[クイック アクセス]** テキスト ボックスに「**Z:\** 」と入力して Enter キーを押します。
1. ファイル **File1.txt** が詳細ウィンドウに表示されることを確認します。 これは、Azure ファイル共有にアップロードしたファイルです。
1. **File1.txt** をダブルクリックするか、選択し、Enter キーを押して、メモ帳でファイルを開きます。 
1. メモ帳を使用して、最後の行に自分の名前を追加してファイルの内容を変更し、変更を保存してメモ帳を閉じます。
1. 右クリックするか、 **[File1]** のコンテキスト メニューにアクセスして **[プロパティ]** を選択し、 **[File1.txt のプロパティ]** ウィンドウで **[以前のバージョン]** タブを選択します。
1. 以前のバージョンのファイルが 1 つ使用できることを確認します。 そのバージョン (**File1.txt**) を選択し、 **[復元]** を 2 回選択して、 **[OK]** を 2 回選択します。
1. **File1.txt** をダブルクリックするか、選択し、Enter キーを押して、それに自分の名前が含まれていないことを確認します。 これは、ファイルを変更する前に作成したスナップショットを復元したためです。
1. **[メモ帳]** を閉じます。

### <a name="task-3-deploy-storage-sync-service-and-a-file-sync-group"></a>タスク 3: ストレージ同期サービスと File Sync グループをデプロイする

1. **[SEA-ADM1]** 上の Azure portal のツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、**Azure File Sync** を検索して選択します。
1. **[File Sync のデプロイ]** ページの **[基本]** タブにある **[リソース グループ]** ドロップダウン リストで、 **[AZ800-L1001-RG]** を選択します。 
1. **[ストレージ同期サービス名]** テキスト ボックスに、「**FileSync1**」と入力します。 
1. **[リージョン]** ドロップダウン リストで、ストレージ アカウントを作成したリージョンと同じリージョンを選択します。 
1. **[File Sync のデプロイ]** ページの **[基本]** タブで、 **[確認と作成]** 、 **[作成]** の順に選択します。
1. **[デプロイ]** ブレードで、File Sync がプロビジョニングされたら、 **[リソースに移動]** を選択します。
1. **[FileSync1** **ストレージ同期サービス]** ページで、 **[同期グループ]** 、 **[+ 同期グループ]** の順に選択して、新しい File Sync グループを作成します。
1. **[同期グループ]** ページで、 **[同期グループ名]** テキスト ボックスに「**Sync1**」と入力します。
1. **[ストレージ アカウントの選択]** を選択し、 **[ストレージ アカウントの変更]** ページで、作成したストレージ アカウントを選択します。 

   >**注:** ストレージ アカウントが見つからない場合は、別の Azure リージョンにデプロイされている可能性があります。 ストレージ アカウントが、ストレージ同期サービスと同じリージョンにあることを確認する必要があります。

1. **[Azure ファイル共有]** ドロップダウン リストで、 **[share1]** を選択し、 **[作成]** を選択します。
1. **[ストレージ同期サービス]** ページで、 **[登録済みサーバー]** を選択し、現在登録されているサーバーがないことを確認します。

## <a name="exercise-3-replacing-dfs-replication-with-file-sync-based-replication"></a>演習 3: DFS レプリケーションを File Sync ベースのレプリケーションに置き換える

### <a name="task-1-add-sea-svr1-as-a-server-endpoint"></a>タスク 1: SEA-SVR1 をサーバー エンドポイントとして追加する

1. **SEA-ADM1** 上の Azure portal の **[FileSync1 \| 登録済みサーバー]** ページで、 **[Azure File Sync エージェント]** リンクを選択して、 **[Azure File Sync エージェント]** Microsoft ダウンロード ページに移動します。  
1. **[Azure File Sync エージェント]** Microsoft ダウンロード ページで、 **[ダウンロード]** を選択し、File Sync agent for Windows Server 2022 のエントリ (**StorageSyncAgent_WS2022.msi**) の横にあるチェックボックスをオンにし、 **[次へ]** を選択してダウンロードを開始します。 ダウンロードが完了したら、ダウンロードのために開いた Microsoft Edge のタブを閉じます。
1. エクスプローラーを使用して、ダウンロードしたファイルを **C:\\Labfiles\\Lab10** フォルダーにコピーします。
1. **C:\\Labfiles\\Lab10** フォルダーの内容が表示されているエクスプローラーの詳細ウィンドウで、ファイル **[Install-FileSyncServerCore.ps1]** を選択して、その状況依存メニューを表示し、そのメニューで、 **[編集]** を選択します。

   >**注:** これにより、[Windows PowerShell ISE] のスクリプト ペインで、ファイル **Install-FileSyncServerCore.ps1** が自動的に開きます。

1. **[Windows PowerShell ISE]** のスクリプト ペインでスクリプトを確認した後、ツール バーで **[スクリプトの実行]** アイコンを選択するか、F5 キーを押してスクリプトを実行します。 

   >**注:** スクリプトの実行を監視します。 これには、約 3 分かかります。

1. サインインを求める **警告** メッセージを含むプロンプトが表示されたら、警告メッセージ内の 9 桁のコードをクリップボードにコピーします。
1. Azure portal が表示されている Microsoft Edge ウィンドウに切り替えて、 **+** を選択して新しいタブを開き、新しいタブで **https://microsoft.com/devicelogin** にアクセスします。
1. Microsoft Edge で、 **[コードの入力]** ダイアログ ボックスに、クリップボードにコピーしたコードを貼り付けます。必要に応じて、**Microsoft Azure PowerShell にサインインしますか?** というメッセージが表示されているページで、Azure 資格情報を使用してサインインし、 **[続行]** を選択し、前の手順で開いた Microsoft Edge タブを閉じます。
1. **[Windows PowerShell ISE]** ウィンドウに切り替えて、スクリプトが正常に完了したことを確認します。 
1. Azure portal が表示されている Microsoft Edge ウィンドウに戻り、 **[FileSync1 \| 登録済みサーバー]** ページで **[更新]** を選択して、登録済みサーバーの現在の一覧を表示します。
1. **FileSync1** ストレージ同期サービスの登録済みサーバーの一覧に **SEA-SVR1.Contoso.com** サーバーが表示されていることを確認します。
1. **SEA-ADM1** で、エクスプローラーに切り替えて、 **\\\\SEA-SVR1\\Data** 共有にアクセスし、このフォルダーに現在 **File1.txt** が含まれていないことを確認します。
1. Azure portal が表示されている Microsoft Edge ウィンドウに切り替えて、 **[FileSync1 \| 登録済みサーバー]** ページで、 **[同期グループ]** 、 **[Sync1]** の順に選択し、 **[Sync1]** ページで **[サーバー エンドポイントの追加]** を選択します。
1. **[サーバー エンドポイントの追加]** タブの **[登録済みサーバー]** 一覧で、 **[SEA-SVR1.Contoso.com]** を選択します。
1. **[パス]** テキスト ボックスに「**S:\\Data**」と入力し、 **[作成]** を選択します。
1. エクスプローラー ウィンドウに切り替えて、 **\\\\SEA-SVR1\\Data** フォルダーに **File1.txt** が含まれるようになったことを確認します。

   >**注:** **File1.txt** を Azure ファイル共有にアップロードしました。このファイルは、そこから File Sync によって **SEA-SVR1** に同期されました。

### <a name="task-2-register-sea-svr2-with-file-sync"></a>タスク 2: SEA-SVR2 を File Sync に登録する

1. **SEA-ADM1** で、 **[Windows PowerShell ISE]** ウィンドウに切り替えて、**Install-FileSyncServerCore.ps1** ファイルの内容が表示されているスクリプト ペインのタブに移動します。
1. **[Windows PowerShell ISE]** のスクリプト ペインで、最初の行の `SEA-SVR1` を `SEA-SVR2` に置き換えて、変更を保存し、ツール バーで **[スクリプトの実行]** アイコンを選択するか、F5 キーを押してスクリプトを実行します。 

   >**注:** スクリプトの実行を監視します。 これには、約 3 分かかります。

1. サインインを求める **警告** メッセージを含むプロンプトが表示されたら、警告メッセージ内の 9 桁のコードをクリップボードにコピーします。
1. Azure portal が表示されている Microsoft Edge ウィンドウに切り替えて、 **+** を選択して新しいタブを開き、新しいタブで **https://microsoft.com/devicelogin** にアクセスします。
1. Microsoft Edge で、 **[コードの入力]** ダイアログ ボックスに、クリップボードにコピーしたコードを貼り付けます。必要に応じて、**Microsoft Azure PowerShell にサインインしますか?** というメッセージが表示されているページで、Azure 資格情報を使用してサインインし、 **[続行]** を選択し、前の手順で開いた Microsoft Edge タブを閉じます。
1. **[Windows PowerShell ISE]** ウィンドウに切り替えて、スクリプトが正常に完了したことを確認します。 
1. スクリプトが完了したら、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えて、 **[FileSync1 \| 登録済みサーバー]** ページに戻ります。
1. **SEA-SVR1.Contoso.com** と **SEA-SVR2.Contoso.com** の両方が、**FileSync1** ストレージ同期サービスの登録済みサーバーとして一覧に表示されることを確認します。

### <a name="task-3-remove-dfs-replication-and-add-sea-svr2-as-a-server-endpoint"></a>タスク 3: DFS レプリケーションを削除し、SEA-SVR2 をサーバー エンドポイントとして追加する

1. **SEA-ADM1** のタスク バーで **[DFS 管理]** を選択します。
1. **[DFS 管理]** のナビゲーション ウィンドウで、右クリックするか、 **[Branch1]** のコンテキスト メニューにアクセスして、 **[削除]** 、 **[はい、レプリケーション グループを削除し、関連付けられたすべてのレプリケート フォルダーのレプリケーションを停止して、レプリケーション グループのメンバーをすべて削除します]** オプション、 **[OK]** の順に選択します。
1. Azure portal が表示されている Microsoft Edge ウィンドウに切り替えて、 **[FileSync1** **ストレージ同期サービス]** ページに戻り、同期グループの一覧で **[Sync1]** を選択し、 **[Sync1]** ページで **[サーバー エンドポイントの追加]** を選択します。
1. [サーバー エンドポイントの追加] ウィンドウの **[登録済みサーバー]** 一覧で **[SEA-SVR2.Contoso.com]** を選択し、 **[パス]** テキスト ボックスに「**S:\\Data**」と入力して、 **[作成]** を選択します。

## <a name="exercise-4-verifying-replication-and-enabling-cloud-tiering"></a>演習 4: レプリケーションの検証と、クラウドを使った階層化の有効化

### <a name="task-1-verify-file-sync"></a>タスク 1: File Sync を検証する

1. **SEA-ADM1** で、 **\\\\SEA-SVR1\\Data** 共有が表示されているエクスプローラー ウィンドウに切り替えます。 
1. **\\\\SEA-SVR1\\Data** フォルダーに、任意の名前を付けた別のファイルを作成します。
1. **SEA-ADM1** で、 **\\\\SEA-SVR2\\Data** 共有の内容が表示されているエクスプローラー ウィンドウに切り替えた直後に、同じ名前を持つファイルが **\\\\SEA-SVR2\\Data** フォルダー内にも表示されることを確認します。

   >**注** 前の演習で DFS レプリケーションを削除しました。これは、File Sync によって、新しく作成されたファイルがレプリケートされたことを意味します。

### <a name="task-2-enable-cloud-tiering"></a>タスク 2: クラウドを使った階層化を有効にする

1. **SEA-ADM1** 上の Azure portal の **[Sync1]** 同期グループ ページにある **[サーバー エンドポイント]** セクションで、 **[SEA-SVR2.Contoso.com]** を選択します。
1. [サーバー エンドポイントのプロパティ] ウィンドウの **[クラウドを使った階層化]** セクションで、 **[有効]** を選択します。
1. **[指定した割合の空きスペースをボリュームに常時残す]** テキスト ボックスに「**90**」と入力し、 **[日付ポリシー]** を **[有効]** に設定します。 **[指定した日数内にアクセスまたは変更されたファイルのみをキャッシュする]** テキスト ボックスに「**14**」と入力し、 **[保存]** を選択します。

   >**注:** しばらくすると、**SEA-SVR2** 上のファイルが自動的に階層化されます。 このプロセスは、PowerShell を使用してトリガーします。

1. **SEA-ADM1** で、 **[Windows PowerShell ISE]** ウィンドウに切り替えます。
1. **[Windows PowerShell ISE]** のコンソール ウィンドウで、次のコマンドを入力し、各コマンドの後に Enter キーを押すと、階層化が即座にトリガーされます。

   ```powershell
   Enter-PSSession -computername SEA-SVR2
   fsutil file createnew S:\Data\report1.docx 254321098
   fsutil file createnew S:\Data\report2.docx 254321098
   fsutil file createnew S:\Data\report3.docx 254321098
   fsutil file createnew S:\Data\report4.docx 254321098
   Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
   Invoke-StorageSyncCloudTiering -Path S:\Data 
   ```
1. **SEA-ADM1** で、 **\\\\SEA-SVR2\\Data** フォルダーの内容が表示されているエクスプローラー ウィンドウに切り替えます。
1. エクスプローラー ウィンドウの詳細ウィンドウで、右クリックするか、**タイトル** 列のコンテキスト メニューにアクセスして、詳細ウィンドウに **[属性]** 列を追加します。たとえば、 **[名前]** 列で、 **[その他]** を選択し、 **[属性]** チェックボックスをオンにし、 **[OK]** を選択します。
1. **[属性]** 列を **[名前]** 列の横にドラッグし、ファイルの日付と属性をメモします。
1. 属性が **[L]** 、 **[M]** 、 **[O]** であるファイルを確認します。これらは、階層化が行われたことを示します。 

## <a name="exercise-5-troubleshooting-replication-issues"></a>演習 5: レプリケーション問題のトラブルシューティング

### <a name="task-1-monitor-file-sync-replication"></a>タスク 1: File Sync レプリケーションを監視する

1. **SEA-ADM1** で、エクスプローラーを使用して **C:\\Windows\\INF** フォルダーを **\\\\SEA-SVR2\\Data\\** にコピーします。 このフォルダーはクラウド エンドポイントと同期され、同期トラフィックが発生します。
1. **SEA-ADM1** で、**FileSync1** ストレージ同期サービスの **Sync1** 同期グループ ページが表示されている Azure portal に切り替えます。
1. **[サーバー エンドポイント]** セクションで、両方のエンドポイントの **[正常性]** に緑色のチェック マークが付いていることを確認します。
1. [サーバー エンドポイントのプロパティ] ウィンドウで **[SEA-SVR2.Contoso.com]** エンドポイントを選択し、 **[同期アクティビティ]** を確認し、ウィンドウを閉じます。
1. **[同期されたファイル数]** グラフを選択し、フィルターを使用してグラフをカスタマイズする方法を調べます。
1. Azure ファイル共有にマップされたドライブ **Z** の内容が表示されているエクスプローラー ウィンドウに切り替えて、ドライブに **\\\\SEA-SVR2\\Data** から同期された **INF** フォルダーの内容が含まれていることを確認します。
1. Azure portal に切り替えて、**INF** 同期トラフィックが、 **[同期されたファイル数]** および **[同期されたバイト数]** のグラフに反映されていることを確認します。 **INF** フォルダーには 800 個以上のファイルが含まれており、そのサイズは 40 MB を超えます。

   >**注:** 更新された統計を確認するには、Azure portal が表示されているページを更新することが必要な場合があります。

### <a name="task-2-test-replication-conflict-resolution"></a>タスク 2: レプリケーションの競合の解決をテストする

1. **SEA-ADM1** で、 **\\\\SEA-SVR1\Data\\** と **\\\\SEA-SVR2\Data\\** の内容が並べて表示されているエクスプローラー ウィンドウに移動します。
1. **\\\\SEA-SVR1\Data\\** の内容が表示されているエクスプローラー ウィンドウで、**Demo.txt** という名前のファイルを作成します。 
1. **\\\\SEA-SVR2\Data\\** の内容が表示されているエクスプローラー ウィンドウで、**Demo.txt** という名前のファイルを作成します。 
1. 最初の **Demo.txt** ファイルに任意のテキストを追加し、変更を保存します。
1. その後すぐに、2 番目の **Demo.txt** ファイルに任意のテキスト (前の手順で使用したものとは異なるテキスト) を追加し、変更を保存します。

   >**注:** 2 番目のファイルに対する変更をできる限り早く保存してください。 同期の競合を意図的にトリガーするために、名前は同一で、内容が異なるファイルを作成しています。

1. 各エクスプローラー ウィンドウで、内容を確認し、**Demo.txt** ファイルだけでなく、**Demo-SEA-SVR2.txt** (および場合によっては **Demo-Cloud.txt**) も含まれていることを確認します。 

   >**注:** これは、File Sync によって同期の競合が検出され、エンドポイント名を表すサフィックス (**SEA-SVR2**) または **Cloud** が、競合が発生したファイルに追加されたためです。

   >**注:** 同期の競合が発生するまで数分待つ必要がある場合があります。

## <a name="exercise-6-cleaning-up-the-azure-subscription"></a>演習 6: Azure サブスクリプションのクリーンアップ

### <a name="task-1-delete-the-azure-resources-that-were-created-in-the-lab"></a>タスク 1: ラボで作成された Azure リソースを削除する

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えて、 **[FileSync1 ストレージ同期サービス]** ページにアクセスします。
1. **[ストレージ同期サービス]** ページで、 **[登録済みサーバー]** を選択します。
1. 詳細ウィンドウで、右クリックするか、 **[SEA-SVR2.Contoso.com]**  のコンテキスト メニューにアクセスして、 **[サーバーの登録解除]** を選択します。
1. [サーバーの登録解除] ウィンドウで、テキスト ボックスに 「**SEA-SVR2.Contoso.com**」と入力し、 **[登録解除]** を選択します。
1. [ストレージ同期サービス] ウィンドウで、 **[登録済みサーバー]** を選択します。
1. 詳細ウィンドウで、右クリックするか、 **[SEA-SVR1.Contoso.com]** のコンテキスト メニューにアクセスして、 **[サーバーの登録解除]** を選択します。 
1. [サーバーの登録解除] ウィンドウで、テキスト ボックスに「**SEA-SVR1.Contoso.com**」と入力し、 **[登録解除]** を選択します。
1. 両方のサーバーの登録が削除されるまで待ちます。
1. [ストレージ同期サービス] ウィンドウで **[同期グループ]** を選択し、詳細ウィンドウで **[Sync1]** を選択します。
1. Sync1 ウィンドウの **クラウド エンドポイント** セクションで、右クリックするか、**share1** のコンテキスト メニューにアクセスして、**削除**、**OK** の順に選択します。
1. **[share1]** が削除されるまで待ちます。
1. **[削除]** 、 **[OK]** の順に選択します。
1. ナビゲーション ウィンドウで、 **[すべてのリソース]** を選択します。
1. 詳細ウィンドウで、 **[FileSync1]** と、このラボで作成した Azure ストレージ アカウントを選択します。
1. [リソースの削除] ウィンドウで、 **[削除]** を選択し、テキスト ボックスに「**はい**」と入力し、 **[削除]** を選択します。
1. ナビゲーション ウィンドウで、 **[リソース グループ]** を選択します。
1. 詳細ウィンドウで、 **[AZ800-L1001-RG]** 、 **[リソース グループの削除]** の順に選択し、「**AZ800-L1001-RG**」と入力して、 **[削除]** を選択します。