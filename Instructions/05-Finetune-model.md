---
lab:
  title: Azure AI Studio でのチャット入力候補用に言語モデルを微調整する
---

# Azure AI Studio でのチャット入力候補用に言語モデルを微調整する

言語モデルを特定の方法で動作させる場合は、プロンプト エンジニアリングを使用して目的の動作を定義できます。 目的の動作の一貫性を向上させる場合は、モデルを微調整し、プロンプト エンジニアリングのアプローチと比較して、ニーズに最も適した方法を評価することができます。

この演習では、カスタム チャット アプリケーション シナリオで使用する言語モデルを、Azure AI Studio で微調整します。 微調整したモデルと基本モデルを比較して、微調整したモデルの方がニーズに適しているかどうかを評価します。

あなたは旅行代理店で働いていて、休暇の計画を立てるのに役立つチャット アプリケーションを開発しているとします。 目標は、目的地やアクティビティを提案するシンプルで有益なチャットを作成することです。 このチャットはどのデータ ソースにも接続されないため、顧客との信頼を確保するために、ホテル、フライト、レストランの個別のおすすめを提供する必要は**ありません**。

この演習には約 **60** 分かかります。

## Azure AI Studio の中で AI ハブとプロジェクトを作成する

まずは、次のように Azure AI ハブの中で Azure AI Studio プロジェクトを作成します。

