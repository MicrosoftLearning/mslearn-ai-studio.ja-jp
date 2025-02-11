---
lab:
  title: Azure AI Foundry SDK を使用してチャット アプリを作成する
---

# Azure AI Foundry SDK を使用してチャット アプリを作成する

この演習では、Azure AI Foundry SDK を使用してシンプルなチャット アプリを作成します。

この演習は約 **20** 分かかります。

## Azure AI Foundry プロジェクトを作成する

まず、Azure AI Foundry プロジェクトを作成します。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ホーム ページで、**[+ 作成]** を選択します。
1. **プロジェクトの作成** ウィザードで、適切なプロジェクト名 (たとえば、`my-ai-project`) を入力してから、プロジェクトをサポートするために自動的に作成される Azure リソースを確認します。
1. **[カスタマイズ]** を選択し、ハブに次の設定を指定します。
    - **[ハブ名]**: *一意の名前 - たとえば `my-ai-hub`*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]**: *一意の名前 (たとえば、`my-ai-resources`) で新しいリソース グループを作成するか、既存のものを選びます*
    - **[場所]**: 次の一覧からランダム リージョンを選択します\*
        - 米国東部
        - 米国東部 2
        - 米国中北部
        - 米国中南部
        - スウェーデン中部
        - 米国西部
        - 米国西部 3
    - **Azure AI サービスまたは Azure OpenAI の接続**: *適切な名前 (たとえば、`my-ai-services`) を使用して新しい AI サービス リソースを作成するか、既存のものを使用します*
    - **Azure AI 検索への接続**:接続をスキップする

    > \* モデルのクォータは、リージョンのクォータによってテナント レベルで制限されます。 ランダム リージョンを選択すると、複数のユーザーが同じテナントで作業しているときに、クォータの可用性を均等配置できます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[次へ]** を選択し、構成を確認します。 **[作成]** を選択し、プロセスが完了するまで待ちます。
1. プロジェクトが作成されたら、表示されているヒントをすべて閉じて、Azure AI Foundry ポータルのプロジェクト ページを確認します。これは次の画像のようになるはずです。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./media/ai-foundry-project.png)

## 生成 AI モデルを展開する

これで、チャット アプリケーションをサポートする生成 AI 言語モデルをデプロイする準備ができました。 この例では、Microsoft Phi-4 モデルを使用します。しかし、原則はどのモデルでも同じです。

1. プロジェクトの左側のウィンドウの **[マイ アセット]** セクションで、**[モデル + エンドポイント]** ページを選択します。
1. **[モデル + エンドポイント]** ページの **[モデル デプロイ]** タブの **[+ モデルのデプロイ]** メニューで、**[基本モデルのデプロイ]** を選択します。
1. 一覧で **Phi-4** モデルを検索してから、それを選択して確認します。
1. **[Azure AI Content Safety を使用するサーバーレス API]** デプロイ オプションを選択します。
1. デプロイの詳細で **[カスタマイズ]** を選択して、以下の設定でモデルをデプロイします。
    - **[デプロイ名]**: *モデル デプロイの一意の名前 - たとえば、`phi-4-model` (割り当てた名前を覚えておいてください。後で必要になります*)
    - **コンテンツ フィルター**: 有効

## モデルとチャットするクライアント アプリケーションを作成する

モデルをデプロイしたら、Azure AI Foundry SDK を使用して、モデルとチャットするアプリケーションを開発できます。

