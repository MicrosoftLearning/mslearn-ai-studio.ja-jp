---
lab:
  title: 生成 AI チャット アプリを作成する
  description: Azure AI Foundry SDK を使用して、プロジェクトに接続して言語モデルとチャットするアプリを構築する方法について説明します。
---

# 生成 AI チャット アプリを作成する

この演習では、Azure AI Foundry SDK を使用して、プロジェクトに接続して言語モデルとチャットするシンプルなチャット アプリを作成します。

この演習は約 **40** 分かかります。

> **注**: この演習は、変更される可能性があるプレリリース SDK に基づいています。 必要に応じて、特定のバージョンのパッケージを使用しました。利用可能な最新バージョンが反映されていない可能性があります。 予期しない動作、警告、またはエラーが発生する場合があります。

## Azure AI Foundry プロジェクトにモデルをデプロイする

まず、Azure AI Foundry プロジェクトにモデルをデプロイします。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ホーム ページの **[モデルと機能を調査する]** セクションで、プロジェクトで使用する `gpt-4o` モデルを検索します。
1. 検索結果で **gpt-4o** モデルを選んで詳細を確認してから、モデルのページの上部にある **[このモデルを使用する]** を選択します。
1. プロジェクトの作成を求められたら、プロジェクトの有効な名前を入力し、**[詳細]** オプションを展開します。
1. **[カスタマイズ]** を選択し、ハブに次の設定を指定します。
    - **Azure AI Foundry リソース**: *Azure AI Foundry リソースの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **リージョン**: ***AI サービスでサポートされている場所を選択します***\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[作成]** を選択し、選んだ gpt-4 モデル デプロイを含むプロジェクトが作成されるまで待ちます。
1. プロジェクトが作成されると、チャット プレイグラウンドが自動的に開きます。
1. **[セットアップ]** ウィンドウで、モデル デプロイの名前をメモします (**gpt-4o** のはずです)。 これを確認するには、**[モデルとエンドポイント]** ページでデプロイを表示します (左側のナビゲーション ウィンドウでそのページを開くだけです)。
1. 左側のナビゲーション ウィンドウで **[概要]** を選択すると、プロジェクトのメイン ページが表示されます。次のようになります。

    > **注**: *アクセス許可が不十分です** というエラーが表示された場合は、**[修正]** ボタンを使用してエラーを解決します。

    ![Azure AI Foundry プロジェクトの概要ページのスクリーンショット。](./media/ai-foundry-project.png)

## モデルとチャットするクライアント アプリケーションを作成する

これでモデルをデプロイしたので、Azure AI Foundry SDKと Azure AI モデル推論 SDK を使用して、モデルとチャットするアプリケーションを開発できます。

> **ヒント**: Python または Microsoft C# を使用してソリューションを開発することを選択できます。 選択した言語の適切なセクションの指示に従います。

### アプリケーション構成を準備する

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。
1. **[プロジェクトの詳細]** エリアの **[Azure AI Foundry プロジェクト エンドポイント]** をメモします。 クライアント アプリケーションで、このエンドポイントを使用してプロジェクトに接続します。
1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。

    ウェルカム通知を閉じて、Azure portal のホーム ページを表示します。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、サブスクリプションにストレージがない ***PowerShell*** 環境を選択します。

    Azure portal の下部にあるペインに Cloud Shell のコマンド ライン インターフェイスが表示されます。 作業しやすくするために、このウィンドウのサイズを変更したり最大化したりすることができます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. Cloud Shell 画面で、次のコマンドを入力して、この演習のコード ファイルを含む GitHub リポジトリを複製します (コマンドを入力するか、クリップボードにコピーしてから、コマンド ラインで右クリックし、プレーンテキストとして貼り付けます)。

    ```
   rm -r mslearn-ai-foundry -f
   git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **ヒント**: Cloudshell にコマンドを入力すると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. リポジトリが複製されたら、チャット アプリケーションのコード ファイルを含んだフォルダーに移動します。

    選択したプログラミング言語に応じて、次のコマンドを使用します。

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/c-sharp
    ```

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.9
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.5
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

