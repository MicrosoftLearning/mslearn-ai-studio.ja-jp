---
title: Azure OpenAI 演習
permalink: index.html
layout: home
---

# Azure AI Studio の演習

次の演習は、[Microsoft Learn](https://learn.microsoft.com/training) のモジュールをサポートするように設計されています。

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}