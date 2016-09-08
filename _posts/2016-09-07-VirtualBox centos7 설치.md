---
layout:            post
title:             "VirtualBox centos7 설치"
date:              2016-09-07 16:50:00 +0300
tags:              VirtualBox centos 7.0
category:          OS
author:            eceris
---

### **VirtualBox centos 7.0 설치 및 고정아이피 할당**

jekyll 을 docker로 설치하기 위해 centos7.0 을 VirtualBox에 설치한다.

크게는 아래와 같은 단계로 진행한다.

```bash
1. VM 생성

2. 네트워크 설정

3. VM 시작, ISO 마운트

4. CentOS 7 설치

5. 네트워크 설정 및 네트워크 서비스 재시작
```  

#### **VM 생성**
* Virtual Box 실행
* [새로만들기] 클릭
* [가상머신 만들기] -- 이름 : centos -- [다음]
* 메모리 크기 -- [다음]
* 하드디스크 -- [만들기]
* 가상하드 디스크 만들기 -- [다음]
* 동적할당 -- [다음]
* 파일 위치 및 크기 [만들기]
이제 Centos 이라는 vm이 보인다.

#### **네트워크 설정**
* CentOS7 우클릭 -- [설정]
* 네트워크 -- 어댑터 1 -- 다음에 연결됨: NAT -- 케이블연결됨 : check
* 네트워크 -- 어댑터 2 -- 다음에 연결됨: 호스트전용어댑터 -- 케이블연결됨 : check
![](https://eceris.github.io/media/img/20160907_network.png)
* [확인]

#### **VM 시작, ISO 마운트**
* [시작] 버튼을 클릭하여 vm 시작
* 시동디스크 선택창에 iso 파일 선택 -- [시작]

#### **CentOS 7 설치**
* ↑키를 눌러 Install CentOS 7 선택 -- [엔터]
* (기본값) English -- [Continue]
* [INSTALLATION DESTINATION 클릭]
* 좌상단 [DONE] 클릭
* [NETWORK & HOSTNAME 클릭]
* Ethernet 두개를 모두 [ON] 클릭
* 좌상단 [DONE] 클릭
* [Begin Installation] 클릭 
* [ROOT PASSWORD] 클릭
* Root Password: P@ssw0rd --- Confirm: P@ssw0rd --- [Done] 클릭
* [Reboot] 클릭

#### **네트워크 설정 및 네트워크 서비스 재시작**
* [/etc/sysconfig/network-scripts/ifcfg-*]하위의 설정을 변경
* 아래와 같이 Ethernet 2개를 설정(HWADDR은 VM네트워크 설정에 있는 MAC 주소를 지정)
* ifcfg-enp0s3
![](https://eceris.github.io/media/img/20160907_enp0s3.png)
* ifcfg-enp0s8
![](https://eceris.github.io/media/img/20160907_enp0s8.png)
* [sudo network service restart] 명령어로 네트워크 서비스 재시작

