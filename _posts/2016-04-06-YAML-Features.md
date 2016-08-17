---
layout:            post
title:             "YAML 커스텀 특징"
menutitle:         "YAML 특징"
date:              2016-04-06 00:40:00 +0300
tags:              Jekyll YAML 특징 설명
category:          Features
author:            eceris
cover:             /assets/mountain-alternative-cover.jpg
published:         true
redirect_from:     "/YAML-Features-Redirect/"
cover:             /assets/mountain-alternative-cover.jpg
language:          ko
---

이번 포스트에서 프론트 페이지에 보이는  `YAML 커스텀 특징` to `YAML 특징` 의 제목을 변경했다.

```bash
---
title:             "YAML 커스텀 특징"
menutitle:         "YAML 특징"
---
```

또한, 이번 포스트로 들어오는 리다이렉트를 추가하였다. 만약에 [YAML-Feature-Redirect]({{ site.github.url }}/YAML-Features-Redirect) 으로 들아간다면, 무조건 이 페이지로 리다이엑트 될 것이다.(플러그인을 삭제한 적이 없다면, 무조건 동작함.)
I also added a redirect to this post. If you browse to [YAML-Feature-Redirect]({{ site.github.url }}/YAML-Features-Redirect) you should be redirected to this page. This only works if you have not removed the plugins.

```bash
---
redirect_from:     "/YAML-Features-Redirect"
---
```

만약 여러개의 소스로 리다이렉트 하고 싶다면, 배열 형태로 아래와 같이 정의한다.

```bash
---
redirect_from:  
  - "/YAML-Features-Redirect/"
  - "/blog/old-category/YAML-Features/"
---
``````
또한, YAML을 통해 이번 블록의 커버이미지를 변경하였다.

```bash
---
cover:         /assets/mountain-alternative-cover.jpg
---
``````

검색엔진의 키워드를 생성하기 위해, 포스트에 정의된 태그를 사용한다. 만약 페이지를 작성한다면, 태그 대신에 키워드를 정의해야 한다.

```bash
---        
tags:          Jekyll YAML Features Explained  #Only used in posts!
keywords:      Jekyll YAML Features Explained  #Only used in pages!
---
```
포스트는 카테고리별로 프론트 페이지에 정렬되어 있을 것이다. 아래가 바로 어떻게 카테고리를 YAML에 정의해야 하는지를 알려준다.

```bash
---        
category:     Readme
---
```
페이지에는 여러 옵션이 존재한다. 예를 들면, 페이지를 `visible` 태그를 사용하여 메뉴에서 숨길 수 있다.

```bash
---        
visible:       false     
---
```

만약 페이지를 메뉴에 넣고 싶다면 `weight` 태그를 페이지에 추가하여 순서를 지정할 수 있다.


```bash
---        
weight:       5  
---
```

기본 언어 설정은 `_config.yml` 파일에 정의되어 있지만, 포스트나 페이지에 다른 언어 설정을 지정하고 싶다면 `language` 속성을 사용하면 된다. 여기를 [language codes](http://www.w3schools.com/tags/ref_language_codes.asp)  참고하기 바란다. 

```bash
---        
language:       en  
---
```

추가로 [YAML Front Matter documentation](https://jekyllrb.com/docs/frontmatter). 에서도 여러 설정을 찾을 수 있다.
