---
layout: post
title: MySQL에 MeCab, 은전한닢 설치
subtitle: MySQL 한국어 형태소분석기 플러그인 추가를 위한 설치
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

# MySQL에 MeCab, 은전한닢 설치

## 개요

- 도커로 centos7을 띄어 설치하고 테스트해보았지만, 설치해야 되는 리스트가 많아 별도로 쓰게 되었다.

### 1. 서버제원

- Docker image

  ```sh
  $ docker pull centos:centos7.9.2009
  $ docker run -ti -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 80:80 centos:centos7.9.2009
  ```

- OS 정보

  ```sh
  [root@3ae62c03f268 ~]# grep . /etc/os-release
  NAME="CentOS Linux"
  VERSION="7 (AltArch)"
  ID="centos"
  ID_LIKE="rhel fedora"
  VERSION_ID="7"
  PRETTY_NAME="CentOS Linux 7 (AltArch)"
  ANSI_COLOR="0;31"
  CPE_NAME="cpe:/o:centos:centos:7:server"
  HOME_URL="https://www.centos.org/"
  BUG_REPORT_URL="https://bugs.centos.org/"
  CENTOS_MANTISBT_PROJECT="CentOS-7"
  CENTOS_MANTISBT_PROJECT_VERSION="7"
  REDHAT_SUPPORT_PRODUCT="centos"
  REDHAT_SUPPORT_PRODUCT_VERSION="7"
  ```

- CPU 정보

  ```sh
  [root@3ae62c03f268 ~]# lscpu
  Architecture:          aarch64
  Byte Order:            Little Endian
  CPU(s):                8
  On-line CPU(s) list:   0-7
  Thread(s) per core:    1
  Core(s) per socket:    8
  Socket(s):             1
  Model:                 0
  BogoMIPS:              48.00
  L1d cache:             unknown size
  L1i cache:             unknown size
  L2 cache:              unknown size
  Flags:                 fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimdrdm jscvt fcma lrcpc dcpop sha3 asimddp sha512 asimdfhm dit uscat ilrcpc flagm sb paca pacg dcpodp flagm2 frint
  ```

### 2. 설치

1.  yum 설치 및 확인

    ```sh
    $ yum --version
    ```

    yum 사용 시 "curl#6 - "Could not resolve host: mirrorlist.centos.org;..." 에러 발생 시 아래 명령어 사용,

    dns, 포트 등 다양한 원인으로 에러가 발생함, 에러 발생 저장소를 변경하는 형태로 사용

    ```sh
    $ sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
    $ sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
    ```

2.  linux 기본 유틸 설치

    ```sh
    yum install wget
    yum install make
    ```

3.  C, C++ 컴파일러 설치

    ```sh
    # c complier gcc install
    yum install gcc
    # gcc install check
    gcc --version

    # c++ compiler g++ install
    Yum install gcc-c++
    # g++ install check
    g++ --version
    ```

4.  MeCab 설치

    ```sh
    $ wget https://bitbucket.org/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz
    $ tar xvfz mecab-0.996-ko-0.9.2.tar.gz
    $ cd mecab-0.996-ko-0.9.2/
    $ ./configure --prefix=/path/to/mecab
    $ make
    $ make install
    ```

    make 명령어 사용 시 arm64인 경우 make 커맨드에서 아래 오류발생..

    > aarch64 configure: error: cannot guess build type;...

    configure에서 cpu 아키텍쳐를 명시하면 됨.

    ```sh
    $ ./configure --prefix=/path/to/mecab --build=aarch64-unknown-linux-gnu
    ```

5.  은전한닢 설치

    ```sh
    $ wget https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.1.1-20180720.tar.gz
    $ tar xvfz mecab-ko-dic-2.1.1-20180720.tar.gz
    $ cd mecab-ko-dic-2.1.1-20180720/
    $ ./configure --with-mecab-config=/path/to/mecab/bin/mecab-config
    $ make
    $ make install
    ```

    **은전한닢**은 가장 최근 사전으로 추가하여 사용, 2024.09 기준 최근 사전 추가함.

6.  MeCab에 은전한닙 사전 등록

    ```sh
    vi /path/to/mecab/etc/mecabrc

    # 경로변경
    dicdir =  /path/to/mecab/lib/mecab/dic/mecab-ko-dic/
    ```

7.  라이브러리 경로 등록

    ```sh
    $ vi ~/.bash_profile

    #내용추가
    export PATH=/path/to/mecab/bin/:$PATH
    export LD_LIBRARY_PATH=/path/to/mecab/lib/:$LD_LIBRARY_PATH
    ```

8.  설정 동기화

    ```sh
    $ source ~/.bash_profile
    ```

9.  테스트

    ```sh
    $ echo "꽁꽁 얼어붙은 한강위를 고양이가 걸어다닙니다." |mecab

    꽁꽁	MAG,*,T,꽁꽁,*,*,*,*,*
    얼어붙	VV,*,T,얼어붙,*,*,*,*,*
    은	ETM,*,T,은,*,*,*,*,*
    한	MM,~가산명사,T,한,*,*,*,*,*
    강위	NNG,*,F,강위,Compound,*,*,강+위,강/NNG/*/1/1+강위/Compound/*/0/2+위/NNG/*/1/1
    를	JKO,*,T,를,*,*,*,*,*
    고양이	NNG,*,F,고양이,*,*,*,*,*
    가	JKS,*,F,가,*,*,*,*,*
    걸	VV,*,T,걸,Inflect,VV,VV,걷/VV,*
    어	EC,*,F,어,*,*,*,*,*
    다닙니다	VV+EF,*,F,다닙니다,Inflect,VV,EF,다니/VV+ᄇ니다/EF,*
    .	SF,*,*,*,*,*,*,*,*
    EOS
    ```

    위 토큰에서 더 필요하다고 생각되는 토큰은 사용자 사전에 추가하여 사용하면 된다.

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
