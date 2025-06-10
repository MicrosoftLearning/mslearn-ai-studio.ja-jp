---
lab:
  title: 独自のデータを使用する生成 AI アプリを作成する
  description: 検索拡張生成 (RAG) モデルを使用して、独自のデータを使ってプロンプトをグラウンディングするチャット アプリを構築する方法について説明します。
---

# 独自のデータを使用する生成 AI アプリを作成する

取得拡張生成 (RAG) は、カスタム データ ソースからのデータを、生成 AI モデルのプロンプトに統合するアプリケーションを構築するために使用される手法です。 RAG は、生成 AI アプリを開発するために一般的に使用されるパターンです。チャット ベースのアプリケーションでは、言語モデルを使用して入力を解釈し、適切な応答を生成します。

この演習では Azure AI Foundry を使用して、カスタム データを生成 AI ソリューションに統合します。

この演習は約 **45** 分かかります。

> **注**: この演習はプレリリース サービスに基づいていますが、このサービスは変更される可能性があります。

## Azure AI Foundry ハブとプロジェクトを作成する

この演習で使用する Azure AI Foundry の機能には、Azure AI Foundry *ハブ* リソースに基づくプロジェクトが必要です。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ブラウザーで `https://ai.azure.com/managementCenter/allResources` に移動し、**[作成]** を選択します。 次に、新しい **AI ハブ リソース**を作成するオプションを選択します。
1. **[プロジェクトの作成]** ウィザードで、プロジェクトの有効な名前を入力します。既存のハブが推奨される場合は、新しいハブを作成するオプションを選択し、**[詳細オプション]** を展開してプロジェクトの以下の設定を指定します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **ハブ名**: ハブの有効な名前
    - **場所**: 米国東部 2 またはスウェーデン中部\*

    > \* 一部の Azure AI リソースは、リージョンのモデル クォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. プロジェクトが作成されるまで待ちます。

## モデルをデプロイする

ソリューションを実装するには、次の 2 つのモデルが必要です。

- 効率的なインデックス作成と処理のためにテキスト データをベクター化する "埋め込み" モデル。**
- ご利用のデータに基づいて、質問に対する自然言語の応答を生成することができるモデル。

1. Azure AI Foundry ポータルで、プロジェクトの左側のナビゲーション ウィンドウの **[マイ アセット]** で、**[モデル + エンドポイント]** ページを選択します。
1. デプロイ モデル ウィザードで **[カスタマイズ]** を選択して、以下の設定で **text-embedding-ada-002** モデルの新しいデプロイを作成します。

    - **デプロイ名**: モデル デプロイの有効な名前**
    - **デプロイの種類**: グローバル標準
    - **モデルのバージョン**: *Select the default version (既定のバージョンの選択)*
    - **接続先 AI リソース**: *以前に作成したリソースを選択します*
    - **1 分あたりのトークンのレート制限 (1,000)**: 50,000 * (または 50,000 未満の場合はサブスクリプションで使用可能な最大値)*
    - **コンテンツ フィルター**: DefaultV2

    > **注**: 現在の AI リソースの場所に、デプロイするモデルで使用可能なクォータがない場合は、新しい AI リソースが作成され、プロジェクトに接続される別の場所を選択するように求められます。

1. **[モデル + エンドポイント]** ページに戻り、前の手順を繰り返し、TPM レート制限が **50K** (または、50K 未満の場合はサブスクリプションで使用可能な最大値) の最新バージョンの **グローバル標準**デプロイを使用して、**gpt-4o** モデルをデプロイします。

    > **注**:1 分あたりのトークン数 (TPM) を減らすと、ご利用のサブスクリプション内で使用可能なクォータが過剰に消費されるのを回避するのに役立ちます。 この演習で使用するデータには、50,000 TPM で十分です。

## プロジェクトにデータを追加する

ご利用のアプリのデータは、架空の旅行代理店 *Margie's Travel* の旅行パンフレット (PDF 形式) のセットで構成されています。 それらをプロジェクトに追加しましょう。

