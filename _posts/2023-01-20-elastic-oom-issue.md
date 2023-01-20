---
layout: post
title: ElasticSearch OLD GC로 인한 OOM 발생이슈
subtitle: Text analyzer token issue
categories: [elasticSearch]
tags: [ElaticSearch, Node Cluster, Old GC, ElasticSearch OOM]
---

## ElasticSearch OLD GC로 인한 OOM 발생이슈

10,000여건 되는 데이터를 색인하던 중 ES의 클러스터에서 힙메모리가 급작스럽에 풀을 쳤다. GC 로그를 확인해보니 OLD GC가 초당 5번정도 일어나고 있었다.

운영이 시작된 시점에서 Aggregation의 호출이 많아 생기는 문제로 생각되어 GC를 수정하는 방향을 생각했다. 사용하는 GC는 CMS GC를 사용하고 있었으며, G1GC를 통해 OLD GC를 줄이는 방식을 생각했다.

하지만 GC의 문제는 아니었고, 이용자가 없는 새벽 시간에도 GC는 짧은 간격으로 돌고 있었다. 데이터를 전체적으로 다시 색인하면서 시간 체크를 통해 데이터 상 문제가 있는지 체크하였다. 다음날 확인해보니 유독 색인에 긴 시간을 필요로 하는 데이터가 존재했다.

해당 데이터의 필드는 공백 없이 10,000자가 넘어갔다.

확인해보니, lucene에서 데이터 분석 시 max 00을 넘어 발생하는 문제였다. 해당 데이터는 중요도가 떨어지는 데이터로 판단되어 5000자 기준으로 공백을 넣어 색인 처리하였다.
