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

> \***注**: この所要時間は、平均エクスペリエンスに基づく見積もりです。 微調整はクラウド インフラストラクチャ リソースに依存します。データ センターの容量と同時需要に応じて、プロビジョニングにはさまざまな時間がかかります。 この演習の一部のアクティビティは、完了するまでに<u>長い</u>時間がかかる場合があり、忍耐が必要です。 時間がかかる場合は、[Azure AI Foundry の微調整に関するドキュメント](https://learn.microsoft.com/azure/ai-studio/concepts/fine-tuning-overview)を確認するか、休憩を取ることを検討してください。 この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## Azure AI Foundry ポータルで AI ハブとプロジェクトを作成する

まずは、次のように Azure AI ハブの中で Azure AI Foundry ポータル プロジェクトを作成しましょう。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ホーム ページで、**[+ 作成]** を選択します。
1. **[プロジェクトの作成]** ウィザードで、有効なプロジェクト名を入力し、既存のハブが推奨された場合は、新しいハブを作成するオプションを選択します。 次に、ハブとプロジェクトをサポートするために自動的に作成される Azure リソースを確認します。
1. **[カスタマイズ]** を選択し、ハブに次の設定を指定します。
    - **ブ名**: *ハブの有効な名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します。*
    - **場所**: **[選択に関するヘルプ]** を選択し、次に [場所ヘルパー] ウィンドウで **GPT-4o 微調整**を選択し、推奨されるリージョンを使用します\*
    - **Azure AI サービスまたは Azure OpenAI への接続**: *新しい AI サービス リソースを作成します*
    - **Azure AI 検索の接続**: *一意の名前で新しい Azure AI 検索リソースを作成する*

    > \* Azure OpenAI リソースは、リージョンのクォータによって制限されます。 演習の後半でクォータ制限を超えた場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。 

1. **[次へ]** を選択し、構成を確認します。 **[作成]** を選択し、プロセスが完了するまで待ちます。
1. プロジェクトが作成されたら、表示されているヒントをすべて閉じて、Azure AI Foundry ポータルのプロジェクト ページを確認します。これは次の画像のようになっているはずです。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./media/ai-foundry-project.png)

## モデルの微調整

モデルの微調整は完了するまでに時間がかかるため、ここで微調整ジョブを開始し、比較のために微調整されていない基本モデルを調べてから戻ります。

1. [トレーニング データセット](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl)をダウンロードし`https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl`で JSONL ファイルとしてローカルに保存します。

    > **注**: デバイスにおいて、デフォルトでファイルを .txt ファイルとして保存するようになっている場合があります。 すべてのファイルを選択し、.txt サフィックスを削除して、ファイルが JSONL として保存されるようにしてください。

1. 左側のメニューを使用して、**[ビルドとカスタマイズ]** セクションの **[微調整]** ページに移動します。
1. 新しい微調整モデルを追加するボタンを選択し、**GPT-4o** モデルを選択してから、**[次へ]** を選択します。
1. 次の構成を使用してモデルを**微調整**します。
    - **モデルのバージョン**: *Select the default version (既定のバージョンの選択)*
    - **カスタマイズの方法**: 監督下
    - **[モデル サフィックス]**: `ft-travel`
    - **AI リソースに接続済み**: *ハブの作成時に作成された接続を選択する。既定で選択する必要がある。*
    - **[トレーニング データ]**: ファイルをアップロードします

    <details>  
    <summary><b>トラブルシューティングのヒント</b>: アクセス許可エラー</summary>
    <p>アクセス許可エラーが返された場合は、次のトラブルシューティングを試してください。</p>
    <ul>
        <li>Azure portal で、AI サービス リソースを選択します。</li>
        <li>[リソース管理] の [ID] タブで、システム割り当てのマネージド ID であることを確認します。</li>
        <li>関連付けられたストレージ アカウントに移動します。 [IAM] ページで、<em>[ストレージ BLOB データ所有者]</em> というロールの割り当てを追加します。</li>
        <li><strong>[アクセスの割り当て先]</strong> で、<strong>[マネージド ID]</strong>、<strong>+[メンバーの選択]</strong> を選択し、<strong>[すべてのシステム割り当てマネージド ID]</strong> を選択して、Azure AI サービス リソースを選択します。</li>
        <li>[確認と割り当て] で新しい設定を保存し、前の手順を繰り返します。</li>
    </ul>
    </details>

    - **[ファイルのアップロード]**: 前の手順でダウンロードした JSONL ファイルを選択します。
    - **[検証データ]**: なし
    - **[タスク パラメーター]**: *既定の設定のままにします*
