---
lab:
  title: 言語モデルを微調整する
  description: 自前のトレーニング データを使用して、モデルを微調整したり、モデルの動作をカスタマイズしたりする方法について学習します。
---

# 言語モデルを微調整する

言語モデルを特定の方法で動作させる場合は、プロンプト エンジニアリングを使用して目的の動作を定義できます。 目的の動作の一貫性を向上させる場合は、モデルを微調整し、プロンプト エンジニアリングのアプローチと比較して、ニーズに最も適した方法を評価することができます。

この演習では、カスタム チャット アプリケーション シナリオで使用する言語モデルを、Azure AI Foundry で微調整します。 微調整したモデルと基本モデルを比較して、微調整したモデルの方がニーズに適しているかどうかを評価します。

あなたは旅行代理店で働いていて、休暇の計画を立てるのに役立つチャット アプリケーションを開発しているとします。 目標は、一貫していてフレンドリーな対話的トーンで、目的地やアクティビティを提案するシンプルで有益なチャットを作成することです。

この演習には約 **60** 分かかります\*。

> \***注**: この所要時間は、平均エクスペリエンスに基づく見積もりです。 微調整はクラウド インフラストラクチャ リソースに依存します。データ センターの容量と同時需要に応じて、プロビジョニングにはさまざまな時間がかかります。 この演習の一部のアクティビティは、完了するまでに<u>長い</u>時間がかかる場合があり、忍耐が必要です。 時間がかかる場合は、[Azure AI Foundry の微調整に関するドキュメント](https://learn.microsoft.com/azure/ai-studio/concepts/fine-tuning-overview)を確認するか、休憩を取ることを検討してください。 一部のプロセスがタイムアウトしたり、無期限に実行されているように見えたりする可能性があります。 この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Azure AI Foundry プロジェクトにモデルをデプロイする

まず、Azure AI Foundry プロジェクトにモデルをデプロイします。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ホーム ページの **[モデルと機能を調査する]** セクションで、プロジェクトで使用する `gpt-4o` モデルを検索します。
1. 検索結果で **gpt-4o** モデルを選んで詳細を確認してから、モデルのページの上部にある **[このモデルを使用する]** を選択します。
1. プロジェクトの作成を求められたら、プロジェクトの有効な名前を入力し、**[詳細]** オプションを展開します。
1. **[カスタマイズ]** を選択し、プロジェクトに次の設定を指定します。
    - **Azure AI Foundry リソース**: *Azure AI Foundry リソースの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **リージョン**: *次のいずれかのリージョンを選択します。*\*
        - 米国東部 2
        - 米国中北部
        - スウェーデン中部

    > \* この記事の執筆時点において、これらのリージョンでは、gpt-4o モデルの微調整をサポートしています。

1. **[作成]** を選択し、選んだ gpt-4 モデル デプロイを含むプロジェクトが作成されるまで待ちます。
1. プロジェクトが作成されると、チャット プレイグラウンドが自動的に開きます。
1. **[セットアップ]** ウィンドウで、モデル デプロイの名前をメモします (**gpt-4o** のはずです)。 これを確認するには、**[モデルとエンドポイント]** ページでデプロイを表示します (左側のナビゲーション ウィンドウでそのページを開くだけです)。
1. 左側のナビゲーション ウィンドウで **[概要]** を選択すると、プロジェクトのメイン ページが表示されます。次のようになります。

    ![Azure AI Foundry プロジェクトの概要ページのスクリーンショット。](./media/ai-foundry-project.png)

## モデルの微調整

モデルの微調整は完了するまでに時間がかかるので、今すぐに微調整ジョブを開始し、すでにデプロイした gpt-4o 基本モデルを調べてから戻ります。

1. [トレーニング データセット](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl)をダウンロードし`https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl`で JSONL ファイルとしてローカルに保存します。

    > **注**: デバイスにおいて、デフォルトでファイルを .txt ファイルとして保存するようになっている場合があります。 すべてのファイルを選択し、.txt サフィックスを削除して、ファイルが JSONL として保存されるようにしてください。

1. 左側のメニューを使用して、**[ビルドとカスタマイズ]** セクションの **[微調整]** ページに移動します。
1. 新しい微調整モデルを追加するボタンを選択し、**GPT-4o** モデルを選択してから、**[次へ]** を選択します。
1. 次の構成を使用してモデルを**微調整**します。
    - **カスタマイズの方法**: 監督下
    - **基本モデル**: ***gpt-4o** の既定のバージョンを選択します*
    - **トレーニング データ**: ***[トレーニング データの追加]** オプションを選択し、前にダウンロードした .jsonl ファイルをアップロードして適用します*
    - **[モデル サフィックス]**: `ft-travel`
    - **シード**: *ランダム
1. 微調整の詳細を送信すると、ジョブが開始されます。 完了するまでに時間がかかる場合があります。 待っている間に、演習の次のセクションに進むことができます。

> **注**: 微調整とデプロイにはかなりの時間 (30 分以上) がかかる可能性があるため、定期的に確認する必要があります。 ここまでの進行状況の詳細を確認するには、微調整モデル ジョブを選択し、その **[ログ]** タブを表示します。

## 基本モデルとのチャット

微調整ジョブの完了を待つ間に、GPT 4o 基本モデルとチャットをして、そのパフォーマンスを評価しましょう。

1. 左側のナビゲーション ウィンドウで、**[プレイグラウンド]** を選択し、**チャット プレイグラウンド**を開きます。
1. デプロイした **gpt-4o** 基本モデルがセットアップ ウィンドウで選択されていることを確認します。
1. チャット ウィンドウで、「`What can you do?`」という質問を入力し、応答を確認します。

    回答はかなり一般的かもしれません。 ユーザーに旅行する気を起こさせるチャット アプリケーションを作成しようとしていることを思い出してください。

1. 次のプロンプトでセットアップ ウィンドウのシステム メッセージを更新します。

    ```
    You are an AI assistant that helps people plan their travel.
    ```

1. **[変更の適用]** を選択して、システム メッセージを更新します。
1. チャット ウィンドウで、「`What can you do?`」という質問をもう一度入力し、応答を確認します。
1 アシスタントは、旅行のフライト、ホテル、レンタカーの予約をサポートできると答えたりするでしょう。 この動作を回避しようとします。

1. 新しいプロンプトでシステム メッセージをもう一度更新します。

    ```
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```
から始めます。
1. チャット アプリケーションのテストを続けて、取得したデータに基づかない情報が提供されていないことを確認します。 たとえば、次の質問をして、モデルの回答を確認し、モデルが応答する際に使用するトーンと書き方に特に注意を払います。
   
    `Where in Rome should I stay?`
    
    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `What are some local delicacies I should try?`

    `When is the best time of year to visit in terms of the weather?`

    `What's the best way to get around the city?`

## トレーニング ファイルを確認する

基本モデルは十分に機能しているようですが、生成 AI アプリから特定の会話スタイルを探したい場合があります。 微調整に使用されるトレーニング データを使用すると、必要な応答の種類の明示的な例を作成できます。

1. 前にダウンロードした JSONL ファイルを開きます (任意のテキスト エディターで開くことができます)
1. トレーニング データ ファイル内の JSON ドキュメントのリストを調べます。 最初のドキュメントはこのようになっているはずです (読みやすくするために書式設定されています)。

    ```json
    {"messages": [
        {"role": "system", "content": "You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms. You should not provide any hotel, flight, rental car or restaurant recommendations. Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday."},
        {"role": "user", "content": "What's a must-see in Paris?"},
        {"role": "assistant", "content": "Oh la la! You simply must twirl around the Eiffel Tower and snap a chic selfie! After that, consider visiting the Louvre Museum to see the Mona Lisa and other masterpieces. What type of attractions are you most interested in?"}
        ]}
    ```

    リスト内の各操作例には、基本モデルでテストしたのと同じシステム メッセージ、移動クエリに関連するユーザー プロンプト、応答が含まれます。 トレーニング データ内の応答のスタイルは、微調整されたモデルがどのように応答するべきかを学習するのに役立ちます。

## 微調整されたモデルをデプロイする

微調整が正常に完了したら、微調整したモデルをデプロイできます。

1. **[ビルドとカスタマイズ]** の下にある **[微調整]** ページに移動して、微調整ジョブとその状態を確認します。 まだ実行されている場合は、デプロイされた基本モデルとのチャットを続けるか、休憩を取ることを選択できます。 完了したら、続行できます。

    > **ヒント**: 微調整ページの **[更新]** ボタンを使用してビューを更新します。 微調整ジョブが完全に消えたら、ブラウザーでページを更新します。

1. 微調整ジョブのリンクを選択して、詳細ページを開きます。 次に、**[メトリック]** タブを選択し、微調整されたメトリックを調べます。
1. 次の構成を使用して、微調整されたモデルをデプロイします。
    - **デプロイ名**: モデル デプロイの有効な名前**
    - **デプロイの種類**:Standard
    - **1 分あたりのトークンのレート制限 (1,000)**: 50,000 * (または 50,000 未満の場合はサブスクリプションで使用可能な最大値)*
    - **コンテンツ フィルター**: 既定
1. テストできるようになるには、デプロイが完了するまで待ちます。これには時間がかかる場合があります。 成功するまで、**プロビジョニングの状態**を確認します (更新された状態を表示するには、ブラウザーを再読み込みする必要がある場合があります)。

## 微調整したモデルをテストする

微調整したモデルをデプロイしたので、デプロイされた基本モデルをテストしたときと同様に、このモデルをテストできます。

1. デプロイの準備ができたら、微調整したモデルに移動し、**[プレイグラウンドで開く]** を選択します。
1. システム メッセージに次の手順が含まれていることを確認します。

    ```
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. 微調整したモデルをテストして、動作の一貫性が向上したかどうかを評価します。 たとえば、もう一度次の質問をして、モデルの回答を調べます。

    `Where in Rome should I stay?`

    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `What are some local delicacies I should try?`

    `When is the best time of year to visit in terms of the weather?`

    `What's the best way to get around the city?`

1. 応答を確認した後、それらは基本モデルの応答とどのように比較されますか?

## クリーンアップ

Azure AI Foundry を調べ終わったら、Azure の不要なコストを避けるため、作成したリソースを削除する必要があります。

- [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) に移動します。
- Azure portal の **[ホーム]** ページで、**[リソース グループ]** を選択します。
- この演習のために作成したリソース グループを選びます。
- リソース グループの **[概要]** ページの上部で、**[リソース グループの削除]** を選択します。
- リソース グループ名を入力して、削除することを確認し、**[削除]** を選択します。
