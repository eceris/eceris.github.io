---
layout:            post
title:             "CentOS6 pymssql 설치"
date:              2016-11-03 17:30:00 +0300
tags:              pymssql python
category:          Infra
author:            eceris
---

### **CentOS 6 에서 pymssql 설치**

Python에서 MSSQL에 접속하여 쿼리를 수행하고 싶다면 pymssql 모듈을 설치해야하는데 그 과정을 정리한다.

크게는 아래와 같이 진행한다.

```bash
1. pymssql 모듈 다운로드

2. setup.py install 명령어로 설치

3. trouble shooting

```  

#### **pymssql 다운로드 및 압축 해제**
* https://pypi.python.org/pypi/pymssql/2.1.3#downloads 에서 pymssql-2.1.3.tar.gz 를 다운로드
* tar xvzf pymssql-2.1.3.tar.gz

#### **psycopg2 모듈 다운로드**
* python3.3 을 통해 설치
* ..../python33/bin/python3.3 setup.py install

#### **trouble shooting**
* sqlfront.h 파일을 찾지 못한다는 에러
```bash
[root@do pymssql-2.1.3]# /opt/TerraceTims/3rd/python33/bin/python3.3 setup.py install
setup.py: platform.system() => 'Linux'
setup.py: platform.architecture() => ('64bit', 'ELF')
setup.py: platform.linux_distribution() => ('CentOS', '6.5', 'Final')
setup.py: platform.libc_ver() => ('glibc', '2.3.2')
setup.py: Not using bundled FreeTDS
setup.py: include_dirs = ['/usr/local/include']
setup.py: library_dirs = ['/usr/local/lib']
Download error on https://pypi.python.org/simple/setuptools_git/: [Errno -2] Name or service not known -- Some packages may not be found!
Installed /opt/pymssql-2.1.3/setuptools_git-1.1-py3.3.egg
running install
running bdist_egg
running egg_info
writing dependency_links to pymssql.egg-info/dependency_links.txt
writing top-level names to pymssql.egg-info/top_level.txt
writing pymssql.egg-info/PKG-INFO
reading manifest file 'pymssql.egg-info/SOURCES.txt'
writing manifest file 'pymssql.egg-info/SOURCES.txt'
installing library code to build/bdist.linux-x86_64/egg
running install_lib
running build_ext
building '_mssql' extension
creating build
creating build/temp.linux-x86_64-3.3
gcc -pthread -fno-strict-aliasing -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -fPIC -I/usr/local/include -I/opt/TerraceTims/3rd/python33/include/python3.3m -c _mssql.c -o build/temp.linux-x86_64-3.3/_mssql.o -DMSDBLIB
_mssql.c:266:22: error: sqlfront.h: 그런 파일이나 디렉터리가 없습니다
In file included from _mssql.c:268:
cpp_helpers.h:34:19: error: sybdb.h: 그런 파일이나 디렉터리가 없습니다
```
* sqlfront.h, sybdb.h 를 python이 찾을 수 있도록 해주자.
```bash
해결방안 : /usr/local/include, /usr/local/lib  디렉토리가 gcc 컴파일시 import 되고 있으므로 못찾고 있는 파일들을 해당 위치에 복사하면 끝!
find . -name sqlfront.h
 >> ./freetds/nix_64/include/sqlfront.h
 >> ./freetds/nix_32/include/sqlfront.h
find . -name sybdb.h
 >> ./freetds/nix_64/include/sybdb.h
 >> ./freetds/nix_32/include/sybdb.h
cp pymssql-2.1.3/freetds/nix_64/include/* /usr/local/include
cp pymssql-2.1.3/freetds/nix_64/lib/* /usr/local/lib
```