1. 新しいブラウザー タブで、[パンフレットの zip 形式アーカイブ](https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip)を `https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip` からダウンロードし、ローカル ファイル システム上の「**brochures**」という名前のフォルダーに展開します。
1. Azure AI Foundry ポータル内にあるプロジェクトの左側ナビゲーション ウィンドウ内で、**[マイ アセット]** の下にある **[データ + インデックス]** ページを選択します。
1. **[+ New data]\(+ 新しいデータ\)** を選択します。
1. **[データの追加]** ウィザードで、ドロップダウン メニューを展開して **[Upload files/folders]\(ファイル/フォルダーのアップロード\)** を選択します。
1. **[フォルダーのアップロード]** を選択し、**brochures** フォルダーを選択します。 フォルダー内のすべてのファイルが一覧表示されるまで待ちます。
1. **[次へ]** を選択して、データ名を `brochures` に設定します。
1. フォルダーがアップロードされるまで待ちます。これにはいくつかの .pdf ファイルが含まれています。

## データのインデックスを作成する

これで、ご利用のプロジェクトにデータ ソースを追加したので、それを使用して Azure AI 検索リソース内にインデックスを作成することができます。

1. Azure AI Foundry ポータル内にあるプロジェクトの左側ナビゲーション ウィンドウ内で、**[マイ アセット]** の下にある **[データ + インデックス]** ページを選択します。
1. **[インデックス]** タブで、次の設定で新しいインデックスを追加します。
    - **ソースの場所**:
        - **データ ソース**: Azure AI Foundry のデータ
            - [データ ソース] で **[brochures]** を選択します**
    - **インデックスの構成**:
        - **Azure AI 検索サービスの選択**: *次の設定で新しい Azure AI 検索リソースを作成します*。
            - **サブスクリプション**: *お使いの Azure サブスクリプション*
            - **リソース グループ**: *お使いの AI ハブと同じリソース グループ*
            - **サービス名**: *お使いの AI 検索リソースの有効な名前*
            - **場所**: *お使いの AI ハブと同じ場所*
            - **価格レベル**: Basic
            
            AI 検索リソースが作成されるまで待ちます。 次に、Azure AI Foundry に戻り、**[その他の Azure AI 検索リソースへの接続]** を選択し、先ほど作成した AI 検索リソースへの接続を追加して、インデックスの構成を完了します。
 
        - **ベクトル インデックス**: `brochures-index`
        - **仮想マシン**:自動選択
    - **[検索設定]**:
        - **[Vector settings]**:ベクトル検索をこの検索リソースに追加します
        - **Azure OpenAI 接続**: *ハブの既定の Azure OpenAI リソースを選択します。*
        - **埋め込みモデル**: text-embedding-ada-002
        - **埋め込みモデルのデプロイ**: ** text-embedding-ada-002 *モデルのデプロイ*

1. ベクトル インデックスを作成し、そのインデックス作成プロセスが完了するまで待ちます。サブスクリプションで使用可能なコンピューティング リソースによっては、しばらく時間がかかる場合があります。

    このインデックス作成操作は、次のジョブで構成されます。

    - ご利用の brochures データ内でテキスト トークンを分解し、塊に分けて、埋め込みます。
    - Azure AI 検索インデックスを作成します。
    - インデックス資産を登録します。

    > **ヒント**: インデックスが作成されるのを待っている間に、ダウンロードしたパンフレットを見て内容を把握しましょう。

## プレイグラウンドでインデックスをテストする

RAG ベースのプロンプト フローでインデックスを使用する前に、それを使用して生成 AI 応答に作用することができるのを確認しましょう。

1. 左側ナビゲーション ウィンドウ内で、**[プレイグラウンド]** ページを選択し、**[チャット]** プレイグラウンドを開きます。
1. [チャット] プレイグラウンド ページの [セットアップ] ウィンドウで、**GPT-4** モデル デプロイが選択されていることを確認します。 次に、メインの [チャット セッション] パネルで、プロンプト「`Where can I stay in New York?`」を送信します。
1. その応答を確認します。これは、インデックスからのデータを含まない、このモデルからの一般的な回答であるはずです。
1. [セットアップ] ウィンドウで、**"データの追加"** フィールドを展開し、**brochures-index** プロジェクト インデックスを追加して、**[ハイブリッド (ベクトル + キーワード)]** 検索タイプを選択します。

   > **ヒント**: 場合によっては、新しく作成されたインデックスをすぐに使用できない場合があります。 通常はブラウザーの更新で解決しますが、インデックスが見つからない問題がそれでも解決しない場合は、インデックスが認識されるまで待つ必要がある場合があります。

1. インデックスが追加され、チャット セッションが再開された後に、プロンプト「`Where can I stay in New York?`」を再送信します。
1. その応答を確認します。インデックス内のデータに基づいているはずです。

