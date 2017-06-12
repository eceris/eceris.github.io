---
layout:            post
title:             "AsyncTask 사용하기"
date:              2017-06-08 09:50:00 +0300
tags:              Android, asynchronous task
category:          Android
author:            eceris
---

# AsyncTask 사용하기 #

안드로이드는 기본적으로 싱글 스레드 모델로 작동하므로, 이것을 고려하여 개발하지 않으면 애플리케이션의 성능에 많은 영향을 미친다. 따라서 여러가지 방식을 이용해 작업을 분할하여 수행하는데, 안드로이드에서 기본적으로 제공하는 것이 바로 AsyncTask이다.

AsyncTask는 결과값을 UI Thread에서 사용하기에 편리하고 쉽게 해주는 클래스이다. 내부적으로는 Thread간의 복잡한 메시지 핸들링과 Thread 생성등을 핸들링하는 몇개의 runnable로 구성 되어있는데, Thread나 Handler에 대한 이해 없이도 사용할 수 있도록 잘 추상화 되어있다. 그만큼 AsyncTask는 UI Thread(메인 Thread)를 사용하는 쉽고 좋은 방법이다. Thread를 핸들링해야하는 스트레스 없이 백그라운드에서 동작하고 결과를 UI스레드에 반환한다.

사실, AsyncTask는 스레드와 핸들러와 같은 Generic Threading 프레임워크의 헬퍼 클래스로 디자인 되었기 때문에 보통 몇 초안에 끝나는 작업들에 사용것이 좋다. 만약 수십 초 혹은 수 분동안 수행되어야 하는 작업이라면 java.uitl.concurrent 패키지의 다양한 API를 사용하기를 추천한다.(Executor, ThreadPoolExcutor, FutureTask...)

아래의 AsyncTask 예제를 보자.

```java

import android.os.AsyncTask;

public class TestAsyncTask extends AsyncTask<String, Void, String> {

    public String result;

    @Override
    protected void onPreExecute() {
        super.onPreExecute();
    }

    @Override
    protected String doInBackground(String... params) {
        return result;
    }

    @Override
    protected void onProgressUpdate(Void... values) {
        super.onProgressUpdate(values);
    }

    @Override
    protected void onPostExecute(String s) {
        super.onPostExecute(s);
    }
}

```

AsyncTask는 abstract로 작성되어 있기 때문에 상속받아서 사용해야한다.(혹은 익명클래스로 사용하던지...)

```java
public class TestAsyncTask extends AsyncTask<String, Void, String>
```
제네릭 인자를 3개 정해야 하는데 첫번째 인자는 doInBackground 메서드에 선언하는 가변 매개변수 타입이다. 두번째 인자는 onProgressUpdate 메서드에 선언하는 가변 매개변수의 타입이다. 세번째 인자는 onPostExecute 메서드에 선언하는 매개변수 타입이다.

```java
@Override
protected void onPreExecute() {
    super.onPreExecute();
}
```

이름에서 볼수 있듯이 background 스레드를 실행하기 전 수행된다.

```java
@Override
protected String doInBackground(String... params) {
    return result;
}
```

background로 수행되는 곳이다. UI Thread와는 별개로 동작한다.(비동기)

```java
@Override
protected void onProgressUpdate(Void... values) {
    super.onProgressUpdate(values);
}
```

doInBackground 메서드에서 중간중간 UI Thread에 일을 시킬 경우 사용한다. 매개변수로 Void를 받으므로, doInBackGround안에 실제 인자없이, publishProgress() 메서드를 호출하면 중간 중간 UI Tread에 일을 시킬수 있다.

```java
@Override
protected void onPostExecute(String s) {
    super.onPostExecute(s);
}
```

background Thread가 완료된 후 수행된다. 보통 매개변수로 들어온 s 값을 UI의 변경에 사용한다.

위의 AsyncTask를 사용하기 위해 아래와 같이 작성한다.

```java
TestAsyncTask task = new TestAsyncTask();
task.execute();
```
execute의 아규먼트는 doInBackground에서 받는 String... 가변인수이다. 필요할 경우 매개변수를 작성하여 사용하면 된다.