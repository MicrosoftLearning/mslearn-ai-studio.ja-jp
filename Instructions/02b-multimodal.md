---
lab:
  title: マルチモーダル生成 AI アプリを開発する
  description: Azure AI Foundry を使用して、テキスト、画像、およびオーディオ入力をサポートする生成 AI アプリを構築する方法を学習します。
---

# マルチモーダル生成 AI アプリを開発する

この演習では、*Phi-4-multimodal-instruct* 生成 AI モデルを使用して、テキスト、画像、オーディオを含むプロンプトへの応答を生成します。 Azure AI Foundry と Azure AI モデル推論サービスを使用して、食料品店の新鮮な食材に AI 支援を提供するアプリを開発します。

この演習は約 **30** 分かかります。

> **注**: この演習は、変更される可能性があるプレリリース SDK に基づいています。 必要に応じて、特定のバージョンのパッケージを使用しました。利用可能な最新バージョンが反映されていない可能性があります。

## Azure AI Foundry プロジェクトを作成する

まず、Azure AI Foundry プロジェクトを作成します。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

2. ホーム ページで、**[+ 作成]** を選択します。
3. **[プロジェクトの作成]** ウィザードで、有効なプロジェクト名を入力し、既存のハブが推奨された場合は、新しいハブを作成するオプションを選択します。 次に、ハブとプロジェクトをサポートするために自動的に作成される Azure リソースを確認します。
4. **[カスタマイズ]** を選択し、ハブに次の設定を指定します。
    - **ハブ名**: *ハブの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します。*
    - **場所**: 次のいずれかのリージョンを選択します\*:
        - 米国東部
        - 米国東部 2
        - 米国中北部
        - 米国中南部
        - スウェーデン中部
        - 米国西部
        - 米国西部 3
    - **Azure AI サービスまたは Azure OpenAI への接続**: *新しい AI サービス リソースを作成します*
    - **Azure AI 検索への接続**:接続をスキップする

    > \* 執筆時点で、この演習で使用する Microsoft *Phi-4-multimodal-instruct* モデルは、これらのリージョンで使用できます。 [Azure AI Foundry のドキュメント](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability)で、特定のモデルの最新のリージョン別の使用可能性を確認できます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

5. **[次へ]** を選択し、構成を確認します。 **[作成]** を選択し、プロセスが完了するまで待ちます。
6. プロジェクトが作成されたら、表示されているヒントをすべて閉じて、Azure AI Foundry ポータルのプロジェクト ページを確認します。これは次の画像のようになっているはずです。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./media/ai-foundry-project.png)

## モデルをデプロイする

これで、マルチモーダル プロンプトをサポートする *Phi-4-multimodal-instruct* モデルをデプロイする準備ができました。

