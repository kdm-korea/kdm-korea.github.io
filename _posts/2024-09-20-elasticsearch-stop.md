---
layout: post
title: Elasticsearch 종료
subtitle: 엘라스틱서치 종료 방법
categories: [ElasticSearch]
sitemap:
  changefreq: daily
  priority: 0.8
banner:
  image: https://bit.ly/3xTmdUP
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags:
  [ElasticSearch, ElasticSearch stop, ElasticSearch kill, ElasticSearch signal]
sidebar: []
---

## 개요

- 엘라스틱 종료 시 보편적으로 Kill 명령어를 사용함. 하지만 강제 종료로 인해 미처리된 리소스 종료와 클러스터 정리 등 작업들을 정상적으로 완료하지 못함.

### 시그널을 추가한 종료

```bash
$ Kill SIGTERM ${PID}
```

- 위 시그널을 추가하게 되면 정상적인 엘라스틱서치 종료가 가능함.

### 치명적인 오류로 인한 중단

- Elasticsearch에서 치명적인 오류 감지 시 자동으로 중지함. 해당 경우 아래 특수 코드를 함께 반환함.

  | 에러코드 | 설명                             |
  | :------- | :------------------------------- |
  | 158      | jvmkiller 에이전트에 의해 사망됨 |
  | 143      | 사용자 또는 커널 SIGTERM         |
  | 137      | 커널 oom-killer에 의해 살해됨    |
  | 134      | 세그먼테이션 오류                |
  | 128      | JVM 내부 오류                    |
  | 127      | 메모리 부족 오류                 |
  | 126      | 스택 오버플로우 오류             |
  | 125      | 알 수 없는 가상 머신 오류        |
  | 124      | 심각한 I/O 오류                  |
  | 78       | 부트스트랩 검사 실패             |
  | 1        | 알 수 없는 치명적인 오류         |

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
