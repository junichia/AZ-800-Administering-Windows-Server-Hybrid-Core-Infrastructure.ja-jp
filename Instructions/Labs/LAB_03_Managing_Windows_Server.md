---
lab:
  title: 'ラボ: Windows Server の管理'
  module: 'Module 3: Windows Server administration'
---

# <a name="lab-managing-windows-server"></a>ラボ: Windows Server の管理

## <a name="scenario"></a>シナリオ

Contoso, Ltd. は、自社の環境に新しいサーバーをいくつか新規に実装することを望んでいて、Server Core を使用することを決定しています。 また、組織内のこれらのサーバーとその他のサーバーの両方をリモートで管理するために Windows Admin Center を実装することも希望しています。

## <a name="objectives"></a>目標

- Windows Admin Center を実装および構成する

## <a name="estimated-time-45-minutes"></a>予想所要時間: 45 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1** および **AZ-800T00A-ADM1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-800T00A-SEA-DC1** と **AZ-800T00A-SEA-ADM1** の各仮想マシンが **SEA-DC1** と **SEA-ADM1** のインストールをホストしています。

1. **SEA-ADM1** を選択します。

1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、使用可能な VM 環境と Azure AD テナントを使用します。 

## <a name="exercise-1-implementing-and-using-remote-server-administration"></a>演習 1: リモート サーバー管理の実装と使用

### <a name="scenario"></a>シナリオ 

Server Core サーバーをデプロイしたので、リモート管理のために Windows Admin Center を実装する必要があります。

この演習の主なタスクは次のとおりです。

1. Windows Admin Center をインストールします。
1. リモート管理用のサーバーを追加します。
1. Windows Admin Center 拡張機能を構成します。
1. リモート管理を確認します。
1. リモート PowerShell を使用してサーバーを管理します。

#### <a name="task-1-install-windows-admin-center"></a>タスク 1: Windows Admin Center をインストールする

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を開始します。
1. **Windows PowerShell** コンソールで、次のコマンドを実行して Windows Admin Center の最新バージョンをダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

   > **注**:インストールが完了すると、"ERR_Connection_Refused" というエラー メッセージが表示されることがあります。 これが発生した場合は、SEA-ADM1 を再起動して問題を修正します。

1. インストールが完了したら サーバーを再起動してください。再起動には少し時間がかかことがあります（15分程度）。

#### <a name="task-2-add-servers-for-remote-administration"></a>タスク 2: リモート管理用のサーバーを追加する

1. **SEA-ADM1** にログインし、Microsoft Edge を開始して、`https://SEA-ADM1.contoso.com` にアクセスします。 

1. メッセージが表示されたら、ユーザー名として **CONTOSO\\Administrator** を、パスワードとして **Pa55w.rd** を使用してサインインします。

1. **[All Connections/すべての接続]** ページを確認します。**sea-adm1.contoso.com** エントリ（自分自身）が含まれていることに注目します。 

1. [All Connections/すべての接続] ウィンドウで、**Add** をクリックして **Servers** の **Add** から `sea-dc1.contoso.com` に接続を追加します。

1. メッセージが表示されたら、ユーザー名として **CONTOSO\\Administrator** を、パスワードとして **Pa55w.rd** を使用してサインインします。

   > **注**: シングル サインオンを実行するには、Kerberos の制約付き委任を設定する必要があります。

#### <a name="task-3-configure-windows-admin-center-extensions"></a>タスク 3: Windows Admin Center 拡張機能を構成する

1. **SEA-ADM1** の右上隅にある **[設定]** アイコン (歯車) を選択します。

1. **Extentions** をクリックして、使用可能な拡張機能を確認します。

1. **Security(Preview)** 拡張機能を選択し、上にある **Install** をクリックしてインストールします。 拡張機能がインストールされ Windows Admin Center が更新されます。

   > **注**: **Security(Preview)** 拡張機能を使用できない場合は、別の Microsoft 拡張機能を選択します。

1. **Installed extentions** タブを開いて、一覧に DNS (プレビュー) 拡張機能が含まれていることを確認します。

1. 上部左側にあるメニューの **[Settings/設定]** の横にあるドロップダウン矢印を選択してから、**[サーバー マネージャー]** を選択します。

1. Windows Admin Center 内で、`sea-dc1.contoso.com` をクリックして接続し、必要に応じて、ユーザー名として **CONTOSO\\Administrator** を、パスワードとして **Pa55w.rd** を使用してサインインします。
1. `sea-dc1.contoso.com` 上の左側のアイコン一覧から**DNS** サーバーアイコンをクリックし、**Install**をクリックして DNS PowerShell ツールをインストールします。
1. **Contoso.com** ゾーンを選択し、その DNS レコードの一覧を確認します。

#### <a name="task-4-verify-remote-administration"></a>タスク 4: リモート管理を確認する

1. **SEA-ADM1** の Windows Admin Center で、`sea-dc1.contoso.com` に接続されていることを確認して [Overview/概要] ペインを確認します。 Windows Admin Center の詳細ウィンドウには、基本的なサーバー情報とパフォーマンスの監視が表示されます。

1. Windows Admin Center で、左側のメニューから **Roles ＆ Features** ツールを選択jして、**Telnet Client** を `sea-dc1.contoso.com` にインストールします。 

1. Windows Admin Center で、左側のメニュー一覧の一番下にある **Settings** を選択し、**Remote Desktop** をクリックします。

1. **Allow Remote connections to this computer** を選択し、**Save** します。

1. リモート デスクトップクライアント（mstsc)を起動して、リモートデスクトップ経由で `sea-dc1.contoso.com` に接続できることを確認します。

1. リモート デスクトップ セッションを切断します。 

1. Microsoft Edge ウィンドウを閉じます。

#### <a name="task-5-administer-servers-with-remote-powershell"></a>タスク 5: リモート PowerShell を使用してサーバーを管理する

1. **SEA-ADM1** 上で、**Windows PowerShell** コンソールに切り替えます。

1. **Windows PowerShell** コンソールで、次のコマンドを実行して、**SEA-DC1** への PowerShell リモート処理セッションを開始します。

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```
1. **[SEA-DC1]** プロンプトから、次のコマンドを実行して、アプリケーション ID サービス (AppIDSvc) の状態を表示します。

   ```powershell
   Get-Service -Name AppIDSvc
   ```

   > **注**: サービスが現在停止中(Stopped)であることを確認します。

1. **[SEA-DC1]** プロンプトから、次のコマンドを実行して Application Identity サービスを開始します。

   ```powershell
   Start-Service -Name AppIDSvc
   ```
1. **[SEA-DC1]** プロンプトから、次のコマンドを実行して、アプリケーション ID サービス (AppIDSvc) の状態を表示します。

   ```powershell
   Get-Service -Name AppIDSvc
   ```

   > **注**: サービスが現在実行中(Running) であることを確認します。

### <a name="results"></a>結果

この演習を完了すると、Windows Admin Center がインストールされ、ラボ環境内のサーバーに接続されます。 機能のインストールや、リモート デスクトップ接続の有効化およびテストなどのリモート管理タスクを複数実行しました。 最後に、PowerShell リモート処理を使用してサービスの状態を確認して、開始しました。