### アプリケーション構成を準備する

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。
1. **プロジェクトの詳細**エリアで、**プロジェクト接続文字列**を書き留めます。 この接続文字列を使用して、クライアント アプリケーションでプロジェクトに接続します。
1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 それから新しいブラウザー タブの `https://portal.azure.com` で [Azure portal](https://portal.azure.com) を開き、プロンプトが表示されたら Azure 資格情報を使用してサインインします。
1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成します。***PowerShell*** 環境を選択します。 Azure portal の下部にあるペインに、Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

1. PowerShell ペインで、次のコマンドを入力して、この演習用の GitHub リポジトリを複製します。

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

1. リポジトリが複製されたら、**mslearn-ai-foundry/labfiles/01a-azure-foundry-sdk/python** フォルダーに移動します。

    ```
    cd mslearn-ai-foundry/labfiles/01a-azure-foundry-sdk/python
    ```

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、これから使用する Python ライブラリをインストールします。
    - **python-dotenv** : アプリケーション構成ファイルから設定を読み込む際に使用されます。
    - **azure-identity**: Entra ID 資格情報を使用して認証するために使用されます。
    - **azure-ai-projects**: Azure AI Foundry プロジェクトの操作に使用されます。
    - **azure-ai-inference**: 生成 AI モデルとチャットするために使用されます。

    ```
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

1. 次のコマンドを入力して、提供されている **.env** Python 構成ファイルを編集します。

    ```
    code .env
    ```

    コード エディターでファイルを開きます。

1. コード ファイルで、**your_project_endpoint** プレースホルダーをプロジェクトの接続文字列 (Azure Ai Foundry ポータルのプロジェクトの**概要**ページからコピー) に置き換え、**your_model_deployment** プレースホルダーを Phi-4 モデル デプロイに割り当てた名前に置き換えます。
1. プレースホルダーを置き換えたら、**CTRL + S** コマンドを使用して変更を保存してから、**CTRL + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### プロジェクトに接続してモデルとチャットするためのコードを記述する

> **ヒント**: Python コード ファイルにコードを追加するときは、必ず正しいインデントを維持してください。

1. 次のコマンドを入力して、提供されている **chat-app.py** Python コード ファイルを編集します。

    ```
    code chat-app.py
    ```

1. コード ファイルで、ファイルの先頭に追加された既存の**インポート** ステートメントを書き留めます。 次に、コメント **# AI プロジェクトのリファレンスの追加**で、次のコードを追加して、Azure AI プロジェクト ライブラリを参照します。

    ```python
    from azure.ai.projects import AIProjectClient
    ```

1. **main** 関数のコメント **# 構成設定の取得**で、コードが **.env** ファイルで定義したプロジェクト 接続文字列とモデル デプロイ名の値を読み込むことに注意してください。
1. コメント **# プロジェクト クライアントの初期化**で、次のコードを追加して、現在サインインしている Azure 資格情報を使用して Azure AI Foundry プロジェクトに接続します。

    ```python
    project = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential()
        )
    ```
    
1. コメント **# チャット クライアントの取得**で、次のコードを追加して、モデルとチャットするためのクライアント オブジェクトを作成します。

    ```python
    chat = project.inference.get_chat_completions_client()
    ```

1. コードには、ユーザーが "quit" と入力するまでプロンプトを入力できるようにするループが含まれていることに注意してください。 次に、ループ セクションのコメント **# チャットの完了の取得**で、次のコードを追加して、プロンプトを送信し、モデルから完了を取得します。

    ```python
    response = chat.complete(
        model=model_deployment,
        messages=[
            {"role": "system", "content": "You are a helpful AI assistant that answers questions."},
            {"role": "user", "content": input_text},
            ],
        )
    print(response.choices[0].message.content)
    ```

1. **CTRL + S** コマンドを使用してコード ファイルに変更を保存してから、**CTRL + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### チャット アプリケーションを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Python コードを実行します。

    ```
    python chat-app.py
    ```

1. メッセージが表示されたら、`What is the fastest animal on Earth?` などの質問を入力し、生成 AI モデルからの応答を確認します。
1. さらにいくつかの質問をお試しください。 終了したら、`quit` を入力してプログラムを終了します。

## まとめ

この演習では、Azure AI Foundry SDK を使用して、Azure AI Foundry プロジェクトにデプロイした生成 AI モデル用のクライアント アプリケーションを作成しました。

## クリーンアップ

Azure AI Foundry ポータルを確認し終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブの `https://portal.azure.com` で [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
