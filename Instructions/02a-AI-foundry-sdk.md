---
lab:
  title: 生成 AI チャット アプリを作成する
  description: Azure AI Foundry SDK を使用して、プロジェクトに接続して言語モデルとチャットするアプリを構築する方法について説明します。
---

# 生成 AI チャット アプリを作成する

この演習では、Azure AI Foundry SDK を使用して、プロジェクトに接続して言語モデルとチャットするシンプルなチャット アプリを作成します。

この演習は約 **40** 分かかります。

> **注**: この演習は、変更される可能性があるプレリリース SDK に基づいています。 必要に応じて、特定のバージョンのパッケージを使用しました。利用可能な最新バージョンが反映されていない可能性があります。 予期しない動作、警告、またはエラーが発生する場合があります。

## Azure AI Foundry プロジェクトを作成する

まず、Azure AI Foundry プロジェクトを作成します。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ホーム ページで、**[+ 作成]** を選択します。
1. **[プロジェクトの作成]** ウィザードで、有効なプロジェクト名を入力し、既存のハブが推奨された場合は、新しいハブを作成するオプションを選択します。 次に、ハブとプロジェクトをサポートするために自動的に作成される Azure リソースを確認します。
1. **[カスタマイズ]** を選択し、ハブに次の設定を指定します。
    - **ブ名**: *ハブの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します。*
    - **場所**: **[選択に関するヘルプ]** を選択し、次に [場所ヘルパー] ウィンドウで **gpt-4o** を選択し、推奨されるリージョンを選択します\*
    - **Azure AI サービスまたは Azure OpenAI への接続**: *新しい AI サービス リソースを作成します*
    - **Azure AI 検索への接続**:接続をスキップする

    > \* Azure OpenAI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[次へ]** を選択し、構成を確認します。 **[作成]** を選択し、プロセスが完了するまで待ちます。
1. プロジェクトが作成されたら、表示されているヒントをすべて閉じて、Azure AI Foundry ポータルのプロジェクト ページを確認します。これは次の画像のようになっているはずです。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./media/ai-foundry-project.png)

## 生成 AI モデルを展開する

これで、チャット アプリケーションをサポートする生成 AI 言語モデルをデプロイする準備ができました。 この例では、OpenAI GPT-4o モデルを使用します。しかし、原則はどのモデルでも同じです。