<!-- DEPRECATED STEPS

## Create a RAG client app with the Azure AI Foundry and Azure OpenAI SDKs

Now that you have a working index, you can use the Azure AI Foundry and Azure OpenAI SDKs to implement the RAG pattern in a client application. Let's explore the code to accomplish this in a simple example.

> **Tip**: You can choose to develop your RAG solution using Python or Microsoft C#. Follow the instructions in the appropriate section for your chosen language.

### Prepare the application configuration

1. In the Azure AI Foundry portal, view the **Overview** page for your project.
1. In the **Project details** area, note the **Project connection string**. You'll use this connection string to connect to your project in a client application.
1. Return to the browser tab containing the Azure portal (keeping the Azure AI Foundry portal open in the existing tab).
1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a ***PowerShell*** environment with no storage in your subscription.

    The cloud shell provides a command-line interface in a pane at the bottom of the Azure portal. You can resize or maximize this pane to make it easier to work in.

    > **Note**: If you have previously created a cloud shell that uses a *Bash* environment, switch it to ***PowerShell***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).

    **<font color="red">Ensure you've switched to the classic version of the cloud shell before continuing.</font>**

1. In the cloud shell pane, enter the following commands to clone the GitHub repo containing the code files for this exercise (type the command, or copy it to the clipboard and then right-click in the command line and paste as plain text):

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **Tip**: As you paste commands into the cloudshell, the output may take up a large amount of the screen buffer. You can clear the screen by entering the `cls` command to make it easier to focus on each task.

1. After the repo has been cloned, navigate to the folder containing the chat application code files:

    > **Note**: Follow the steps for your chosen programming language.

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/c-sharp
    ```

1. In the cloud shell command-line pane, enter the following command to install the libraries you'll use:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-projects azure-identity openai
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```
    

1. Enter the following command to edit the configuration file that has been provided:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    The file is opened in a code editor.

1. In the code file, replace the following placeholders: 
    - **your_project_connection_string**: Replace with the connection string for your project (copied from the project **Overview** page in the Azure AI Foundry portal).
    - **your_gpt_model_deployment** Replace with the name you assigned to your **gpt-4o** model deployment.
    - **your_embedding_model_deployment**: Replace with the name you assigned to your **text-embedding-ada-002** model deployment.
    - **your_index**: Replace with your index name (which should be `brochures-index`).
1. After you've replaced the placeholders, in the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

### Explore code to implement the RAG pattern

1. Enter the following command to edit the code file that has been provided:

    **Python**

    ```
   code rag-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. Review the code in the file, noting that it:
    - Uses the Azure AI Foundry SDK to connect to your project (using the project connection string)
    - Creates an authenticated Azure OpenAI client from your project connection.
    - Retrieves the default Azure AI Search connection from your project so it can determine the endpoint and key for your Azure AI Search service.
    - Creates a suitable system message.
    - Submits a prompt (including the system and a user message based on the user input) to the Azure OpenAI client, adding:
        - Connection details for the Azure AI Search index to be queried.
        - Details of the embedding model to be used to vectorize the query\*.
    - Displays the response from the grounded prompt.
    - Adds the response to the chat history.

    \* *The query for the search index is based on the prompt, and is used to find relevant text in the indexed documents. You can use a keyword-based search that submits the query as text, but using a vector-based search can be more efficient - hence the use of an embedding model to vectorize the query text before submitting it.*

1. Use the **CTRL+Q** command to close the code editor without saving any changes, while keeping the cloud shell command line open.

### Run the chat application

1. In the cloud shell command-line pane, enter the following command to run the app:

    **Python**

    ```
   python rag-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. When prompted, enter a question, such as `Where should I go on vacation to see architecture?` and review the response from your generative AI model.

    Note that the response includes source references to indicate the indexed data in which the answer was found.

1. Try a follow-up question, for example `Where can I stay there?`

1. When you're finished, enter `quit` to exit the program. Then close the cloud shell pane.

-->

## クリーンアップ

不要な Azure のコストとリソース使用を回避するには、この演習でデプロイしたリソースを削除する必要があります。

1. Azure AI Foundry の確認が完了したら、[Azure portal](https://portal.azure.com) (`https://portal.azure.com`) に戻ます。必要に応じて、ご自身の Azure 資格情報を使用してサインインします。 次に、Azure AI 検索と Azure AI リソースをプロビジョニングしたリソース グループ内のリソースを削除します。
