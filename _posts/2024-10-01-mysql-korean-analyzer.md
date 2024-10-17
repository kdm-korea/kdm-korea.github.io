---
layout: post
title: MySQL에서의 한국어 형태소 분석 및 전문검색 테스트
subtitle: FullText Search, MeCab을 활용한 사용자 검색 고도화
categories: [mysql, analyzer, fulltext search, FTS, MeCab]
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
tags: [mysql, analyzer, fulltext search, FTS, MeCab, 형태소 분석]
sidebar: []
---

# MySQL에서 형태소 분석 및 전문검색 방법

## 개요

- Elasticsearch를 활용한 검색만을 사용하다가, ES검색을 사용하지 않고 DB로만 사용하고 싶다는 요구가 있어 만들어보게 되었다.
- 먼저, InnoDB에서 지원하는 FTS를 사용하여 ngram을 추가하고, 검색결과 고도화를 위해 MeCab을 이용하여 형태소 분석을 진행하였다.

### 1. MySQL FullText Search

- MySQL 내 **MyISAM은 MySQL5.5버전 이상부터 지원**하고, **InnoDB는 MySQL 5.6버전**부터 FTS(FullText Search)를 지원한다. CJK(중국어, 일본어, 한국어)를 지원한다.
- 기본적인 InnoDB 전문검색(FTS)는 구분자(공백, 단어분리자)를 이용하여 토큰을 생성한다. 하지만 한국어는 **조사와 같은 품사**가 있어, 다른 방법을 사용해야 한다.
- 인덱스 파서 방식은 여러가지가 있지만 검색 시 가장 매칭이 높게 나올 수 있는 `ngram parser`방식을 사용하여 토큰화 해보았다.
- `minimum_length`가 2인 경우

  ```
  ex) 대한민국
  analyzer => 대한, 한민, 민국
  ```

- `minimum_length`가 3인 경우

  ```
  ex) 대한민국
  analyzer => 대한민, 한민국
  ```

  > 위와 같이 MySQL에서 ngram은 단일 길이로만 토큰이 생성된다. 최소~최대 길이 ngram 토큰을 생성하는 Multi parser index를 사용하는 방식도 존재한다고 한다.

- 사용방법

  1. Table 생성 시 검색 대상 필드에 사용한 분석기와 FTS 인덱스 설정 / 테이블 수정

     ```sql
     mysql> CREATE TABLE articles
     (
             FTS_DOC_ID BIGINT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
             title VARCHAR(100),
             FULLTEXT INDEX ngram_idx(title) WITH PARSER ngram
     ) Engine=InnoDB CHARACTER SET utf8mb4;
     Query OK, 0 rows affected (1.26 sec)

     mysql> # ALTER TABLE articles ADD FULLTEXT INDEX ngram_idx(title) WITH PARSER ngram;
     mysql> # CREATE FULLTEXT INDEX ngram_idx ON articles(title) WITH PARSER ngram;
     ```

     데이터 생성...

  2. 토큰화된 데이터 확인 및 결과 유추

     ```sql
     mysql> INSERT INTO articles (title) VALUES ('my sql');
     Query OK, 1 row affected (0.03 sec)

     mysql> SET GLOBAL innodb_ft_aux_table="test/articles";
     Query OK, 0 rows affected (0.00 sec)

     mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE;
     ```

     Set Global 쿼리는 테이블 최적화 시 테이블 크기에 따라 오래 걸릴 수 있기 때문에 FullText에 대한 캐시 확인

  3. 검색 시 SQL

     ```sql
     mysql> SELECT * FROM articles WHERE MATCH(title) AGAINST('sql' IN BOOLEAN MODE);
     ```

     추가적으로 제외어, 조건절 등 여러 검색 연산자를 지원함.

     | 조건절 | 설명                 |
     | :----: | :------------------- |
     |   \+   | and 조건             |
     |   \-   | not 조건             |
     |   \<   | 검색순위 하햫        |
     |   \>   | 검색순위 상향        |
     |  \()   | 검색결과 내 검색     |
     |   \~   | 검색순위 최저 하향   |
     |   \*   | 검색어 기준 전체검색 |
     |  \""   | 구문검색             |

     ```sql
     mysql> SELECT * FROM articles WHERE MATCH(title) AGAINST('+sql -my' IN BOOLEAN MODE);
     ```

  4. 문제점

     (정확성)해당 Analyzer를 사용했을 때, 검색의 결과는 나오지만, 사용자 니즈에 맞는 **정확한 검색결과**를 돌출하지 않는다.

     ```
     ex) 검색어: ten
     result: sentence, ten, ...
     ```

     위 에시에서 숫자를 검색했을떄 뜻하지 않은 sentence가 반환된다.

     사용자의 의도를 파악하여 결과를 반환하려면 의미있는 단어에 대한 필터링이 필요했다.

     위 조건에 맞추려면 형태소 분석이 필요하여 플러그인을 추가하게 되었다.

  5. stopWord는 불용어 용어집을 추가하고, 토큰을 확인하여 불필요한 단어 추가

</br>

### 2. MeCab, 한국어 형태소 분석

- MySQL에서 MeCab을 플러그인 형태로 지원하고 있어 사용하게 되었다.
- MeCab은 일본어 형태소 분석기 플러그인으로써, 기본적으로 한국어는 존재하지 않는다. 해당 플러그인으로 한국어 형태소 분석을 하기 위해 은전한닢의 사전을 사용한다.
- 은전한닢과 MaCab의 설치, 연결되어 있다는 가정으로 진행함

1. [은전한닙, MeCab 설치 =>](https://kdm-korea.github.io/mysql/analyzer/fulltext%20search/fts/mecab/2024/09/27/mysql-mecab-install.html)
2. Mysql 설정에 플러그인 등록

   ```sh
   vi /etc/mysql/my.cnf

   # mecab 한국어 사전 추가
   [mysqld]
   loose-mecab-rc-file = /path/to/mecab/etc/mecabrc
   innodb_ft_min_token_size = 1
   ```

3. 플러그인 설치
   ```sql
   install plugin mecab soname 'libpluginmecab.so';
   ```
4. 테이블 생성 및 사용
   ```sql
   mysql> CREATE TABLE articles_mecab (id SERIAL PRIMARY KEY, title VARCHAR(255), FULLTEXT(title) WITH PARSER mecab) CHARACTER SET utf8;
   Query OK, 0 rows affected (1.26 sec)
   ```
5. 결과 확인

   ```sql
   mysql> INSERT INTO articles_mecab (title) VALUES ('my sql');
   Query OK, 1 row affected (0.03 sec)

   mysql> SET GLOBAL innodb_ft_aux_table="test/articles_mecab";
   Query OK, 0 rows affected (0.00 sec)

   mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE;
   ```

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
