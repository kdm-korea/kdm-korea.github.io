---
layout: post
title: Elasticsearch Reindex 방법
subtitle: 데이터 이관 및 재색인에 대한 고민
categories: [ElasticSearch]
banner:
  image: https://bit.ly/3xTmdUP
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: [ElaticSearch, ElasticSearch reindex, ElasticSearch Data Migration]
sidebar: []
---

# ReIndex

## 개요

- 인덱스를 복제할 수 있는 방법이다. 대량의 문서를 재색인해야 하는 경우 혹은, 데이터 복제본을 이용한 무언가의 작업이 필요한 경우 데이터를 복제할 수 있는 기능이다.

### 데이터 이관 방법에 대한 협의

- 이전 데이터를 마이그래이션하는 과정에서 데이터를 어떤 방식으로 이관할지에 대한 논의가 나왔다.

  1. 스냅샷을 이용한 데이터 이관

  2. Reindex를 이용한 데이터 이관

스냅샷은 우리가 필요로 하는 데이터를 제외하고도 수많은 인덱스가 생성되어 있었고, 신규 엘라스틱에 색인된 데이터가 존재하는 상태였다. 하여 `Reindex`를 이용한 데이터 이관을 하게 되었다.

### ReIndex 방법

1. 기존 ES의 elasticsearch.yml에 신규 ES의 주소를 입력 후 재기동한다. 동일한 서버 내 이관 시에는 필요없다.

   ```yaml
   reindex.remote.whitelist:
   ```

2. 신규 es를 이용하는 키바나에서 아래 명령어를 입력한다.

   ```js
   POST _reindex?wait_for_completion=false
   {
     "source": {
       "index": "index_old"
     },
     "dest": {
       "index": "index_new"
     }
   }
   ```

   - `wait_for_complate`
     - 끝나는 시점까지 기다릴지 여부
   - elasticsearch의 http는 **Timeout**이 **30초**가 기본값이다.

     </br>

3. 반환된 태스크값을 이용하여 실행경과를 확인한다.

   ```js
   GET _tasks/jrw3kRS2QaGqunpyqm7Paw:40188327
   ```

---

### 이슈사항

1. thread pool

   위와 같은 방법으로 진행했을 때 thread가 pool나는 현상이 발생했다. 확인해보니, 데이터의 크기가 커서 생기는 문제였다. 기본 배치의 문서 갯수는 1,000이 기본값이다.

   ES에서는 100mb 이하의 용량으로 인덱싱 작업을 권장하였고, 현행 데이터에서 평균크기를 구하여 100개씩 색인하였다. 또한, 특수하게 큰 데이터가 존재할 수 있어, scroll 시간을 60분으로 맞추었다.

   ```js
     POST _reindex?wait_for_completion=false&scroll=60m
     {
       "source": {
         "index": "index_old",
         "size": 100
       },
       "dest": {
         "index": "index_new"
       }
     }
   ```

2. rejected exception

   해당 문제는 한번에 너무 많은 문서를 reindex하여 발생하는 문제였다. 수많은 벌크 인덱스가 인입되었으며, 처리속도가 이를 따라가지 못해 큐가 다 차버리면서 es가 동작하지 않았다.</br>
   큐의 할당은 **1,000**이 기본값이다.</br>
   해당 문제 해결을 위해 큐 사이즈를 5,000으로 변경하고 reindex의 갯수를 두개씩 진행하는 방향으로 설정하였다.

### 테스트

reindex 작업 시 배치 사이즈가 클수록 재색인 속도가 빠를까?

1. size를 10개로 잡아 재색인
2. size를 1,000로 잡아 재색인

데이터를 테스트해본 결과 확연하게 배치가 클수록 속도가 빨랐다.</br>
서버간 이관이라 3 way handshake도 지속적으로 이루어질테고, 데이터 복사에 따른 피봇관리가 리소스를 차지한다고 생각이 들었다.</br>
문서 각기의 데이터를 유추할 수 있다면 100mb 이하가 되는한 가장 큰 사이즈로 구성하는것을 추천한다.

### 속도개선

reindex 진행 시 속도를 개선할 수 있는점을 확인해보았다.</br>
물론 성능를 높이는 서버 증설이나, 힙메모리 증대는 허용하는 최대치(31GB)를 맞춰놓은 상태였다.</br>
index의 replica shard의 갯수를 0으로 변경하여 재색인 속도를 향상시켰다.

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
