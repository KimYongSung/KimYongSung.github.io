---
layout: post
title: "java securerandom hang 발생 현상"
date: 2019-09-19 23:26:40
image: 'https://openjdk.java.net/images/openjdk.png'
description: java securerandom hang 발생 현상
category: 'java'
tags:
- java
introduction: java SecureRandom hang 발생 현상
---

jdbc를 이용하여 Oracle DB 접속시 connection reset 현상이 발생했던 경험을 기록하고자 작성합니다.

Linux 환경에서 SecureRandom 기본 구현이 NativePRNG로 속도가 매우 느립니다.

Windows에서 기본값은 SHA1PRNG로 Linux에서 명시적으로 사용할 경우 정상적으로 사용이 가능합니다만, 직접 핸들링하지 못하는 경우는 아래와 같이 설정 해보세요.

해당 hang 현상을 개선할 수 있는 설정은 크게 2가지 옵션이 있습니다.

## 1. /dev/random의 엔트로피 개선 

haveged 데몬을 설치하여 엔트로피를 지속적으로 수집하는 방법

### 1.1 우분투 / 데비안
```text
apt-get install haveged
update-rc.d haveged defaults
service haveged start
```

### 1.2 RHEL / CentOS
```text
yum install haveged
systemctl enable haveged
systemctl start haveged
```

## 2. /dev/urandom 사용
### 2.1 java 옵션 추가
```text
# java 5 미만
-Djava.security.egd=file:/dev/urandom

# java 5 이상
-Djava.security.egd=file:/dev/./urandom
```

### 2.2 java security 파일에서 수정

```text
# java 8 이하
$JAVA_HOME/jre/lib/security/java.security

# java 9 이상
 $JAVA_HOME/conf/security/java.security
```

### 참고

#### /dev/urandom 과 /dev/random 란?

>  유닉스 계열 운영 체제에서 차단 방식의 유사난수 발생기의 역할을 수행하는 특수 파일이다. 장치 드라이버와 기타 소스로부터 모은 환경적 노이즈로의 접근을 허용한다.

* 출처 [위키백과](https://ko.wikipedia.org/wiki//dev/random) 


-----