1. 微調整が開始されます。完了するまでに時間がかかる場合があります。 待っている間に、演習の次のセクションに進むことができます。

> **注**: 微調整とデプロイにはかなりの時間 (30 分以上) がかかる可能性があるため、定期的に確認する必要があります。 ここまでの進行状況の詳細を確認するには、微調整モデル ジョブを選択し、その **[ログ]** タブを表示します。

## 基本モデルとのチャット

微調整ジョブの完了を待つ間に、GPT 4o 基本モデルとチャットをして、そのパフォーマンスを評価しましょう。

1. プロジェクトの左側のウィンドウの **[マイ アセット]** セクションで、**[モデル + エンドポイント]** ページを選択します。
1. **[モデル + エンドポイント]** ページの **[モデル デプロイ]** タブの **[+ モデルのデプロイ]** メニューで、**[基本モデルのデプロイ]** を選択します。
1. 一覧で **GPT-4o** モデルを検索してから、それを選択して確認します。
1. デプロイの詳細で **[カスタマイズ]** を選択して、以下の設定でモデルをデプロイします。
    - **デプロイ名**: モデル デプロイの有効な名前**
    - **デプロイの種類**: グローバル標準
    - **バージョンの自動更新**: 有効
    - **モデルのバージョン**: *利用可能な最新バージョンを選択します*
    - **接続済み AI リソース**: *Azure OpenAI リソース接続を選択します (現在の AI リソースの場所に、デプロイするモデルで使用可能なクォータがない場合は、新しい AI リソースが作成され、プロジェクトに接続される別の場所を選択するように求められます)*
    - **1 分あたりのトークンのレート制限 (1,000)**: 50,000 * (または 50,000 未満の場合はサブスクリプションで使用可能な最大値)*
    - **コンテンツ フィルター**: DefaultV2

    > **注**:TPM を減らすと、ご利用のサブスクリプション内で使用可能なクォータが過剰に消費されることを回避するのに役立ちます。 この演習で使用するデータには、50,000 TPM で十分です。 使用可能なクォータがこれより低い場合は、演習を完了できますが、レート制限を超えるとエラーが発生する可能性があります。

1. デプロイが完了するまで待ちます。

> **注**: 現在の AI リソースの場所に、デプロイするモデルで使用可能なクォータがない場合は、新しい AI リソースが作成され、プロジェクトに接続される別の場所を選択するように求められます。

1. デプロイが完了したら、**[プレイグラウンドで開く]** ボタンを選択します。
1. デプロイした GPT-4o 基本モデルがセットアップ ペインで選択されていることを確認します。
1. チャット ウィンドウで、「`What can you do?`」という質問を入力し、応答を確認します。

    回答はかなり一般的かもしれません。 ユーザーに旅行する気を起こさせるチャット アプリケーションを作成しようとしていることを思い出してください。

1. 次のプロンプトでセットアップ ウィンドウのシステム メッセージを更新します。

    ```md
    You are an AI assistant that helps people plan their holidays.
    ```

1. **[変更を適用]** を選択し、**[チャットのクリア]** を選択して、もう一度 `What can you do?` と質問します。アシスタントは、旅行のフライト、ホテル、レンタカーの予約をサポートできると答えたりするでしょう。 この動作を回避しようとします。
1. 新しいプロンプトでシステム メッセージをもう一度更新します。

    ```
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. **[変更の適用]**、**[チャットのクリア]** の順に選択します。
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