1. コード ファイルで、**your_project_endpoint** プレースホルダーをプロジェクトのエンドポイント (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーをお使いの gpt-4 モデル デプロイの名前に置き換えます。
1. プレースホルダーを置き換えたら、コード エディター内で、**Ctrl + S** コマンドを使用するか、**右クリックして保存**で変更を保存してから、**Ctrl + Q** コマンドを使用するか、**右クリックして終了**で、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

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

1. コード ファイルで、ファイルの先頭に追加された既存のステートメントを書き留めて、必要な SDK 名前空間をインポートします。 次に、コメント**参照の追加**を探して、次のコードを追加し、前にインストールしたライブラリの名前空間を参照します。

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import SystemMessage, UserMessage, AssistantMessage
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

1. **main** 関数のコメント**構成設定の取得**で、構成ファイルで定義したプロジェクト接続文字列とモデル デプロイ名の値がコードで読み込まれることに注意してください。
1. コメント**プロジェクト クライアントの初期化**を探して、次のコードを追加し、現在のサインインに使用した Azure 資格情報で Azure AI Foundry プロジェクトに接続します。

    > **ヒント**: コードのインデント レベルを正しく維持するように注意してください。

    **Python**

    ```python
   # Initialize the project client
   projectClient = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_connection,
        )
    ```

    **C#**

    ```csharp
   // Initialize the project client
   DefaultAzureCredentialOptions options = new()
       { ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true };
   var projectClient = new AIProjectClient(
        new Uri(project_connection),
        new DefaultAzureCredential(options));
    ```

1. コメント**チャット クライアントの取得**を探して、次のコードを追加し、モデルとチャットするためのクライアント オブジェクトを作成します。

    **Python**

    ```python
   # Get a chat client
   chat = projectClient.inference.get_chat_completions_client()
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

    > **注**: このコードは、Azure AI Foundry プロジェクト クライアントを使用して、プロジェクトに関連付けられている既定の Azure AI モデル推論サービス エンドポイントへの安全な接続を作成します。 また、Azure AI モデル推論 SDK を使用し、Azure AI Foundry ポータルまたは Azure portal の対応する Azure AI Services リソース ページでサービス接続に表示されるエンドポイント URI を指定し、認証キーまたは Entra 資格情報トークンを使用して、エンドポイントに*直接*接続することもできます。 Azure AI モデル推論サービスへの接続の詳細については、「[Azure AI モデル推論 API](https://learn.microsoft.com/azure/machine-learning/reference-model-inference-api)」をご覧ください。

1. コメント**プロンプトの初期化**を探して、次のコードを追加し、システム プロンプトを使用してメッセージのコレクションを初期化します。

    **Python**

    ```python
   # Initialize prompt with system message
   prompt=[
            SystemMessage("You are a helpful AI assistant that answers questions.")
        ]
    ```

    **C#**

    ```csharp
   // Initialize prompt with system message
   var prompt = new List<ChatRequestMessage>(){
                    new ChatRequestSystemMessage("You are a helpful AI assistant that answers questions.")
                };
    ```

1. コードには、ユーザーが「quit」と入力するまでプロンプトを入力できるようにするループが含まれていることに注意してください。 次に、ループ セクションで、コメント **チャット完了の取得**を探して、次のコードを追加し、ユーザー入力をプロンプトに追加し、モデルから完了を取得し、プロンプトに完了を追加します (今後の反復のためにチャット履歴を保持します)。

    **Python**

    ```python
   # Get a chat completion
   prompt.append(UserMessage(input_text))
   response = chat.complete(
        model=model_deployment,
        messages=prompt)
   completion = response.choices[0].message.content
   print(completion)
   prompt.append(AssistantMessage(completion))
    ```

    **C#**

    ```csharp
   // Get a chat completion
   prompt.Add(new ChatRequestUserMessage(input_text));
   var requestOptions = new ChatCompletionsOptions()
   {
       Model = model_deployment,
       Messages = prompt
   };

   Response<ChatCompletions> response = chat.Complete(requestOptions);
   var completion = response.Value.Content;
   Console.WriteLine(completion);
   prompt.Add(new ChatRequestAssistantMessage(completion));
    ```

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。

### Azure にサインインしてアプリを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Azure にサインインします。

    ```
   az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。
    
1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、Azure AI Foundry ハブを含むサブスクリプションを選択します。
1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. メッセージが表示されたら、`What is the fastest animal on Earth?` などの質問を入力し、生成 AI モデルからの応答を確認します。
1. `Where can I see one?`や`Are they endangered?`など、フォローアップの質問をお試しください。 各反復の背景情報であるチャット履歴を使用して、会話を続行する必要があります。
1. 終了したら、`quit` を入力してプログラムを終了します。

> **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

## まとめ

この演習では、Azure AI Foundry SDK を使用して、Azure AI Foundry プロジェクトにデプロイした生成 AI モデル用のクライアント アプリケーションを作成しました。

## クリーンアップ

Azure AI Foundry ポータルを確認し終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. [Azure ポータル](https://portal.azure.com)を開き、この演習で使用したリソースをデプロイしたリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
