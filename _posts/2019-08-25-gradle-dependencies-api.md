---
layout: post
title: "Gradle dependencies 에서 api() method를 찾을 수 없는 경우"
date: 2019-08-20 23:26:40
image: 'https://upload.wikimedia.org/wikipedia/en/e/ed/Gradle_Logo.png'
description: Gradle dependencies 에서 api() method를 찾을 수 없는 경우
category: 'build tool'
tags:
- build tool
introduction: Gradle java library plugin
---

팀 내부적으로 Gradle 대신 maven을 주로 사용하고, gradle 멀티모듈에 대해서 이야기만 들어오다가

우아한형제들의 기술블로그에서 권용근님이 작성하신 '[멀티모듈 설계 이야기 with Spring, Gradle](http://woowabros.github.io/study/2019/07/01/multi-module.html)'를 보고 토이프로젝트에 적용해보기로 했다.

Gradle 공식 문서와 여러 블로그들을 참조하여 멀티모듈 구성 중 Could not find method api() 에러를 만났다.

![](../assets/img/20190825/gradle_build_fail_log.png)

공식 사이트에 [The Java Library Plugin](https://docs.gradle.org/current/userguide/java_library_plugin.html)  확인해보니 plugin에 추가 적으로 설정이 필요하였다.

```gradle
plugins {
    'java-library'
}

or

apply plugin: 'java-library'

```

해당 설정 후 빌드시 정상적으로 성공하였다.

![](../assets/img/20190825/gradle_build_success_log.png)

-----