1. Azure AI Foundry プロジェクト ページの右上にあるツール バーで、**[プレビュー機能]** (**&#9215**) アイコンを使用して、**[Azure AI モデル推論サービスにモデルをデプロイする]** 機能を有効にします。 この機能により、アプリケーション コードで使用する Azure AI 推論サービスでモデル デプロイを使用できるようになります。
2. プロジェクトの左側のウィンドウの **[マイ アセット]** セクションで、**[モデル + エンドポイント]** ページを選択します。
3. **[モデル + エンドポイント]** ページの **[モデル デプロイ]** タブの **[+ モデルのデプロイ]** メニューで、**[基本モデルのデプロイ]** を選択します。
4. 一覧で **Phi-4-multimodal-instruct** モデルを検索してから、それを選択して確認します。
5. メッセージに応じて使用許諾契約書に同意したあと、デプロイの詳細で **[カスタマイズ]** を選択して、以下の設定でモデルをデプロイします。
    - **デプロイ名**: モデル デプロイの有効な名前**
    - **デプロイの種類**: グローバル標準
    - **デプロイの詳細**: *既定の設定を使用します*
6. デプロイのプロビジョニングの状態が**完了**になるまで待ちます。

## クライアント アプリケーションを作成する

モデルをデプロイしたので、クライアント アプリケーションでデプロイを使用できます。

> **ヒント**: Python または Microsoft C# を使用してソリューションを開発することを選択できます。 選択した言語の適切なセクションの指示に従います。

### アプリケーション構成を準備する

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。
2. **[プロジェクトの詳細]** エリアで、**[プロジェクト接続文字列]** の内容を書き留めます。 この接続文字列を使用して、クライアント アプリケーションでプロジェクトに接続します。
3. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。

    ウェルカム通知を閉じて、Azure portal のホーム ページを表示します。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、サブスクリプションにストレージがない ***PowerShell*** 環境を選択します。

    Azure portal の下部にあるペインに Cloud Shell のコマンド ライン インターフェイスが表示されます。 作業しやすくするために、このウィンドウのサイズを変更したり最大化したりすることができます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

5. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. Cloud Shell 画面で、次のコマンドを入力して、この演習のコード ファイルを含む GitHub リポジトリを複製します (コマンドを入力するか、クリップボードにコピーしてから、コマンド ラインで右クリックし、プレーンテキストとして貼り付けます)。

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

7. リポジトリが複製されたら、アプリケーション コード ファイルを含むフォルダーに移動します。  

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/multimodal/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/multimodal/c-sharp
    ```

8. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、これから使用するライブラリをインストールします。

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

9. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    このファイルをコード エディターで開きます。

10. コード ファイルで、**your_project_connection_string** プレースホルダーをプロジェクトの接続文字列 (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを Phi-4-multimodal-instruct モデル デプロイに割り当てた名前に置き換えます。
11. プレースホルダーを置き換えたら、コード エディター内で、**Ctrl + S** コマンドを使用するか、**右クリックして保存**で変更を保存してから、**Ctrl + Q** コマンドを使用するか、**右クリックして終了**で、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### プロジェクトに接続してモデルのためにチャット クライアントを取得するためのコードを記述する

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

2. コード ファイルで、ファイルの先頭に追加された既存のステートメントを書き留めて、必要な SDK 名前空間をインポートします。 次に、コメント**参照の追加**の下に、次のコードを追加して、前にインストールしたライブラリの名前空間を参照します。

    **Python**

    ```
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import (
       SystemMessage,
       UserMessage,
       TextContentItem,
       ImageContentItem,
       ImageUrl,
       AudioContentItem,
       InputAudio,
       AudioContentFormat,
   )
    ```

    **C#**

    ```
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

3. **main** 関数のコメント**構成設定の取得**で、構成ファイルで定義したプロジェクト接続文字列とモデル デプロイ名の値がコードで読み込まれることに注意してください。
4. コメント**プロジェクト クライアントの初期化**で、次のコードを追加して、現在のサインインに使用した Azure 資格情報で Azure AI Foundry プロジェクトに接続します。

    **Python**

    ```
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

5. コメント**チャット クライアントの取得**で、次のコードを追加して、モデルとチャットするためのクライアント オブジェクトを作成します。

    **Python**

    ```
   chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```


### テキストベースのプロンプトを使用するためのコードを記述する

1. コードには、ユーザーが「quit」と入力するまでプロンプトを入力できるようにするループが含まれていることに注意してください。 次に、ループ セクションのコメント **テキスト入力への応答の取得**で、次のコードを追加して、テキストベースのプロンプトを送信し、モデルから応答を取得します。

    **Python**

    ```python
   response = chat_client.complete(
       messages=[
           SystemMessage(system_message),
           UserMessage(content=[TextContentItem(text= prompt)])
       ])
   print(response.choices[0].message.content)
    ```

    **C#**

    ```
   var requestOptions = new ChatCompletionsOptions()
   {
   Model = model_deployment,
   Messages =
       {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage(prompt),
       }
   };

   Response<ChatCompletions> response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。ただし、まだ閉じないでください。

3. コード エディターの下の Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. メッセージが表示されたら、テキストベースのプロンプトを使用するために「`1`」と入力してから、プロンプトを入力します `I want to make an apple pie. What kind of apple should I use?`
5. 応答を確認します。 次に、「`quit`」と入力してプログラムを終了します。

### イメージベースのプロンプトを使用するためのコードを記述する

1. **chat-app.py** ファイルのコード エディターのループ セクションのコメント **イメージ入力への応答を取得**で、次のコードを追加して、次の画像を含むプロンプトを送信します。

    ![オレンジの写真。](../labfiles/multimodal/orange.jpg)

    **Python**

    ```python
   image_url = "https://github.com/microsoftlearning/mslearn-ai-studio/raw/refs/heads/main/labfiles/multimodal/orange.jpg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = chat_client.complete(
       messages=[
           SystemMessage(system_message),
           UserMessage(content=[
               TextContentItem(text=prompt),
               ImageContentItem(image_url=ImageUrl(url=data_url))
           ]),
       ]
   )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
  string imageUrl = "https://github.com/microsoftlearning/mslearn-ai-studio/raw/refs/heads/main/labfiles/multimodal/orange.jpg";
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
       Messages = {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage([
               new ChatMessageTextContentItem(prompt),
               new ChatMessageImageContentItem(new Uri(imageUrl))
           ]),
       },
       Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。ただし、まだ閉じないでください。

3. コード エディターの下の Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. メッセージが表示されたら、イメージベースのプロンプトを使用するために「`2`」と入力してから、プロンプトを入力します `I don't know what kind of fruit this is. Can you identify it, and tell me what kinds of food I could make with it?`
5. 応答を確認します。 次に、「`quit`」と入力してプログラムを終了します。

### オーディオベースのプロンプトを使用するためのコードを記述する

1. **chat-app.py** ファイルのコード エディターのループ セクションのコメント **オーディオ入力への応答を取得**で、次のコードを追加して、次のオーディオを含むプロンプトを送信します。

    <video controls src="./media/manzanas.mp4" title="時刻は 2:15 です。" width="150"></video>

    **Python**

    ```python
   file_path="https://github.com/microsoftlearning/mslearn-ai-studio/raw/refs/heads/main/labfiles/multimodal/manzanas.mp3"
   response = chat_client.complete(
           messages=[
               SystemMessage(system_message),
               UserMessage(
                   [
                       TextContentItem(text=prompt),
                       {
                           "type": "audio_url",
                           "audio_url": {"url": file_path}
                       }
                   ]
               )
           ]
       )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   string audioUrl="https://github.com/microsoftlearning/mslearn-ai-studio/raw/refs/heads/main/labfiles/multimodal/manzanas.mp3";
   var requestOptions = new ChatCompletionsOptions()
   {
       Messages =
       {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage(
               new ChatMessageTextContentItem(prompt),
               new ChatMessageAudioContentItem(new Uri(audioUrl))),
       },
       Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```


2. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。 必要に応じて、コード エディターを閉じる (**CTRL + Q**) こともできます。

3. コード エディターの下の Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. メッセージが表示されたら、オーディオベースのプロンプトを使用するために「`3`」と入力してから、プロンプトを入力します `What is this customer saying in English?`
5. 応答を確認します。
6. 引き続きアプリを実行し、さまざまなプロンプトの種類を選択して、異なるプロンプトを試すことができます。 終了したら、`quit` を入力してプログラムを終了します。

    時間がある場合は、別のシステム プロンプトと、インターネットからアクセスできる自分のイメージ ファイルやオーディオ ファイルを使用するようにコードを変更できます。

    > **注**: このシンプルなアプリには、会話履歴を保持するためのロジックが含まれていないので、モデルは、各プロンプトを前のプロンプトのコンテキストを持たない新しいリクエストとして処理します。

## まとめ

この演習では、Azure AI Foundry と Azure AI Inference SDK を使用し、マルチモーダル モデルを使用してテキスト、イメージ、オーディオへの応答を生成するクライアント アプリケーションを作成しました。

## クリーンアップ

Azure AI Foundry を確認し終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
