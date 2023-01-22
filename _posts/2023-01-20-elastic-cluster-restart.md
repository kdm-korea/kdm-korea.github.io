---
layout: post
title: Elasticsearch Cluster node 재시작
subtitle: 엘라스틱 노드 클러터링 간 설정
categories: [ElasticSearch]
banner:
  image: https://bit.ly/3xTmdUP
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: [ElaticSearch, ElasticSearch Cluster, Node Cluster restart, Node restart]
sidebar: []
---

# Elasticsearch Cluster node 재시작

### 1. 개요

1. 고가용성 및 검색속도 안정화를 위해, 노드 2대에서 6대로 ES서버를 늘려야 되는 상황이 발생했다. 클러스터의 노드를 증설 시 순서를 적어놓는다.

2. ES의 설정 변경/ 서버 가감 등, 관리자의 의도된 서버 재시작에 대한 작업은 Rolling Restart의 방법을 사용하는 것이 좋다.

</br>

### 2. Rolling Restart 순서

1. ReBalancing Disable

   ```js
   PUT _cluster/settings
   {
     "transient": {
       "cluster.routing.allocation.enable": "primary"
     }
   }
   ```

   ES 기본 설정으로 노드의 가감이 일어나게 되면 Shard의 Rebalancing이 일어나게 되어 검색속도 및 색인속도에 영향을 받게 된다.

   아래 옵션 적용 시 노드 증설 시 사드의 리밸런싱을 맏아 리소스를 낭비를 최소화할 수 있다.

</br>

2. 노드 가감 & 재기동

</br>

3. ReBalancing able

   ```js
   PUT _cluster/settings
   {
     "transient": {
       "cluster.routing.allocation.enable": "all"
     }
   }
   ```

   리밸런싱을 활성화한다. 리밸런싱 기능이 켜져있지 않다면, `Primary Shard`가 소실되었을 때 `Replica Shard`가 존재하더라도 `Primary Shard`로 승격되는 일이 일어나지 않는다. 또한 신규 색인 시 각 노드간 데이터의 Shard 불균형을 초래하게 된다.

<ins class="kakao_ad_area" style="display:none;"
data-ad-unit = "DAN-IR3SEKWYp9BSWUj6"
data-ad-width = "320"
data-ad-height = "100"></ins>

<script type="text/javascript" src="//t1.daumcdn.net/kas/static/ba.min.js" async></script>
<script>
function changeGiscusTheme () {
    const theme = document.documentElement.getAttribute('data-theme') === 'dark' 'preferred_color_scheme' : 'light_tritanopia'

    console.log(theme)

    function sendMessage(message) {
      const iframe = document.querySelector('iframe.giscus-frame');
      if (!iframe) return;
      iframe.contentWindow.postMessage({ giscus: {
      setConfig: {
        theme: theme
      }
    } }, 'https://giscus.app');
    }

    sendMessage({
      setConfig: {
        theme: theme
      }
    });
  }
</script>
<script src="https://giscus.app/client.js"
        data-repo="kdm-korea/kdm-korea.github.io"
        data-repo-id="R_kgDOIzxYeA"
        data-category="Q&A"
        data-category-id="DIC_kwDOIzxYeM4CTtII"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="top"
        data-theme= "light_tritanopia"
        data-lang="ko"
        crossorigin="anonymous"
        async>
</script>