1. Azure AI Foundry プロジェクト ページの右上にあるツール バーで、**[プレビュー機能]** (**&#9215**) アイコンを使用して、**[Azure AI モデル推論サービスにモデルをデプロイする]** 機能を有効にします。 この機能により、アプリケーション コードで使用する Azure AI 推論サービスでモデル デプロイを使用できるようになります。
1. プロジェクトの左側のウィンドウの **[マイ アセット]** セクションで、**[モデル + エンドポイント]** ページを選択します。
1. **[モデル + エンドポイント]** ページの **[モデル デプロイ]** タブの **[+ モデルのデプロイ]** メニューで、**[基本モデルのデプロイ]** を選択します。
1. 一覧で **GPT-4o** モデルを検索してから、それを選択して確認します。
1. デプロイの詳細で **[カスタマイズ]** を選択して、以下の設定でモデルをデプロイします。
    - **デプロイ名**: モデル デプロイの有効な名前**
    - **デプロイの種類**: グローバル標準
    - **バージョンの自動更新**: 有効
    - **モデルのバージョン**: *利用可能な最新バージョンを選択します*
    - **接続されている AI リソース**: *使用している Azure OpenAI リソース接続を選択します*
    - **1 分あたりのトークンのレート制限 (1,000)**: 50,000 * (または 50,000 未満の場合はサブスクリプションで使用可能な最大値)*
    - **コンテンツ フィルター**: DefaultV2

    > **注**:TPM を減らすと、ご利用のサブスクリプション内で使用可能なクォータが過剰に消費されることを回避するのに役立ちます。 この演習で使用するデータには、50,000 TPM で十分です。 使用可能なクォータがこれより低い場合は、演習を完了できますが、レート制限を超えるとエラーが発生する可能性があります。

1. デプロイが完了するまで待ちます。

## モデルとチャットするクライアント アプリケーションを作成する

これでモデルをデプロイしたので、Azure AI Foundry SDKと Azure AI モデル推論 SDK を使用して、モデルとチャットするアプリケーションを開発できます。

> **ヒント**: Python または Microsoft C# を使用してソリューションを開発することを選択できます。 選択した言語の適切なセクションの指示に従います。

### アプリケーション構成を準備する

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。
1. **[プロジェクトの詳細]** エリアで、**[プロジェクト接続文字列]** の内容を書き留めます。 この接続文字列を使用して、クライアント アプリケーションでプロジェクトに接続します。
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

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、これから使用するライブラリをインストールします。

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
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

1. コード ファイルで、**your_project_connection_string** プレースホルダーをプロジェクトの接続文字列 (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを GPT-4 モデル デプロイに割り当てた名前に置き換えます。
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
   projectClient = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   // Initialize the project client
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
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

### チャット アプリケーションを実行する

1. コード エディターの下の Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

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

## OpenAI SDK を使用する

クライアント アプリは、Azure AI モデル推論 SDK を使用して構築されています。つまり、Azure AI モデル推論サービスにデプロイされたどのモデルでも使用できます。 デプロイしたモデルは OpenAI GPT モデルであり、OpenAI SDK を使用して実行することもできます。

OpenAI SDK を使用してチャット アプリケーションを実装する方法を確認するために、いくつかのコードを変更してみましょう。

1. コード フォルダー (*python* または *c-sharp*) の Cloud Shell コマンド ラインで、次のコマンドを入力して必要なパッケージをインストールします。

    **Python**

    ```
   pip install openai
    ```

    **C#**

    ```
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.6
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

> **注**: Azure AI モデル推論 SDK との一部の非互換性に対する暫定的な回避策として、Azure.AI.Projects パッケージの別のプレリリース バージョンが必要です。

1. コード ファイル (*chat-app.py* または *Program.cs*) がまだ開いていない場合は、次のコマンドを入力してコード エディターで開きます。

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. コード ファイルの先頭に、次の参照を追加します。

    **Python**

    ```python
   import openai
    ```

    **C#**

    ```csharp
   using OpenAI.Chat;
   using Azure.AI.OpenAI;
    ```

1. コメント**チャット クライアントの取得**を探して、クライアント オブジェクトの作成に使用するコードを次のように変更します。

    **Python**

    ```python
   # Get a chat client 
   openai_client = projectClient.inference.get_azure_openai_client(api_version="2024-10-21")
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatClient openaiClient = projectClient.GetAzureOpenAIChatClient(model_deployment);
    ```

    > **注**: このコードは、Azure AI Foundry プロジェクト クライアントを使用して、プロジェクトに関連付けられている既定の Azure OpenAI サービス エンドポイントへの安全な接続を作成します。 また、Azure OpenAI SDK を使用し、Azure AI Foundry ポータルまたは Azure portal の対応する Azure OpenAI または AI Services リソース ページでサービス接続に表示されるエンドポイント URI を指定し、認証キーまたは Entra 資格情報トークンを使用して、エンドポイントに*直接*接続することもできます。 Azure OpenAI Service への接続の詳細については、「[Azure OpenAI でサポートされているプログラミング言語](https://learn.microsoft.com/azure/ai-services/openai/supported-languages)」をご覧ください。

1. コメント**システム メッセージを使用したプロンプトの初期化**を探して、次のようにコードを修正し、システム プロンプトを使用してメッセージのコレクションを初期化します。

    **Python**

    ```python
   # Initialize prompt with system message
   prompt=[
        {"role": "system", "content": "You are a helpful AI assistant that answers questions."}
    ]
    ```

    **C#**

    ```csharp
   // Initialize prompt with system message
    var prompt = new List<ChatMessage>(){
        new SystemChatMessage("You are a helpful AI assistant that answers questions.")
    };
    ```

1. コメント**チャット完了の取得**を探して、次のようにコードを変更し、プロンプトにユーザー入力を追加し、モデルから完了を取得して、プロンプトに完了を追加します。

    **Python**

    ```python
   # Get a chat completion
   prompt.append({"role": "user", "content": input_text})
   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=prompt)
   completion = response.choices[0].message.content
   print(completion)
   prompt.append({"role": "assistant", "content": completion})
    ```

    **C#**

    ```csharp
   // Get a chat completion
   prompt.Add(new UserChatMessage(input_text));
   ChatCompletion completion = openaiClient.CompleteChat(prompt);
   var completionText = completion.Content[0].Text;
   Console.WriteLine(completionText);
   prompt.Add(new AssistantChatMessage(completionText));
    ```

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。

1. コード エディターの下の Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 以前と同様に質問を送信して、アプリをテストします。 終了したら、`quit` を入力してプログラムを終了します。

    > **注**: Azure AI モデル推論 SDK と OpenAI SDK では、同様のクラスとコード コンストラクトが使用されるため、最小限の変更が必要です。 Azure AI モデル推論サービス エンドポイントにデプロイされる*どの*モデルでも Azure AI モデル推論 SDK を使用できます。 OpenAI SDK は OpenAI モデルでのみ機能しますが、Azure AI モデル推論サービス エンドポイントまたは Azure OpenAI エンドポイントにデプロイされたモデルに使用できます。  

## まとめ

この演習では、Azure AI Foundry SDK、Azure AI モデル推論、Azure OpenAI を使用して、Azure AI Foundry プロジェクトにデプロイした生成 AI モデル用のクライアント アプリケーションを作成しました。

## クリーンアップ

Azure AI Foundry ポータルを確認し終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
