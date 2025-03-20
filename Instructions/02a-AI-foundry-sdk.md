---
lab:
  title: 生成 AI チャット アプリを作成する
  description: Azure AI Foundry SDK を使用して、プロジェクトに接続して言語モデルとチャットするアプリを構築する方法について説明します。
---

# 生成 AI チャット アプリを作成する

この演習では、Azure AI Foundry SDK を使用して、プロジェクトに接続して言語モデルとチャットするシンプルなチャット アプリを作成します。

この演習は約 **30** 分かかります。

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
    - **場所**: **[選択に関するヘルプ]** を選択し、次に [場所ヘルパー] ウィンドウで **gpt-4** を選択し、推奨されるリージョンを選択します\*
    - **Azure AI サービスまたは Azure OpenAI の接続**: *適切な名前 (たとえば、`my-ai-services`) を使用して新しい AI サービス リソースを作成するか、既存のものを使用します*
    - **Azure AI 検索への接続**:接続をスキップする

    > \* Azure OpenAI リソースは、リージョンのクォータによってテナント レベルで制限されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[次へ]** を選択し、構成を確認します。 **[作成]** を選択し、プロセスが完了するまで待ちます。
1. プロジェクトが作成されたら、表示されているヒントをすべて閉じて、Azure AI Foundry ポータルのプロジェクト ページを確認します。これは次の画像のようになっているはずです。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./media/ai-foundry-project.png)

## 生成 AI モデルを展開する

これで、チャット アプリケーションをサポートする生成 AI 言語モデルをデプロイする準備ができました。 この例では、Microsoft Phi-4 モデルを使用します。しかし、原則はどのモデルでも同じです。

1. Azure AI Foundry プロジェクト ページの右上にあるツール バーで、**プレビュー機能**アイコンを使用して、**[Azure AI モデル推論サービスにモデルをデプロイする]** 機能を有効にします。 この機能により、アプリケーション コードで使用する Azure AI 推論サービスでモデル デプロイを使用できるようになります。
1. プロジェクトの左側のウィンドウの **[マイ アセット]** セクションで、**[モデル + エンドポイント]** ページを選択します。
1. **[モデル + エンドポイント]** ページの **[モデル デプロイ]** タブの **[+ モデルのデプロイ]** メニューで、**[基本モデルのデプロイ]** を選択します。
1. 一覧で **Phi-4** モデルを検索してから、それを選択して確認します。
1. メッセージに応じて使用許諾契約書に同意したあと、デプロイの詳細で **[カスタマイズ]** を選択して、以下の設定でモデルをデプロイします。
    - **デプロイ名**: *モデル デプロイの一意の名前 - たとえば、`phi-4-model` (割り当てた名前を覚えておいてください。後で必要になります*)
    - **デプロイの種類**: グローバル標準
    - **デプロイの詳細**: *既定の設定を使用します*
1. デプロイのプロビジョニングの状態が**完了**になるまで待ちます。

## モデルとチャットするクライアント アプリケーションを作成する

これでモデルをデプロイしたので、Azure AI Foundry SDK を使用して、モデルとチャットするアプリケーションを開発できます。

> **ヒント**: Python または Microsoft C# を使用してソリューションを開発することを選択できます。 選択した言語の適切なセクションの指示に従います。

### アプリケーション構成を準備する

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。
1. **[プロジェクトの詳細]** エリアで、**[プロジェクト接続文字列]** の内容を書き留めます。 この接続文字列を使用して、クライアント アプリケーションでプロジェクトに接続します。
1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。
1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成します。***PowerShell*** 環境を選択します。 Azure portal の下部にあるペインに、Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. PowerShell ペインで、次のコマンドを入力して、この演習用の GitHub リポジトリを複製します。

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

> **注**: 選択したプログラミング言語の手順に従います。

1. リポジトリが複製されたら、チャット アプリケーションのコード ファイルを含んだフォルダーに移動します。  

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/c-sharp
    ```

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、これから使用するライブラリをインストールします。

    **Python**

    ```
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
   dotnet add package Azure.Identity
    ```
    

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_endpoint** プレースホルダーをプロジェクトの接続文字列 (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを Phi-4 モデル デプロイに割り当てた名前に置き換えます。
1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### プロジェクトに接続してモデルとチャットするためのコードを記述する

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. コード ファイルで、ファイルの先頭に追加された既存のステートメントを書き留めて、必要な SDK 名前空間をインポートします。 次に、コメント**参照の追加**の下に、次のコードを追加して、前にインストールしたライブラリの名前空間を参照します。

    **Python**

    ```
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
    ```

    **C#**

    ```
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

1. **main** 関数のコメント**構成設定の取得**で、構成ファイルで定義したプロジェクト接続文字列とモデル デプロイ名の値がコードで読み込まれることに注意してください。
1. コメント**プロジェクト クライアントの初期化**で、次のコードを追加して、現在のサインインに使用した Azure 資格情報で Azure AI Foundry プロジェクトに接続します。

    **Python**

    ```
   projectClient = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. コメント**チャット クライアントの取得**で、次のコードを追加して、モデルとチャットするためのクライアント オブジェクトを作成します。

    **Python**

    ```
   chat = projectClient.inference.get_chat_completions_client()
    ```

    **C#**

    ```
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

1. コードには、ユーザーが「quit」と入力するまでプロンプトを入力できるようにするループが含まれていることに注意してください。 次に、ループ セクションのコメント**チャット完了の取得**で、次のコードを追加して、プロンプトを送信し、モデルから完了を取得します。

    **Python**

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

    **C#**

    ```
   var requestOptions = new ChatCompletionsOptions()
   {
       Model = model_deployment,
       Messages =
           {
               new ChatRequestSystemMessage("You are a helpful AI assistant that answers questions."),
               new ChatRequestUserMessage(input_text),
           }
   };
    
   Response<ChatCompletions> response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

1. **Ctrl + S** キー コマンドを使用してコード ファイルに変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### チャット アプリケーションを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. メッセージが表示されたら、`What is the fastest animal on Earth?` などの質問を入力し、生成 AI モデルからの応答を確認します。
1. さらにいくつかの質問をお試しください。 終了したら、`quit` を入力してプログラムを終了します。

## まとめ

この演習では、Azure AI Foundry SDK を使用して、Azure AI Foundry プロジェクトにデプロイした生成 AI モデル用のクライアント アプリケーションを作成しました。

## クリーンアップ

Azure AI Foundry ポータルを確認し終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