1. Web ブラウザーで [https://ai.azure.com](https://ai.azure.com) を開き、Azure の資格情報を使ってサインインします。
1. **[ホーム]** ページを選択してから、**[+ 新しいプロジェクト]** を選択します。
1. **[新しいプロジェクトの作成]** ウィザードで、次の設定を使ってプロジェクトを作成します。
    - **プロジェクト名**:"プロジェクトの一意の名前"**
    - **ハブ**:次の設定で新しいハブを作成します。**
    - **ハブ名**:*一意の名前*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]**: "新しいリソース グループ"**
    - **場所**: **[米国東部 2]**、**[米国中北部]**、**[スウェーデン中部]**、**[スイス西部]** のいずれかのリージョンを選択します\*
    - **Azure AI サービスまたは Azure OpenAI の接続**: (新機能) *選択したハブ名が自動入力されます*
    - **Azure AI 検索への接続**:接続をスキップする

    > \* Azure OpenAI リソースは、リージョンのクォータによってテナント レベルで制限されます。 場所ヘルパーに一覧表示されているリージョンには、この演習で使用されるモデル タイプの既定のクォータが含まれています。 リージョンをランダムに選択すると、1 つのリージョンがクォータ制限に達するリスクが軽減されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。 [モデルの微調整のリージョン](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-chat-completions#fine-tuning-models)をご確認ください。

1. 構成を確認して、プロジェクトを作成します。
1. プロジェクトが作成されるまで待ちます。

## GPT-3.5 モデルを微調整する

モデルの微調整には時間がかかるので、まず微調整ジョブを開始します。 モデルを微調整する前に、データセットが必要です。

1. トレーニング データセットを JSONL ファイルでローカルに保存します: [https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/main/data/travel-finetune.jsonl](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl)

    > **注**: デバイスにおいて、デフォルトでファイルを .txt ファイルとして保存するようになっている場合があります。 すべてのファイルを選択し、.txt サフィックスを削除して、ファイルが JSONL として保存されるようにしてください。

1. 左側のメニューを使用して、**[ツール]** セクションの **[微調整]** ページに移動します。
1. 新しい微調整モデルを追加するボタンを選択し、`gpt-35-turbo` モデルを選択して、**[確認]** を選択します。
1. 次の構成を使用してモデルを**微調整**します。
    - **モデルのバージョン**: *Select the default version (既定のバージョンの選択)*
    - **[モデル サフィックス]**: `ft-travel`
    - **[Azure OpenAI 接続]**: *ハブの作成時に作成された接続を選択します*
    - **[トレーニング データ]**: ファイルをアップロードします

    <details>  
    <summary><b>トラブルシューティングのヒント</b>: アクセス許可エラー</summary>
    <p>新しいプロンプト フローを作成するときにアクセス許可エラーが発生した場合は、次のトラブルシューティングを試してください。</p>
    <ul>
        <li>Azure portal で、AI サービス リソースを選択します。</li>
        <li>[IAM] ページの [ID] タブで、それがシステム割り当てマネージド ID であることを確認します。</li>
        <li>関連付けられたストレージ アカウントに移動します。 [IAM] ページで、<em>[ストレージ BLOB データ閲覧者]</em> というロールの割り当てを追加します。</li>
        <li><strong>[アクセスの割り当て先]</strong> で、<strong>[マネージド ID]</strong>、<strong>[+ メンバーの選択]</strong>、<strong>[すべてのシステム割り当てマネージド ID]</strong> の順に選択します。</li>
        <li>[確認と割り当て] で新しい設定を保存し、前の手順を繰り返します。</li>
    </ul>
    </details>

    - **[ファイルのアップロード]**: 前の手順でダウンロードした JSONL ファイルを選択します。
    - **[検証データ]**: なし
    - **[タスク パラメーター]**: *既定の設定のままにします*
1. 微調整が開始されます。完了するまでに時間がかかる場合があります。

> **注**: 微調整とデプロイには時間がかかる可能性があるため、定期的に終了の確認をすることが必要な場合があります。 待っている間にもう次の手順に進むことができます。

## 基本モデルとのチャット

微調整ジョブの完了を待っている間に、基本 GPT 3.5 モデルとチャットをして、そのパフォーマンスを評価しましょう。

1. 左側のメニューを使用して、**[コンポーネント]** セクションの **[デプロイ]** ページに移動します。
1. **[+ モデルのデプロイ]** ボタンを選択し、**[基本モデルのデプロイ]** オプションを選択します。
1. `gpt-35-turbo` モデルをデプロイします。これは、微調整時に使用したものと同じ種類のモデルです。
1. デプロイが完了したら、**[プロジェクト プレイグラウンド]** セクションの **[チャット]** ページに移動します。
1. セットアップ デプロイで、デプロイされた `gpt-35-model` 基本モデルを選択します。
1. チャット ウィンドウで、「`What can you do?`」という質問を入力し、応答を確認します。
    答えは非常に一般的です。 ユーザーに旅行する気を起こさせるチャット アプリケーションを作成しようとしていることを思い出してください。
1. 次のプロンプトでシステム メッセージを更新します。
    ```md
    You are an AI assistant that helps people plan their holidays.
    ```
1. **[保存]** を選択し、**[チャットのクリア]** を選択して、もう一度 `What can you do?` と質問します。アシスタントは、旅行のフライト、ホテル、レンタカーの予約をサポートできると答えたりするでしょう。 この動作を回避しようとします。
1. 新しいプロンプトでシステム メッセージをもう一度更新します。

    ```md
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. **[保存]**、**[チャットのクリア]** の順に選択します。
1. チャット アプリケーションのテストを続けて、取得したデータに基づかない情報が提供されていないことを確認します。 たとえば、次の質問をして、モデルの回答を調べます。
   
    `Where in Rome should I stay?`
    
    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `Give me a list of five bed and breakfasts in Trastevere.`

    ホテルのおすすめ候補を提供しないように指示した場合でも、モデルでホテルの一覧が提供される場合があります。 これは、矛盾した動作の例です。 このような状況で、微調整したモデルのパフォーマンスが向上しているかどうかを調べてみましょう。

1. **[ツール]** の下にある **[微調整]** ページに移動して、微調整ジョブとその状態を確認します。 まだ実行中の場合は、デプロイされた基本モデルを引き続き手動で評価することができます。 完了している場合は、次のセクションに進むことができます。

## 微調整されたモデルをデプロイする

微調整が正常に完了したら、微調整したモデルをデプロイできます。

1. 微調整されたモデルを選択します。 **[メトリック]** タブを選択し、微調整されたメトリックを調べます。
1. 次の構成を使用して、微調整されたモデルをデプロイします。
    - **[デプロイ名]**: *モデルの一意の名前、既定値を使用できます*
    - **デプロイの種類**:Standard
    - **1 分あたりのトークン数のレート制限 (1,000 単位)**:5,000
    - **コンテンツ フィルター**: 既定
1. テストできるようになるには、デプロイが完了するまで待ちます。これには時間がかかる場合があります。

## 微調整したモデルをテストする

微調整したモデルをデプロイしたので、デプロイされた基本モデルをテストしたときと同様に、このモデルをテストできます。

1. デプロイの準備ができたら、微調整したモデルに移動し、**[プレイグラウンドで開く]** を選択します。
1. 以下の手順でシステム メッセージを更新します。

    ```md
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. 微調整したモデルをテストして、動作の一貫性が向上したかどうかを評価します。 たとえば、もう一度次の質問をして、モデルの回答を調べます。
   
    `Where in Rome should I stay?`
    
    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `Give me a list of five bed and breakfasts in Trastevere.`

## クリーンアップ

Azure AI Studio を調べ終わったら、Azure の不要なコストを避けるため、作成したリソースを削除する必要があります。

- [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) に移動します。
- Azure portal の **[ホーム]** ページで、**[リソース グループ]** を選択します。
- この演習のために作成したリソース グループを選びます。
- リソース グループの **[概要]** ページの上部で、**[リソース グループの削除]** を選択します。
- リソース グループ名を入力して、削除することを確認し、**[削除]** を選択します。
