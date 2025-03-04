---
lab:
  title: 'ラボ: ハイブリッド シナリオでの Windows Admin Center の使用'
  module: 'Module 4: Facilitating hybrid management'
---

# <a name="lab-using-windows-admin-center-in-hybrid-scenarios"></a>ラボ: ハイブリッド シナリオでの Windows Admin Center の使用

## <a name="scenario"></a>シナリオ

マネージド システムの場所に関係なく、一貫した運用と管理のモデルに関する懸念事項に対処するには、オンプレミスと Microsoft Azure 仮想マシン (VM) で実行されているさまざまなバージョンの Windows Server オペレーティング システムを含むハイブリッド環境で Windows Admin Center の機能をテストします。

目標は、マネージド システムの場所に関係なく、Windows Admin Center を一貫して使用できることの確認です。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure ネットワーク アダプターを使用してハイブリッド接続をテストします。
- Azure で Windows Admin Center ゲートウェイをデプロイします。
- Azure で Windows Admin Center ゲートウェイの機能を検証します。

## <a name="estimated-time-90-minutes"></a>予想所要時間: 90 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-800T00A-SEA-DC1** および **AZ-800T00A-ADM1** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-800T00A-SEA-DC1** VM と **AZ-800T00A-SEA-ADM1** VM で **SEA-DC1** と **SEA-ADM1** のインストールをホストしています

1. **SEA-ADM1** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、使用可能な VM 環境と Azure サブスクリプションを使用します。 ラボを開始する前に、Azure サブスクリプションがあり、そのサブスクリプションの所有者ロールまたは共同作成者ロールを持ち、そのサブスクリプションに関連付けられている Azure Active Directory (Azure AD) テナントのグローバル管理者ロールを持っているユーザー アカウントがあることを確認します。

## <a name="exercise-1-provisioning-azure-vms-running-windows-server"></a>演習 1: Windows Server を実行する Azure VM のプロビジョニング

### <a name="scenario"></a>シナリオ

オンプレミス サーバーと Azure 仮想ネットワークの間にハイブリッド接続を確立できることを確認する必要があります。 そのためにはまず、Azure Resource Manager テンプレートを使用して、Windows Server を実行する Azure VM をプロビジョニングします。

この演習の主なタスクは次のとおりです。

1. Azure Resource Manager テンプレートを使用して Azure リソース グループを作成する。
1. Azure Resource Manager テンプレートを使用して Azure VM を作成する。

#### <a name="task-1-create-an-azure-resource-group-by-using-an-azure-resource-manager-template"></a>タスク 1: Azure Resource Manager テンプレートを使用して Azure リソース グループを作成する

1. **SEA-ADM1** で Microsoft Edge を起動し、Azure portal を参照して、**Microsoft Account**(このコースの冒頭で新規に作成した outlook.jp アカウント)で認証します。
1. Azure portal の ［Cloud Shell］ を起動します（ポータル右上にあるコマンドプロンプトマーク）
1. PowerShell をクリックします
1. **ストレージの作成** をクリックします（３分程度かかります）
1. Cloud Shell で以下のコマンドを実行して、必要なファイルをGitHubからクローンします
  ```
  git clone https://github.com/MicrosoftLearning/AZ-800-Administering-Windows-Server-Hybrid-Core-Infrastructure.ja-jp.git
  ```

