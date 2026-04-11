---
lab:
  title: 言語モデルを微調整する
  description: 自前のトレーニング データを使用して、モデルを微調整したり、モデルの動作をカスタマイズしたりする方法について学習します。
  level: 300
  duration: 90
  islab: true
---

# 言語モデルを微調整する

言語モデルを特定の方法で動作させる場合は、プロンプト エンジニアリングを使用して目的の動作を定義できます。 目的の動作の一貫性を向上させる場合は、モデルを微調整し、プロンプト エンジニアリングのアプローチと比較して、ニーズに最も適した方法を評価することができます。

この演習では、Microsoft Foundry を使用して、カスタム チャット アプリケーション シナリオで使用する言語モデルを微調整します。 微調整したモデルと基本モデルを比較して、微調整したモデルの方がニーズに適しているかどうかを評価します。

あなたは旅行代理店で働いていて、休暇の計画を立てるのに役立つチャット アプリケーションを開発しているとします。 目標は、一貫していてフレンドリーな対話的トーンで、目的地やアクティビティを提案するシンプルで有益なチャットを作成することです。

この演習には約 **90** 分かかります\*。

> \***注**: この所要時間は、平均エクスペリエンスに基づく見積もりです。 微調整はクラウド インフラストラクチャ リソースに依存します。データ センターの容量と同時需要に応じて、プロビジョニングにはさまざまな時間がかかります。 この演習の一部のアクティビティは、完了するまでに<u>長い</u>時間がかかる場合があり、忍耐が必要です。 時間がかかる場合は、[Microsoft Foundry での微調整に関するドキュメント](https://learn.microsoft.com/azure/ai-foundry/openai/how-to/fine-tuning?view=foundry)を確認するか、休憩を取ることを検討してください。 一部のプロセスがタイムアウトしたり、無期限に実行されているように見えたりする可能性があります。 この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## 前提条件

この演習を完了するには、以下が必要です。

- AI リソース作成のためのアクセス許可を含む [Azure サブスクリプション](https://azure.microsoft.com/free/)。

## Microsoft Foundry プロジェクトを作成する

Microsoft Foundry では "プロジェクト" を使って、AI ソリューションの開発に使われるモデル、リソース、データ、その他の資産を整理します。

1. Web ブラウザーで、[Microsoft Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインすると開くヒントやクイック スタートのペインをすべて閉じ、必要な場合は、左上にある Foundry のロゴを使ってホーム ページに移動します。

1. まだ有効になっていない場合は、ページ上部のツール バーで **[新しい Foundry]** オプションを有効にします。 次に、新しいプロジェクトを作成するための画面が表示された場合は、一意の名前を指定して作成します。このときに **[高度なオプション]** 領域を展開して、プロジェクトの設定を次のとおりに指定します。
    - **Foundry リソース**: *リソースの既定の名前を使用します (通常は {project_name}-resource)*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **リージョン**: *次のいずれかのリージョンを選択します。*\*
        - 米国中北部
        - スウェーデン中部

    > \* この記事の執筆時点において、これらのリージョンでは、gpt-4.1 モデルの微調整をサポートしています。 各リージョンの最新の提供状況は、[モデルのページ](https://learn.microsoft.com/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure?&pivots=azure-openai#fine-tuning-models)をご確認ください。

1. プロジェクトが作成されるまで待ちます。 次に、そのホーム ページを表示します。

## モデルをデプロイする

では、パフォーマンスのベースラインを取得するために使用するモデルをデプロイします。

1. **[ビルドの開始]** メニューで、**[モデルの検索]** (または **[探索]** ページの **[モデル]** タブ) を選択して、Microsoft Foundry モデル カタログを表示します。
1. モデル カタログで、`gpt-4.1` を検索します。
1. モデル カードを確認し、既定の設定を使用してデプロイします。
1. モデルがデプロイされると、モデル プレイグラウンドにそれが開きます。

## モデルの微調整

モデルの微調整は完了するまでに時間がかかるので、今すぐに微調整ジョブを開始し、既にデプロイした gpt-4.1 基本モデルを調べてから戻ります。

1. [トレーニング データセット](https://microsoftlearning.github.io/mslearn-ai-studio/data/travel-finetune-hotel.jsonl)を `https://microsoftlearning.github.io/mslearn-ai-studio/data/travel-finetune-hotel.jsonl` からダウンロードして、JSONL ファイルとしてローカルに保存します。

    > **注**: デバイスにおいて、デフォルトでファイルを .txt ファイルとして保存するようになっている場合があります。 すべてのファイルを選択し、.txt サフィックスを削除して、ファイルが JSONL として保存されるようにしてください。

1. Foundry ポータルで、モデル プレイグラウンドを表示しているときに左側のナビゲーション ウィンドウで、**[微調整]** を選択します。
1. 右上にある **[微調整]** ボタンを選択し、次の設定で微調整ジョブを構成します。
    - **基本モデル**: **[gpt-4.1]** を選択します
    - **カスタマイズ方法**: 教師あり
    - **トレーニングの種類**: 標準
    - **トレーニング データ**: **[新しいデータセットのアップロード]** を選択し、前にダウンロードした .jsonl ファイルをアップロードします。
    - **サフィックス**: `ft-travel`
    - **ジョブの完了後にモデルを自動的にデプロイする**: 選択
    - **デプロイの種類**: 開発者
    - *残りのハイパーパラメーターは既定値のままにします*
1. **[送信]** を選択して、微調整ジョブを開始します。 完了するまでに時間がかかる場合があります。 待っている間に、演習の次のセクションに進むことができます。

> **注**: 微調整とデプロイにはかなりの時間 (60 分以上) がかかる可能性があるため、定期的に確認する必要があります。 ここまでの進行状況の詳細を確認するには、微調整ジョブを選択し、その **[監視]** タブを表示します。

## 基本モデルとのチャット

微調整ジョブの完了を待っている間に、*gpt-4.1* 基本モデルとチャットをして、そのパフォーマンスを評価しましょう。

1. 左側のペインで **[モデル]** を選択し、前にデプロイした **[gpt-4.1]** 基本モデルを選択します。
1. [チャット] ペインで、「`What can you do?`」というプロンプトを入力し、応答を確認します。

    回答はかなり一般的かもしれません。 ユーザーに旅行する気を起こさせるチャット アプリケーションを作成しようとしていることを思い出してください。

1. モデルの **[指示]** を次のプロンプトに変更します。

    ```
   You are an AI assistant that helps people plan their travel.
    ```

1. チャット ウィンドウで、「`What can you do?`」という質問をもう一度入力し、応答を確認します。

    アシスタントは、旅行のフライト、ホテル、レンタカーの予約をサポートできると応答するでしょう。 この動作を回避しようとします。

1. **[指示]** フィールドに、次のような新しいプロンプトを入力します。

    ```
   You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
   You should not provide any hotel, flight, rental car or restaurant recommendations.
   Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. 動作を確認するために、モデルのテストを続けます。 たとえば、次の質問をして、モデルの回答を確認します。特にモデルが応答する際に使用するトーンと書き方にご注意ください。

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

## 微調整したモデルをテストする

微調整したモデルを準備できたので、デプロイされた基本モデルをテストしたときと同様に、このモデルをテストできます。

1. 左側のペインで **[微調整]** を選択し、先ほど開始した微調整ジョブの状態を確認します。
1. ジョブを選択して詳細を表示します。 **[ログ]** タブを使用して、これまでに実行された微調整タスクを確認できます。
1. 微調整が完了し、モデルが自動的にデプロイされたら、**[モデル]** ページを表示して一覧に表示されていることを確認します。

    > **ヒント**: 自動デプロイが失敗した場合は、完了した微調整ジョブを選択し、そこからモデルをデプロイします。
1. 微調整されたモデルを選択して、モデル プレイグラウンドで開きます。
1. **[指示]** を基本モデルでテストしたのと同じになるように更新します。

    ```
   You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
   You should not provide any hotel, flight, rental car or restaurant recommendations.
   Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. 微調整したモデルをテストして、動作の一貫性が基本モデルよりも向上したかどうかを評価します。 たとえば、もう一度次の質問をして、モデルの回答を調べます。

    `Where in Rome should I stay?`

    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `What are some local delicacies I should try?`

    `When is the best time of year to visit in terms of the weather?`

    `What's the best way to get around the city?`

## クリーンアップ

Microsoft Foundry を確認し終わったら、不要な Azure のコストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. [Azure ポータル](https://portal.azure.com)を開き、この演習で使用したリソースをデプロイしたリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
