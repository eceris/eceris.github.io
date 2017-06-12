---
layout:            post
title:             "CentOS6 psycopg2 설치"
date:              2016-09-23 17:30:00 +0300
tags:              psycopg2 python
category:          Infra
author:            eceris
---

### **CentOS 6 에서 psycopg2 설치**

Python에서 Postgresql 에 접속하여 쿼리를 수행하고 싶다면 psycopg2 모듈을 설치해야하는데 그 과정을 정리한다.

크게는 아래와 같이 진행한다.

```bash
1. yum 으로 python과 postgresql devel rpm을 설치

2. psycopg2 모듈 다운로드

3. psycopg2 모듈 빌드 후 설치

```  

#### **python-devel, postgresql-devel rpm 설치**
* yum -y install python-devel postgresql-devel

#### **psycopg2 모듈 다운로드**
* http://initd.org/psycopg/download/ 에서 Download the source package 를 눌러 다운로드
* tar xvzf psycopg2-2.6.1.tar.gz

#### **psycopg2 모듈 빌드 후 설치**
* /opt/TerraceTims/3rd/python33/bin/python3.3 setup.py build
* /opt/TerraceTims/3rd/python33/bin/python3.3 setup.py install
