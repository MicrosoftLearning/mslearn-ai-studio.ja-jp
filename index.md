---
title: Azure OpenAI 演習
permalink: index.html
layout: home
---

# Azure AI Studio を使用して生成 AI アプリケーションを開発する

次の演習は、開発者がチャットベースの "Copilot" などの生成 AI アプリケーションを構築するのに使用する一般的なパターンと手法を探索し、Azure AI サービス (特に Azure OpenAI Service と Azure AI Studio) を使用してこれらのパターンを実装する方法を学習する、実践的な学習エクスペリエンスが得られるように設計されています。

これらの演習は自分で完了することができますが、[Microsoft Learn](https://learn.microsoft.com/training/paths/create-custom-copilots-ai-studio/) のモジュールを補完するように設計されています。このモジュールでは、これらの演習の基になる概念の一部について詳しく説明しています。

> **注**: 演習を完了するには、Azure AI Studio で使用される Azure リソースをプロビジョニングし、Azure OpenAI GPT モデルをデプロイして使用するのに十分なアクセス許可とクォータがある Azure サブスクリプションが必要です。 まだお持ちでない場合は、[Azure アカウント](https://azure.microsoft.com/free)にサインアップできます。 新規ユーザーには、最初の 30 日間のクレジットが付属する無料試用版オプションがあります。

## 演習

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}