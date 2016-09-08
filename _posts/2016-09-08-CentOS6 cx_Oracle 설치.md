---
layout:            post
title:             "CentOS6 cx_Oracle 설치"
date:              2016-09-08 12:50:00 +0300
tags:              cx_Oracle python
category:          OS
author:            eceris
---

### **CentOS 6 에서 cx_Oracle 설치**

파이썬에서 Oracle DB에 붙어서 쿼리를 하고 싶다면 cx_Oracle을 설치해야 하는데 그 과정을 정리한다.

크게는 아래와 같이 진행한다.

```bash
1. Oracle Instant Client가 필요한 필수 라이브러리 설치

2. Oracle Instant Client 설치

3. Oracle 에서 사용하는 path 정보 추가

4. python의 easy_install을 사용하여 cx_Oracle 설치
```  

#### **Oracle Instant Client의 필수 라이브러리 설치**
* yum -y install libaio bc flex

#### **Oracle Instant Client 설치**
* http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html 에서 아래 두가지 rpm을 다운로드
* oracle-instantclient11.2-basic-11.2.0.3.0-1.x86_64.rpm
* oracle-instantclient11.2-devel-11.2.0.3.0-1.x86_64.rpm
* 해당 rpm을 설치(rpm -ivh xxxx.rpm)

#### **Oracle 에서 사용하는 path 정보 추가**

	echo '# Convoluted undocumented Oracle Config.' >> $HOME/.bashrc
	echo 'export ORACLE_VERSION="11.2"' >> $HOME/.bashrc
	echo 'export ORACLE_HOME="/usr/lib/oracle/$ORACLE_VERSION/client64/"' >> $HOME/.bashrc
	echo 'export PATH=$PATH:"$ORACLE_HOME/bin"' >> $HOME/.bashrc
	echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:"$ORACLE_HOME/lib"' >> $HOME/.bashrc
	. $HOME/.bashrc

* 시동디스크 선택창에 iso 파일 선택 -- [시작]

#### **python의 easy_install을 사용하여 cx_Oracle 설치**
* python이 설치되어 있는 경로의 bin/easy_install을 이용하여 cx_oracle 설치
* 아래는 DO에 기본 포함되어 있는 python을 이용


	/opt/TerraceTims/3rd/python33/bin/easy_install-3.3 cx_oracle

#### **테스트**
* python에서 import cx_Oracle 했을 때 정상 작동 되는지 확인
![](https://eceris.github.io/media/img/20160908_cxoracle.png)