1. ls コマンドを使用して AZ-800-Administratering-Windows-Server-Hybrid-Core-Infrastructure.ja-jp が作成されていることを確認してください。
1. **AZ-800-Administratering-Windows-Server-Hybrid-Core-Infrastructure.ja-jp\Allfiles\Labfiles\Lab04** に移動します。
1 [Cloud Shell］ ペインで、次のコマンドを実行して、このラボでプロビジョニングするリソースが入ることになるリソース グループを作成します。

   >**注**: このラボは、米国東部 East US を使用しています。 通常、Azure VM をプロビジョニングできる Azure リージョンを特定するには、「[ご利用のリージョンの Azure クレジット プランを確認する](https://aka.ms/regions-offers)」を参照してください。

   ```powershell
   $location = 'East US'
   $rgName = 'AZ800-L0401-RG'
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az800l04subDeployment `
     -TemplateFile ./L04-sub_template.json `
     -rgLocation $location `
     -rgName $rgName
   ```

#### <a name="task-2-create-an-azure-vm-by-using-an-azure-resource-manager-template"></a>タスク 2: Azure Resource Manager テンプレートを使用して Azure VM を作成する

1. [Cloud Shell] ペインで、次のコマンドを実行して、このラボで使用する、Windows Server を実行している Azure VM をデプロイします。

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az800l04rgDeployment `
     -ResourceGroupName $rgName `
     -TemplateFile ./L04-rg_template.json `
     -TemplateParameterFile ./L04-rg_template.parameters.json
   ```

   >**注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 デプロイには約 5 分かかります。

1. Azure portal の [Cloud Shell] ペインを閉じます。
1. Azure Portal で **VNET/仮想ネットワーク** の管理画面移動します。
1. Azure portal で、**az800l04-vnet** 仮想ネットワークを開きます。
1. **サブネット** をクリックし、**「＋ゲートウェイサブネット**」をクリックします。
1. IP アドレスの範囲を **10.4.3.224/27** に設定し、**GatewaySubnet** を作成します。

## <a name="exercise-2-implementing-hybrid-connectivity-by-using-the-azure-network-adapter"></a>演習 2: Azure ネットワーク アダプターを使用したハイブリッド接続の実装

### <a name="scenario"></a>シナリオ

オンプレミス サーバーと前の演習でプロビジョニングした Azure VM の間にハイブリッド接続を確立できることを確認する必要があります。 この目的のために、Windows Admin Center の Azure ネットワーク アダプター機能を使用します。

この演習の主なタスクは次のとおりです。

1. Windows Admin Center を Azure に登録する。
1. Azure ネットワーク アダプターを作成する。

#### <a name="task-1-register-windows-admin-center-with-azure"></a>タスク 1: Windows Admin Center を Azure に登録する

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を開始します。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。

1. **Windows PowerShell** コンソールで、次のコマンドを実行してから Enter キーを押し、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

1. **SEA-ADM1** を再起動します（10分程度かかる場合があります）
1. **SEA-ADM1** で Microsoft Edge を起動し、`https://SEA-ADM1.contoso.com` で Windows Admin Center のローカル インスタンスに接続します。 

   >**注**: リンクが機能しない場合は、**SEA-ADM1** で **WindowsAdminCenter.msi** ファイルを参照し、コンテキスト メニューを開いて **[修復]** を選択します。 修復が完了した後、Microsoft Edge を更新します。 

1. メッセージが表示されたら、**[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力し、**[OK]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. Windows Admin Center の右上にある「Setting（ギア アイコン）」をクリックします。
1. Account ページが表示されたら **「Register with Azure」** をクリックします
1. **Register** をクリックします
1. 画面右側に表示された**Getting started with Azure in Windows Admin Center** で **Copy** をクリックしてコードをコピーします
1. **Enter the Code** をクリックします。
1. コードをペーストして「Next」をクリックします
1. マイクロソフトアカウントでサインインします
1. **Continue** をクリックします
1. タブを閉じてAdmin Center に戻ります。
1. **Connect** をクリックします
1. **Sign in** をクリックします。

>いくつかのエラーが表示されますが、ここでは無視してください。この後のステップで解決します

1. **View in Azure** リンクをクリックします
1. **Default Directory に管理者の同意を与えます** をクリックし、「はい」を選択します

#### <a name="task-2-create-an-azure-network-adapter"></a>タスク 2: Azure ネットワーク アダプターを作成する

1. 左上の **Windows Admin Center** をクリックします。
2. **SEA-ADM1.contoso.com** に接続します。
3. Tools メニューから**Networks**を選択します
4. **「＋Add Azure Network Adapter**」をクリックします
10. 次の設定で Azure ネットワーク ゲートウェイを作成します。

   |設定|値|
   |---|---|
   |Subscription|このラボで使用している Azure サブスクリプションの名前|
   |Location|East US|
   |Virtual Network|az800l04-vnet|
   |Gateway Subnet|10.4.3.224/27|
   |Gateway SKU|VpnGw1|
   |Client Address Space|192.168.0.0/24|
   |Authentication Certificate|Auto-Generated Self-signed root and client Certificate/自動生成された自己署名ルート証明書とクライアント証明書|

   >**注**: Azure 仮想ネットワーク ゲートウェイのプロビジョニングには最大 45 分かかります。 プロビジョニングが完了するのを待たずに、次の**演習3**に進んでください。

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替え、**WAC-Created-vpngw-** で始まる名前の新しい仮想ネットワーク ゲートウェイがプロビジョニングされていることを確認します。

## <a name="exercise-3-deploying-windows-admin-center-gateway-in-azure"></a>演習 3: Azure での Windows Admin Center ゲートウェイのデプロイ

### <a name="scenario"></a>シナリオ

Windows Admin Center を使用して、Windows Server OS を実行している Azure VM を管理する機能を評価する必要があります。 これを行うには、まず、このラボの最初の演習で実装した Azure 仮想ネットワークに Windows Admin Center ゲートウェイをインストールします。

この演習の主なタスクは次のとおりです。

1. Azure で Windows Admin Center ゲートウェイをインストールする。
1. スクリプト プロビジョニングの結果を確認する。

#### <a name="task-1-install-windows-admin-center-gateway-in-azure"></a>タスク 1: Azure で Windows Admin Center ゲートウェイをインストールする

1. **SEA-ADM1** で、Azure portal が表示されているブラウザー ウィンドウに切り替えます。
1. Azure portal の ［Cloud Shell］ ペインで PowerShell セッションを開始します。
1. **AZ-800-Administratering-Windows-Server-Hybrid-Core-Infrastructure.ja-jp\Allfiles\Labfiles\Lab04** に移動します。
1. [Cloud Shell] ペインで、次のコマンドを実行して、Windows Admin Center プロビジョニング スクリプトが使用する **AzureRm** PowerShell コマンドレットの互換性を有効にします。

   ```powershell
   Enable-AzureRmAlias -Scope Process
   ```
1. 次のコマンドを実行して、Windows Admin Center プロビジョニング スクリプトを実行するために必要な変数の値を設定します。

   ```powershell
   $rgName = 'AZ800-L0401-RG'
   $vnetName = 'az800l04-vnet'
   $nsgName = 'az800l04-web-nsg'
   $subnetName = 'subnet1'
   $location = 'East US'
   $pipName = 'wac-public-ip'
   $size = 'Standard_D2s_v3'
   ```
1. 次のコマンドを実行して、スクリプト パラメーター変数を設定します。

   ```powershell
   $scriptParams = @{
     ResourceGroupName = $rgName
     Name = 'az800l04-vmwac'
     VirtualNetworkName = $vnetName
     SubnetName = $subnetName
     GenerateSslCert = $true
     size = $size
     PublicIPAddressName = $pipname
   }
   ```
1. 次のコマンドを実行して、PowerShell リモート処理の証明書検証を無効にします (最初のコマンドの後にプロンプトが表示されたら、**A** を入力して Enter キーを押します)。

   ```powershell
   install-module pswsman
   Disable-WSManCertVerification -All
   ```
1. 次のコマンドを実行して、プロビジョニング スクリプトを起動します。

   ```powershell
   ./Deploy-WACAzVM.ps1 @scriptParams
   ```
1. ローカル管理者アカウントの名前を入力するように求められたら、「**Student**」と入力します。
1. ローカル管理者アカウントのパスワードを入力するように求められたら、「**Pa55w.rd1234**」と入力します。

    >**注**: プロビジョニング スクリプトが完了するまで待ちます。 これには 5 分ほどかかる場合があります。

1. スクリプトが正常に完了したことを確認し、Windows Admin Center インストールをホストする **Azure VM の完全修飾名を含む URL** を示す最後のメッセージに注意してください。

   >**注**: Azure VM の完全修飾名を記録します。 このラボで後ほど必要になります。

1. [Cloud Shell] ペインを閉じます。

#### <a name="task-2-review-results-of-the-script-provisioning"></a>タスク 2: スクリプト プロビジョニングの結果を確認する

1. Azure portal で、**AZ800-L0401-RG** リソース グループのページを参照します。
1. **AZ800-L0401-RG** ページの **[概要]** ページで、リソース一覧にある 仮想マシン **az800l04-vmwac** をクリックします
1. **az800l04-vmwac | ネットワーク** ページの **[受信ポートの規則]** タブで、TCP ポート 5986 で接続を許可する受信ポートの規則、および TCP ポート 443 で接続を許可する受信規則をそれぞれ示すエントリを確認します。

## <a name="exercise-4-verifying-functionality-of-the-windows-admin-center-gateway-in-azure"></a>演習 4: Azure での Windows Admin Center ゲートウェイの機能の確認

### <a name="scenario"></a>シナリオ

必須コンポーネントがすべて揃っている状態で、このラボの最初の演習でプロビジョニングした Azure 仮想ネットワークにデプロイした Azure VM をターゲットとして、WAC 機能をテストします。

この演習の主なタスクは次のとおりです。

1. Azure VM で実行されている Windows Admin Center ゲートウェイに接続する。
1. Azure VM の PowerShell リモート処理を有効にする。
1. Azure VM で実行されている Windows Admin Center ゲートウェイを使用して、Azure VM に接続する。

#### <a name="task-1-connect-to-the-windows-admin-center-gateway-running-in-azure-vm"></a>タスク 1: Azure VM で実行されている Windows Admin Center ゲートウェイに接続する

1. **SEA-ADM1** で Microsoft Edge を起動し、前の演習で特定したURL（https://az800l104-vmac-xxxxxx.eastus.cloudapp.azure.com) に接続し、Azure VM で実行されている Windows Admin Center ゲートウェイに接続します。
1. メッセージが表示されたら、ユーザー名 **Student** とパスワード **Pa55w.rd1234** でサインインします。
1. Windows Admin Center の [All Connections] ペインで、**az800l04-vmwac[Gateway]** を選択します。
1. Windows Admin Center の [Overview] ペインを確認します。

#### <a name="task-2-enable-powershell-remoting-on-an-azure-vm"></a>タスク 2: Azure VM の PowerShell リモート処理を有効にする

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウで、**az800l04-vm0** Azure VM のページを表示します。

1. **az800l04-vm0** ページで、 **[実行コマンド]** で **RunPowerShell** を選択します。
1. 以下のコマンドを実行して WinRM を有効にします。

   ```powershell
   winrm quickconfig -quiet
   ```

1. 同様に、以下のコマンドを実行して、Windows リモート管理受信ポートを開きます。

   ```powershell
   Set-NetFirewallRule -Name WINRM-HTTP-In-TCP-PUBLIC -RemoteAddress Any
   ```

1. 同様に以下のコマンドを実行して、PowerShell リモート処理を有効にします。

   ```powershell
   Enable-PSRemoting -Force -SkipNetworkProfileCheck
   ```

#### <a name="task-3-connect-to-an-azure-vm-by-using-the-windows-admin-center-gateway-running-in-azure-vm"></a>タスク 3: Azure VM で実行されている Windows Admin Center ゲートウェイを使用して、Azure VM に接続する

1. **az800l04-vmwac** で実行されている Windows Admin Center ゲートウェイで、**All Connections** ページを開き、**「＋Add」** をクリックします
1. **Sign in** が表示されている場合は、マイクロソフトアカウントでサインインします。
1. リソースグループとして AZ800-L0401-RG を選択します。
1. AZ800l04-vm0 を選択します。
1. IP Address として、プライベートIPアドレスを選択します。 
1. **Add** をクリックします。
1. ユーザー名 **Student**、パスワード **Pa55w.rd1234** を使用して認証します。
1. [Overview] ペインを確認します

## <a name="exercise-5-deprovisioning-the-azure-environment"></a>演習 5: Azure 環境のプロビジョニング解除

### <a name="scenario"></a>シナリオ

Azure 関連の料金を最小限に抑えるため、このラボでプロビジョニングした Azure リソースをプロビジョニング解除します。

この演習の主なタスクは次のとおりです。

1. Cloud Shell で PowerShell セッションを開始する。
1. ラボでプロビジョニングしたすべての Azure リソースを特定する。

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>タスク 1: Cloud Shell で PowerShell セッションを開始する

1. **SEA-ADM1** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。
1. Azure portal の ［Cloud Shell］ ペインで PowerShell セッションを開きます。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>タスク 2: ラボでプロビジョニングしたすべての Azure リソースを特定する

1. [Cloud Shell] ペインで次のコマンドを実行して、このラボで作成されたすべてのリソース グループのリストを表示します。

   ```powershell
   Get-AzResourceGroup -Name 'az800l04*'
   ```

1. 次のコマンドを実行して、このラボで作成したすべてのリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'az800l04*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**注**: このコマンドは非同期で実行されます (-AsJob パラメーターによって決定されます)。 そのため、同じ PowerShell セッション内ですぐに別の PowerShell コマンドを実行できるようになりますが、リソース グループが実際に削除されるまでに数分かかります。

### <a name="results"></a>結果

このラボを完了すると、Contoso の管理容易性とセキュリティの要件を満たす、Windows Server を実行している Azure VM がデプロイおよび構成されました。

### <a name="prepare-for-the-next-module"></a>次のモジュールの準備

次のモジュールの準備が完了したら、ラボを終了します。
