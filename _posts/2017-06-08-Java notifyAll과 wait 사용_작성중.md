---
layout:            post
title:             "Java 표준 API의 함수적 인터페이스"
date:              2017-05-31 09:50:00 +0300
tags:              Java
category:          Server
author:            eceris
---

# Java 표준 API의 함수적 인터페이스 #
http://everysw.tistory.com/entry/notifyAll-%EA%B3%BC-wait-%EC%82%AC%EC%9A%A9%EC%8B%9C-%EC%A3%BC%EC%9D%98%EC%A0%90

자바의 버전은 크게 두가지로 구분한다. 기능적인 부분의 개선이 주를 이루는 진화계(evolution), 언어자체의 형태 변화가 오는 혁명계(revolution). 자바 6과 7은 진화계, 자바 5와 8은 혁명계에 속한다. 자바 8에 포함되는 람다식과 관련되는 Functional Interface에 대해 알아본다.

자바의 표준 API에서 한 개의 추상 메소드만을 갖는 인터페이스들은 모두 람다식을 이용하여 간단하게 표현할 수 있다. 예를 들면 Runnable인터페이스는 매개변수와 리턴값이 없는 run() 메소드만 존재하기 때문에 다음과 같이 구현 할 수 있다.
```java
public class Exam {
    public static void main(String[] args) {
        Runnable runnable = () -> {
            for (int i=0 ; i < 10 ; i++) {
                System.out.println(i);
            }
        };

        Thread thread = new Thread(runnable);
        thread.start();
    }
}
```
Thread 생성자를 호출할 때 아래와 같이 구현하여도 된다.
```java
Thread thread = new Thread(() -> {
    for (int i=0 ; i < 10 ; i++) {
        System.out.println(i);
    }
});
```
자바 8 부터 빈번하게 사용되는 Functional Interface는 java.util.function 패키지로 제공하며 크게 **Consumer, Supplier, Function, Operator, Predicate** 로 구분 된다.


##Consumer 함수적 인터페이스##
Consumer 함수적 인터페이스의 특징은 **리턴값이 없는 accept() 메소드**이다. accept() 메소드는 단지 매개변수 값을 소비하는 역할만 한다.(사용만 할 뿐 리턴값이 없다.)
매개 변수의 타입과 수에 따라 아래와 같은 Consumer가 존재한다.

|인터페이스 명 | 추상 메소드 | 설명 |
|:--------|:--------|:--------|
| Consumer<T> | void accept(T t) | 객체 T를 받아 소비|
| BiConsumer<T> | void accept(T t, U u) | 객체 T와 U를 받아 소비|
| DoubleConsumer | void accept(double value) | double 값을 받아 소비|
| IntConsumer | void accept(int value) | int 값을 받아 소비|
| LongConsumer | void accept(long value) | long 값을 받아 소비|
| ObjDoubleConsumer<T> | void accept(T t, double value) | 객체 T와 double값을 받아 소비|
| ObjIntConsumer<T> | void accept(T t, int value) | 객체 T와 int값을 받아 소비|
| ObjLongConsumer<T> | void accept(T t, long value) | 객체 T와 long값을 받아 소비|

Consumer<T> 인터페이스를 타겟 타입으로 하는 람다식은 아래와 같이 작성 한다. accept() 메소드는 매개변수가 T객체 하나이므로 람다식도 한개의 매개변수를 사용한다. 타입 파라미터 T에 String이 대입되어 t의 매개변수 타입은 String이 된다.

```java
Consumer<String> consumer = t -> {t를 소비하는 실행문};
```
```java
Consumer<String> consumer = t -> System.out.print(t + "8");
consumer.accept("자바 ");
```

http://palpit.tistory.com/673


```bash
1. Oracle Instant Client가 필요한 필수 라이브러리 설치

2. Oracle Instant Client 설치

3. Oracle 에서 사용하는 path 정보 추가

4. python의 easy_install을 사용하여 cx_Oracle 설치
```  

#### **Oracle Instant Client의 필수 라이브러리 설치**
* yum -y install libaio bc flex




| 기능 | IntelliJ | Eclipse |
|:--------|:--------|:--------|
| **Navigating** |  | |
| Go to **File** | `Ctrl` + `Alt` + `N` |  |
| Go to **Class** | `Ctrl` + `N` | |
| Go into **method** | `Ctrl` + `Alt` + `B` | |
| Navigate to **Line** | `Ctrl` + `G` | |
| Navigate to **Declaration** | `Ctrl` + `B` | |
| Navigate to **Implementation** | `Ctrl` + `Alt` + `B` | |
| Navigate to **Usage** | `Alt` + `F7` | |
| Navigate to **Super Method** | `Ctrl` + `U` | |
| Navigate to **Previous Method** | `Alt` + `Up` | |
| Navigate to **Next Method** | `Alt` + `Down` | |
| Navigate to **Opening Brace** | `Ctrl` + `[` | |
| Navigate to **Closing Brace** | `Ctrl` + `]` | |
| **Backward** | `Ctrl` + `Alt` + `Left` | |
| **Forward** | `Ctrl` + `Alt` + `Right` | |
| Last Edit Location | `Ctrl` + `Alt` + `Up` | |
| Navigate to **Next Highlighted Error** | `F2` | |
| Navigate to **Previous Highlighted Error** | `Shift` + `F2` | |
| Jump to Source | `F4` | |
| **Selecting Text** | | |
| Select Extension | `Ctrl` + `W` | |
| **View** | | |
| View class Hierarchy | `Ctrl` + `H` | |
| View call Hierarchy | `Ctrl` + `Alt` + `H` | |
| View Current File Location in Explorer View | `Alt` + `F1` | |
| **Generating** | | |
| **Code Generating** | `Alt` + `Enter` | |
| **Code Completion** | `Ctrl` + `Space` | |
| Find | `Ctrl` + `F` | |
| Replace  | `Ctrl` + F -> `Ctrl` + `R` | |
| **Find in Path** | `Ctrl` + `Shift` + `F` | |
| **Refactoring** | | |
| Extract **Constant** | `Ctrl` + `Alt` + `C` | |
| Extract **Variable** | `Ctrl` + `Alt` + `V` | |
| Extract **Method** | `Ctrl` + `Alt` + `M` | |
| Variable **Rename** | `Shift` + `F6` | |
| Inline | `Ctrl` + `Alt` + `N` | |
| Delete Line | `Ctrl` + `Y` | |
| Move Statement UP | `Alt` + `Shift` + `↑` | |
| Move Statement DOWN | `Alt` + `Shift` + `↓` | |
| Toggle **lower or Upper** Case | `Ctrl` + `Shift` + `U` | |
| **Debugging** | | |
| Debug | `Shift` + `F9` | |
| Run | `Shift` + `F10` | |
| Step Into | `F7` | |
| Step Over | `F8` | |
| Step Out | `Shift` + `F8` | |
| Resume | `F9` | |
| **Etc** | | |
| **Auto format** | `Ctrl` + `Alt` + `L` | |
| **Optimizing Import** | `Ctrl` + `Alt` + `O` | |
| **Viewing all breakpoints** | `Ctrl` + `Shift` + `F8` | |
| **Add Bookmark** | 'F11` | |
| **Show Bookmark** | `Shift` + `F11` | |


용어 비교

|IntelliJ | Eclipse |
|:--------|:--------|
| Project | Workspace |
| Module | Project |
| Module JDK | Project-specific JRE |
| Grobal library | User library |
| Path variable | Classpath variable |
| Module dependency | Project dependency |
| Module library | Library |

