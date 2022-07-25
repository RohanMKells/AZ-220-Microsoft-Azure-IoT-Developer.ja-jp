---
title: オンライン ホステッド インストラクション
permalink: index.html
layout: home
ms.openlocfilehash: c906809bcfcfd6cfac0139d9c6d9432cff05fbf1
ms.sourcegitcommit: 5e9f89d47b27285feaf13cacfa097ddd4b888a90
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/14/2022
ms.locfileid: "137893005"
---
# <a name="content-directory"></a>コンテンツ ディレクトリ

各ラボの演習とデモへのハイパーリンクを以下に示します。

## <a name="labs"></a>ラボ

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

{% comment %}
<!-- Comment out the Jekyll template that lists the placeholder demo -->

## <a name="demos"></a>デモ

{% assign demos = site.pages | where_exp:"page", "page.url contains '/Instructions/Demos'" %}
| モジュール | デモ |
| --- | --- | 
{% for activity in demos %}| {{ activity.demo.module }} | [{{ activity.demo.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

{% endcomment %}
