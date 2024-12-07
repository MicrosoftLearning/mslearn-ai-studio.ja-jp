---
lab:
  title: Azure AI Studio でのプロンプト フローを使用したカスタム コパイロットの構築の概要
---

# Azure AI Studio でのプロンプト フローを使用したカスタム コパイロットの構築の概要

この演習では、Azure AI Studio のプロンプト フローを使用して、ユーザー プロンプトとチャット履歴を入力として使用し、Azure OpenAI の GPT モデルを使用して出力を生成するカスタム コパイロットを作成します。

> この演習を完了するには、Azure OpenAI サービスへのアクセスについて Azure サブスクリプションが承認されている必要があります。 [登録フォーム](https://learn.microsoft.com/legal/cognitive-services/openai/limited-access)に記入して、Azure OpenAI モデルへのアクセスをリクエストします。

プロンプト フローを使用してコパイロットを構築するには、次を実行する必要があります。

- Azure AI Studio の中で AI ハブとプロジェクトを作成します。
- GPT モデルをデプロイします。
- デプロイされた GPT モデルを使用して、ユーザーの入力に基づいて回答を生成するフローを作成します。
- フローをテストしてデプロイします。

## Azure AI Studio 内に AI ハブとプロジェクトを作成する

まずは、次のように Azure AI ハブの中で Azure AI Studio プロジェクトを作成します。

1. Web ブラウザーで [https://ai.azure.com](https://ai.azure.com) を開き、Azure の資格情報を使ってサインインします。
1. **[ホーム]** ページを選択してから、**[+ 新しいプロジェクト]** を選択します。
1. **[新しいプロジェクトの作成]** ウィザードで、次の設定を使ってプロジェクトを作成します。
    - **プロジェクト名**:"プロジェクトの一意の名前"**
    - **ハブ**:次の設定で新しいハブを作成します。**
        - **ハブ名**:*一意の名前*
        - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
        - **[リソース グループ]**: "新しいリソース グループ"**
        - **[場所]**: *以下のいずれかのリージョンから**ランダム**に選択する*\*
        - オーストラリア東部
        - カナダ東部
        - 米国東部
        - 米国東部 2
        - フランス中部
        - 東日本
        - 米国中北部
        - スウェーデン中部
        - スイス北部
        - 英国南部
    - **Azure AI サービスまたは Azure OpenAI への接続**:*新しい接続を作成する*
    - **Azure AI 検索への接続**:接続をスキップする

    > \* Azure OpenAI リソースは、リージョンのクォータによってテナント レベルで制限されます。 一覧表示されているリージョンには、この演習で使用されるモデル タイプの既定のクォータが含まれています。 リージョンをランダムに選択すると、1 つのリージョンがクォータ制限に達するリスクが軽減されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。 詳しくは、[リージョンごとのモデルの可用性](https://learn.microsoft.com/azure/ai-services/openai/concepts/models#gpt-35-turbo-model-availability)を参照してください

1. 構成を確認して、プロジェクトを作成します。
1. プロジェクトが作成されるまで 5 から 10 分待ちます。

## GPT モデルをデプロイする

プロンプト フロー内で言語モデルを使用するには、まずモデルを展開する必要があります。 Azure AI Studio を使うと、フローで使用できる OpenAI モデルをデプロイできます。

1. 左側のナビゲーション ウィンドウの **[コンポーネント]** で、**[デプロイ]** ページを選びます。
1. 次の設定で **gpt-35-turbo** モデルの新しいデプロイを作成します。
    - **デプロイ名**:"モデル デプロイの一意の名前"**
    - **モデルのバージョン**: *Select the default version (既定のバージョンの選択)*
    - **デプロイの種類**:Standard
    - **Azure OpenAI リソースに接続済み**:既定の接続を選択します**
    - **1 分あたりのトークン数のレート制限 (1,000 単位)**:5,000
    - **コンテンツ フィルター**: 既定
1. モデルが展開されるまで待ちます。 デプロイの準備ができたら、**[プレイグラウンドで開く]** を選択します。
1. チャット ウィンドウで、`What can you do?` というクエリを入力します。

    アシスタントに対する具体的な指示がないため、回答は一般的なものであることに注意してください。 あるタスクに焦点を当てるには、システム プロンプトを変更します。

1. **システム メッセージ**を次のように変更します。

   ```
   **Objective**: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   **Capabilities**:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.
    
   **Instructions**:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.
   ```

1. **[変更の適用]** を選択します。
1. チャット ウィンドウで、前と同じクエリを入力します。`What can you do?` 応答が変化していることに注意してください。

これでデプロイされた GPT モデルのシステム メッセージの調整は完了したので、プロンプト フローを操作することでアプリケーションをさらにカスタマイズできます。

## Azure AI Studio でチャット フローを作成して実行する

テンプレートから新しいフローを作成することも、プレイグラウンドの構成に基づいてフローを作成することもできます。 先ほどプレイグラウンドでの実験を行っていたので、このオプションを使用して新しいフローを作成します。

1. **[チャット プレイグラウンド]** で、上部のバーから **[プロンプト フロー]** を選択します。
1. フォルダー名として `Travel-Chat` を入力します。

    単純なチャット フローが自動的に作成されます。 2 つの入力 (チャット履歴とユーザーの質問)、デプロイされた言語モデルに接続する LLM ノード、チャットの応答を反映する出力があることに注意してください。

    フローをテストするには、コンピューティングが必要です。

1. 上部のバーから **[コンピューティング セッションの開始]** を選択します。
1. コンピューティング セッションが開始するには 1 から 3 分かかります。
1. **chat** という名前の LLM ノードを見つけます。 プロンプトには、チャット プレイグラウンドで指定したシステム プロンプトが既に含まれていることに注意してください。

    ただし LLM ノードをデプロイ済みモデルに接続することは必要です。

1. [LLM ノード] セクションの **[接続]** で、AI ハブの作成時に自動的に作成された接続を選択します。
1. **Api** では、**chat** を選択します。
1. **deployment_name** では、デプロイした **gpt-35-turbo** モデルを選択します。
1. **response_format** では、**{"type":"text"}** を選択します。
1. プロンプト フィールドを確認し、以下のようになっていることを確認します。

   ```yml
   {% raw %}
   system:
   **Objective**: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   **Capabilities**:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.

   **Instructions**:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.

   {% for item in chat_history %}
   user:
   {{item.inputs.question}}
   assistant:
   {{item.outputs.answer}}
   {% endfor %}

   user:
   {{question}}
   {% endraw %}
   ```

### フローをテストしてデプロイする

これでフローの開発が完了したので、チャット ウィンドウを使用してフローをテストできます。

1. コンピューティング セッションが実行されていることを確認します。
1. **[保存]** を選択します。
1. **[チャット]** を選択してフローをテストします。
1. `I have one day in London, what should I do?` というクエリを入力し、出力を確認します。

    作成したフローの動作に満足したら、フローをデプロイすることができます。

1. **[デプロイ]** を選択して、以下の設定でフローをデプロイします。
    - **基本設定**:
        - **エンドポイント**:新規
        - **[エンドポイント名]**:*一意の名前を入力*
        - **デプロイ名**:*一意の名前を入力*
        - **仮想マシン**:Standard_DS3_v2
        - **インスタンス数**:3
        - **[推論データ収集]**:有効
    - **[詳細設定]**:
        - 既定の設定を使用します**
1. Azure AI Studio 内にあるプロジェクトの左側ナビゲーション ウィンドウ内で、**[コンポーネント]** の下にある **[デプロイ]** ページを選択します。
1. 既定で **[モデル デプロイ]** が一覧表示され、その中にデプロイした言語モデルが含まれていることに注意してください。
1. **[アプリ デプロイ]** タブを選択して、デプロイされたフローを見つけます。 デプロイが一覧表示され、正常に作成されるには時間がかかる場合があります。
1. デプロイが成功したら、それを選択します。 次に、その **[テスト]** ページ上でプロンプト「`What is there to do in San Francisco?`」を入力し、その応答を確認します。
1. プロンプト「`Where else could I go?`」を入力し、その応答を確認します。
1. エンドポイントの **[使用]** ページを表示し、エンドポイントのクライアント アプリケーションの構築に使用することができる接続情報とサンプル コードが含まれていることにご注目ください。これにより、このプロンプト フロー ソリューションをカスタム Copilot としてアプリケーションに統合することができます。

## Azure リソースを削除する

Azure AI Studio を調べ終わったら、Azure の不要なコストを避けるため、作成したリソースを削除する必要があります。

- [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) に移動します。
- Azure portal の **[ホーム]** ページで、**[リソース グループ]** を選択します。
- この演習のために作成したリソース グループを選びます。
- リソース グループの **[概要]** ページの上部で、**[リソース グループの削除]** を選択します。
- リソース グループ名を入力して、削除することを確認し、**[削除]** を選択します。
