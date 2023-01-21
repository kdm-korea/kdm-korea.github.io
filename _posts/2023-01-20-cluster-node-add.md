---
layout: post
title: Elasticsearch Cluster node 증설
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
tags: [ElaticSearch, ElasticSearch Cluster, Node Cluster]
sidebar: []
---

# Elasticsearch Cluster node 증설

### 1. 개요

1. 고가용성 및 검색속도 안정화를 위해, 노드 2대에서 6대로 ES서버를 늘려야 되는 상황이 발생했다. 클러스터의 노드를 증설 시 ES 설정을 적어놓는다.

### 2. 서버 증설을 위한 ES 설정

1.  ES 클러스터를 위한 기본설정

    ```yaml
    cluster.name: cluster_test
    node.name: node1
    network.host: 111.0.111.111
    discovery.seed_hosts: [""]
    cluster.initial_master_nodes: ["node1"]
    ```

    1. cluster.name
       1. 클러스터를 구성하는 이름, ES의 모든 클러스터명은 동일해야 같은 집단으로 구성됨
    2. node.name
       1. 각 ES의 별칭, 고유한 값을 지녀야 함
    3. network.host
       1. 클러스터 구성을 위한 네크워크 IP
    4. discovery.seed_hosts
       1. Cluster를 구성하는 모든 ES의 IP:PORT
       2. 해당 PORT는 내부 통신 포트(transport.port)로 구성해야 함.
    5. cluster.initial_master_nodes
       1. 클러스터를 구성하기 위한 노드명

2.  예시

    - 1번 노드

      ```yaml
      ------------------------ Cluster -------------------------
      cluster.name: cluster_test
      -------------------------- Node --------------------------
      node.name: node1
      ------------------------- NetWork ------------------------
      network.host: '111.0.111.111'
      http.port: 100
      transport.port: 101
      ------------------------- Discovery ----------------------
      discovery.seed_hosts: ['111.0.111.111:101', '111.0.111.111:201', '111.0.111.111:301']
      cluster.initial_master_nodes: ['node1', node2, node3]
      ```

    - 2번 노드

      ```yaml
      ------------------------ Cluster -------------------------
      cluster.name: cluster_test
      -------------------------- Node --------------------------
      node.name: node2
      ------------------------- NetWork ------------------------
      network.host: '111.0.111.111'
      http.port: 200
      transport.port: 201
      ------------------------- Discovery ----------------------
      discovery.seed_hosts: ['111.0.111.111:101', '111.0.111.111:201', '111.0.111.111:301']
      cluster.initial_master_nodes: ['node1', node2, node3]
      ```

    - 3번 노드
      ```yaml
      ------------------------ Cluster -------------------------
      cluster.name: cluster_test
      -------------------------- Node --------------------------
      node.name: node3
      ------------------------- NetWork ------------------------
      network.host: '111.0.111.111'
      http.port: 300
      transport.port: 301
      ------------------------- Discovery ----------------------
      discovery.seed_hosts: ['111.0.111.111:101', '111.0.111.111:201', '111.0.111.111:301']
      cluster.initial_master_nodes: ['node1', node2, node3]
      ```

3.  클러스터 설정 시 주의점
    1.  minium master node
        1. 데이터를 유지할 수 있는 최소 Master Node의 수를 기입
        2. Master Node가 최소 Master Node 보다 적게 구동된다면 Split Brain이 발생할 수 있음.

<ins class="kakao_ad_area" style="display:none;"
data-ad-unit = "DAN-IR3SEKWYp9BSWUj6"
data-ad-width = "320"
data-ad-height = "100"></ins>

<script type="text/javascript" src="//t1.daumcdn.net/kas/static/ba.min.js" async></script>
<script>
function changeGiscusTheme () {
    const theme = document.documentElement.getAttribute('data-theme') === 'dark' 'preferred_color_scheme' : 'light_tritanopia';

    function sendMessage(message) {
      const iframe = document.querySelector('iframe.giscus-frame');
      if (!iframe) return;
      iframe.contentWindow.postMessage({ giscus: message }, 'https://giscus.app');
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
        data-theme= "preferred_color_scheme"
        data-lang="ko"
        crossorigin="anonymous"
        async>
</script>

document.getElementsByTagName('html')[0].getAttribute('data-theme')
