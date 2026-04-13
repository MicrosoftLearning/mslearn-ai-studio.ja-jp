---
lab:
  title: 生成 AI チャット アプリを作成する
  description: OpenAI SDK と Responses API を使用して、Microsoft Foundry にデプロイされたモデルに接続するチャット アプリを構築する方法について説明します。
  level: 300
  duration: 45
---

# 生成 AI チャット アプリを作成する

この演習では、OpenAI SDK と Responses API を使用して、Microsoft Foundry プロジェクトにデプロイされたモデルに接続するチャット アプリを作成します。

この演習は約 **45** 分かかります。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

この演習を開始するには、以下のものが必要です。

- 有効な [Azure サブスクリプション](https://azure.microsoft.com/pricing/purchase-options/azure-account)
- [Visual Studio Code](https://code.visualstudio.com/) がインストールされていること
- [Python バージョン**3.13.xx**](https://www.python.org/downloads/release/python-31312/) がインストール済み\*
- [Git](https://git-scm.com/install/) がインストールおよび構成済み
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) がインストールされていること

> \* Python 3.14 を使用できますが、一部の依存関係がそのリリース用にまだコンパイルされていません。 このラボでは Python 3.13.12 でテストが正常に終了しました。

## Microsoft Foundry プロジェクトを作成する

Microsoft Foundry では "プロジェクト" を使って、AI ソリューションの開発に使われるモデル、リソース、データ、その他の資産を整理します。

1. Web ブラウザーで、[Microsoft Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインすると開くヒントやクイック スタートのペインをすべて閉じ、必要な場合は、左上にある Foundry のロゴを使ってホーム ページに移動します。

1. まだ有効になっていない場合は、ページ上部のツール バーで **[新しい Foundry]** オプションを有効にします。 次に、新しいプロジェクトを作成するための画面が表示された場合は、一意の名前を指定して作成します。このときに **[高度なオプション]** 領域を展開して、プロジェクトの設定を次のとおりに指定します。
    - **Foundry リソース**: "リソースの既定の名前を使用します (通常は {project_name}-resource)"**
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **[リージョン]**: **[AI Foundry 推奨]** のリージョンのいずれかを選択します。

1. プロジェクトが作成されるまで待ちます。 次に、そのホーム ページを表示します。

## モデルをデプロイする

次に、チャット アプリケーションで使用するモデルをデプロイします。

1. **[ビルドの開始]** メニューで、**[モデルの検索]** を選択して (または **[検出]** ページで **[モデル]** タブを選択して) Microsoft Foundry モデル カタログを表示します。
1. モデル カタログで、`gpt-4.1` を検索します。
1. モデル カードを確認し、既定の設定を使用してデプロイします。
1. モデルがデプロイされると、モデルのプレイグラウンドで開きます。必要に応じてそこでモデルのテストを行うことができます。

## エンドポイントを取得する

クライアント アプリケーションからモデルに接続するには、エンドポイントが必要です。 この演習では、OpenAI SDK を使用してモデルとチャットします。また、Azure OpenAI エンドポイントと Entra ID 認証を使用して接続します。

> **注**: Entra ID 認証の代わりに、プロジェクトに API キーを使用できます。 可能な限り、Entra ID 認証を使用することをお勧めします。

1. メニュー バーで、**[ホーム]** ページを選択します。
1. そこに **Azure OpenAI エンドポイント**が表示されていることに注目します。

    > **ヒント**: この演習では、**Azure OpenAI エンドポイント**を使用します。プロジェクト エンドポイントは<u>使用しません</u>。

## モデルとチャットするクライアント アプリケーションを作成する

これでモデルをデプロイしたので、OpenAI SDK と Responses API を使用して、モデルとチャットするアプリケーションを開発できます。

### GitHub からアプリケーション ファイルを取得する

チャット アプリケーションの開発に必要な初期アプリケーション ファイルが、GitHub リポジトリに提供されます。

1. Visual Studio Code を開きます。
1. コマンド パレット (*Ctrl + Shift + P*) を開き、`Git:clone` コマンドを使用して、`https://github.com/microsoftlearning/mslearn-ai-studio` リポジトリをローカル フォルダーにクローンします (どのフォルダーでもかまいません)。 次に、それを開きます。

    作成者を信頼することを確認するメッセージが表示される場合があります。

### アプリケーション構成を準備する

1. Visual Studio Code で **[拡張機能]** ペインを表示します。まだインストールされていない場合は、**Python** 拡張機能をインストールします。
1. **コマンド パレット**の `python:select interpreter` コマンドを使用します。 次に、Python 3.13 のインストールに基づいて新しい **Venv** 環境を作成します。

    > **ヒント**: 依存関係のインストールを求めるダイアログが表示された場合は、*/labfiles/foundry-chat/python/chat-app* フォルダーの*requirements.txt* ファイルに記載されているものをインストールできますが、インストールしない場合でも後でインストールするので問題ありません。

1. [エクスプローラー] ペインで、**/labfiles/foundry-chat/python/chat-app** にあるアプリケーション コード ファイルを含むフォルダーに移動します。 アプリケーション フローには以下が含まれます。
    - **.env** (アプリケーション構成ファイル)
    - **requirements.txt** (インストールする必要がある Python パッケージの依存関係)
    - **chat-app.py** (チャット アプリケーションのコード ファイル)
    - **chat-async.py** (アプリケーションの非同期バージョンのコード ファイル)

1. **[エクスプローラー]** ペインで、アプリケーション ファイルを含む **chat-app** フォルダーを右クリックし、**[統合ターミナルで開く]** (または **[ターミナル]** メニューでターミナルを開き、*/labfiles/foundry-chat/python/chat-app* フォルダーに移動します)。

    > **注**: Visual Studio Code でターミナルを開くと、Python 環境が自動的にアクティブ化されます。 お使いのシステムでスクリプトの実行を有効にする必要がある場合があります。

1. **labfiles/foundry-chat/python/chat-app** フォルダーで、作成した Python 環境がアクティブであることを示すプレフィックス **(.venv)** 付きでターミナルが開かれていることを確認します。
1. 次のコマンドを実行して、OpenAI SDK、Azure ID、およびその他の必要なパッケージをインストールします。

    ```
    pip install -r requirements.txt
    ```

1. **[エクスプローラー]** ペインの **labfiles/foundry-chat/python/chat-app** フォルダーで、**.env** ファイルを選択して開きます。 次に、構成値を更新して、**Azure OpenAI エンドポイント**と **gpt-4.1** モデルのデプロイを含めます。

    > **ヒント**: Foundry ポータルのプロジェクト ホーム ページから **Azure OpenAI エンドポイント** (プロジェクト エンドポイントではありません) をコピーし、デプロイの名前が *gpt-4.1* でない場合は、モデル デプロイの名前を変更します。

    変更した構成ファイルを保存します。

### *ChatCompletions* API を使用してモデルとチャットする

*ChatCompletions* API は、大規模言語モデル用のクライアント アプリケーションを構築するための確立された方法であり、広く採用されています。

1. **[エクスプローラー]** ペインの **labfiles/foundry-chat/python/chat-app** フォルダーで、**chat-app.py** ファイル (*chat-async.py* <u>ではありません</u>) を選択して開きます。
1. 既存のコードを確認します。 OpenAI SDK を使用してモデルにアクセスするコードを追加します。

    > **ヒント**: コード ファイルにコードを追加する際は、必ず正しいインデントを維持してください。

1. コード ファイルの上部にある既存の名前空間参照の下で、**Import namespaces (名前空間をインポートする)** というコメントを検索し、OpenAI SDK を使用するために必要な名前空間をインポートする次のコードを追加します。

    ```python
   # import namespaces
   from openai import OpenAI
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
    ```

1. **main** 関数では、構成ファイルからエンドポイントとキーを読み込むためのコードが既に提供されていることに注目してください。 次に、**Initialize the OpenAI client (OpenAI クライアントを初期化する)** というコメントを見つけ、次のコードを追加して、OpenAI API 用のクライアントを作成します。

    ```python
   # Initialize the OpenAI client
   token_provider = get_bearer_token_provider(
        DefaultAzureCredential(), "https://ai.azure.com/.default"
   )
    
   openai_client = OpenAI(
        base_url=azure_openai_endpoint,
        api_key=token_provider
   )
    ```

1. **main** 関数で、ユーザーがアプリを終了するまでユーザー プロンプトを要求するコードが提供されていることに注目してください。 このループ内で、**Get a response (応答を取得する)** コメントを検索し、次のコードを追加します。

    ```python
   # Get a response
   completion = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {
                "role": "system",
                "content": "You are a helpful AI assistant that answers questions and provides information."
            },
            {
                "role": "user",
                "content": input_text
            }
        ]
   )
   print(completion.choices[0].message.content)
    ```

    *ChatCompletions* API は、*messages* の JSON コレクションを使用して会話をカプセル化します。 多くの場合、これらは、モデルに指示を提供する "システム プロンプト" と、ユーザーの入力を含む "ユーザー プロンプト" で構成されます。****

1. コード ファイルに加えた変更を保存します。 次に、ターミナル ウィンドウで、次のコマンドを使用して Azure にサインインします。

    ```powershell
    az login
    ```

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。

1. メッセージが表示されたら、指示に従って Azure にサインインします。 次に、コマンド ラインでサインイン プロセスを完了し、Foundry リソースを含むサブスクリプションの詳細を表示 (必要に応じて確認) します。
1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```powershell
   python chat-app.py
    ```

    プログラムはターミナルで実行されている必要があります (そうでない場合は、エラーを解決してからもう一度お試しください)。

1. メッセージが表示されたら、次のプロンプトを入力します。

    ```input
    Tell me about the ELIZA chatbot.
    ```

    しばらくすると、アプリは 1960 年代に作成された ELIZA チャットボットに関するいくつかの情報を応答します。

1. プロンプト `quit` を入力して、アプリケーションを終了します。

### *Responses* API を使用してモデルとチャットする

*ChatCompletions* API は広く使用されていますが、新しい *Responses* API に置き換えられる傾向にあります。 それを使用するようにコードを更新しましょう。

1. **chat-app.py** コードの **main** 関数で、**Get a response (応答を取得する)** コメントの下のコードを、*Responses* API を使用する次のコードに置き換えます。

    ```python
   # Get a response
   response = openai_client.responses.create(
                model=model_deployment,
                instructions="You are a helpful AI assistant that answers questions and provides information.",
                input=input_text
   )
   print(response.output_text)
    ```

    システム メッセージが *instructions* パラメーターに、ユーザー プロンプトが *input* パラメーターに割り当てられている、より単純な構文に注目してください。

1. コードへの変更を保存し、ターミナル ウィンドウでアプリケーション (`python chat-app.py`) を再実行します。
1. ダイアログが表示されたら、前と同じプロンプトを入力します。

    ```input
    Tell me about the ELIZA chatbot.
    ```

    しばらくすると、アプリは再び ELIZA チャットボットに関する情報で応答します。

1. 会話を続けるには、次のプロンプトを入力します。

    ```input
    How does it compare to modern LLMs?
    ```

    このアプリは、"it" が何を指しているのかを理解していないことを示す方法で応答します。 会話のコンテキストが失われました。 修正しましょう。

1. プロンプト `quit` を入力して、アプリケーションを終了します。

### 会話の追跡を追加する

会話のコンテキストを維持するには、新しい要求ごとに以前の応答への参照を含める必要があります。

1. **chat-app.py** コードの **main** 関数で、**Loop until the user wants to quit (ユーザーが終了するまでループする)** というコメントを見つけ、次のコードをその<u>上</u>に追加します (Loop の "前" に)。**

    ```python
   # Track responses
   last_response_id = None
    ```

1. **Get a response (応答を取得する)** コメントの下のコードを変更し、要求時に前回の応答 ID を渡すようにします。その後、新しい応答 ID を取得して次回に追加できるようにします。

    ```python
   # Get a response
   response = openai_client.responses.create(
                model=model_deployment,
                instructions="You are a helpful AI assistant that answers questions and provides information.",
                input=input_text,
                previous_response_id=last_response_id,
   )
   print(response.output_text)
   last_response_id = response.id
    ```

    この手法を使用すると、コンテキストを維持するために前の応答の ID を渡すことができます。 また、会話を誘導したり、以前の会話スレッドを再開したりするために、任意の以前の応答から ID を渡す、より複雑なロジックを実装することもできます。

1. コードへの変更を保存し、ターミナル ウィンドウでアプリケーション (`python chat-app.py`) を再実行します。
1. ダイアログが表示されたら、前と同じプロンプトを入力します。

    ```input
    Tell me about the ELIZA chatbot.
    ```

    しばらくすると、アプリは再び ELIZA チャットボットに関する情報で応答します。

1. 会話を続けるには、次のプロンプトを入力します。

    ```input
    How does it compare to modern LLMs?
    ```

    今回は、ELIZA チャットボットと最新の LLM の比較を使ってアプリが応答する必要があります。 応答は非常に長い場合があり、アプリはモデルからすべての応答を受けとるまで待機してから表示します。これにより、アプリが応答していないように見えることがあります。 次は、これを修正しましょう。

1. プロンプト `quit` を入力して、アプリケーションを終了します。

### "ストリーミング" 応答を実装する**

長い応答を処理するために "ストリーミング" を使用すると、完全なテキストが返される前に部分的な応答の処理を開始できます。**

1. **chat-app.py** コードの **main** 関数で、**Get a response (応答を取得する)** コメントの下のコードを、"ストリーミング" を使用する次のコードに置き換えます。**

    ```python
   # Get a response
   stream = openai_client.responses.create(
                model=model_deployment,
                instructions="You are a helpful AI assistant that answers questions and provides information.",
                input=input_text,
                previous_response_id=last_response_id,
                stream=True
   )
   for event in stream:
        if event.type == "response.output_text.delta":
            print(event.delta, end="")
        elif event.type == "response.completed":
            last_response_id = event.response.id
   print()
    ```

    *stream=True* パラメーターは、新しいチャンク (または "差分") の処理の準備が整うと "イベント" が発生するストリーム化された応答を作成します。****

1. コードへの変更を保存し、ターミナル ウィンドウでアプリケーション (`python chat-app.py`) を再実行します。
1. ダイアログが表示されたら、前と同じプロンプトを入力します。

    ```input
    Tell me about the ELIZA chatbot.
    ```

    しばらくすると、アプリは ELIZA チャットボットに関するいくつかの情報を含む応答を開始するはずです。 各チャンクが返されると、応答が増分的に表示されます。

1. 会話を続けるには、次のプロンプトを入力します。

    ```input
    How does it compare to modern LLMs?
    ```

    ここでも、応答は段階的に表示されるはずです。

1. プロンプト `quit` を入力して、アプリケーションを終了します。

### 非同期 API を使用する

OpenAI SDK には、実行時間の長いモデルまたはエージェント操作を使用するときにアプリケーションの応答性を高めることができる非同期オプションが用意されています。

1. **[エクスプローラー]** ペインの **labfiles/foundry-chat/python/chat-app** フォルダーで、**chat-async.py** ファイル (*chat-app.py* <u>ではありません</u>) を選択して開きます。
1. 既存のコードを確認します。 OpenAI SDK 非同期 API を使用してモデルにアクセスするコードを追加します。

    > **ヒント**: コード ファイルにコードを追加する際は、必ず正しいインデントを維持してください。

1. コード ファイルの上部にある既存の名前空間参照の下で、**Import namespaces (名前空間をインポートする)** というコメントを検索し、OpenAI SDK を使用するために必要な名前空間をインポートする次のコードを追加します。

    ```python
   # import namespaces for async
   import asyncio
   from openai import AsyncOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    ```

1. **main** 関数では、構成ファイルからエンドポイントとキーを読み込むためのコードが既に提供されていることに注目してください。 次に、**Initialize the async OpenAI client (非同期 OpenAI クライアントを初期化する)** というコメントを見つけ、次のコードを追加して、OpenAI API 用のクライアントを作成します。

    ```python
   # Initialize an async OpenAI client
   credential = DefaultAzureCredential()
   token_provider = get_bearer_token_provider(
    credential, "https://ai.azure.com/.default"
   )

   async_client = AsyncOpenAI(
        base_url=azure_openai_endpoint,
        api_key=token_provider
   )
    ```

1. **main** 関数で、ユーザーがアプリを終了するまでユーザー プロンプトを要求するコードが提供されていることに注目してください。 このループ内で、**Await an asynchronous response (非同期応答を待機する)** コメントを検索し、次のコードを追加します。

    ```python
   # Await an asynchronous response
   response = await async_client.responses.create(
                model=model_deployment,
                instructions="You are a helpful AI assistant that answers questions and provides information.",
                input=input_text,
                previous_response_id=last_response_id
   )
   assistant_text = response.output_text
   print("Assistant:", assistant_text)
   last_response_id = response.id
    ```

    このコードは、モデルからの非同期応答を待機します。

1. **main** 関数の末尾にある **finally** ブロックで、**Close the async client session (非同期クライアント セッションを閉じる)** というコメントを見つけ、次のコードを追加して非同期クライアントを閉じます。

    ```python
   # Close the async client session
    await credential.close()
    ```

1. コード ファイルに加えた変更を保存します。 次に [ターミナル] ペインで、次のコマンドを入力して、プログラムを実行します。

    ```powershell
   python chat-async.py
    ```

    プログラムはターミナルで実行されている必要があります (そうでない場合は、エラーを解決してからもう一度お試しください)。

1. メッセージが表示されたら、次のプロンプトを入力します。

    ```input
    Tell me about the Turing test.
    ```

    しばらくすると、アプリはチューリング テストに関するいくつかの情報で応答するはずです。

1. プロンプト `quit` を入力して、アプリケーションを終了します。

## まとめ

この演習では、OpenAI SDK、*ChatCompletions*、*Responses* API を使用して、Microsoft Foundry プロジェクトにデプロイした生成 AI モデルのクライアント アプリケーションを作成しました。 会話のコンテキストを追跡し、ストリーミングを実装して応答性の高いチャット エクスペリエンスを提供して、モデルの動作をカスタマイズしました。

## クリーンアップ

Microsoft Foundry を確認し終わったら、不要な Azure のコストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. [Azure ポータル](https://portal.azure.com)を開き、この演習で使用したリソースをデプロイしたリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
