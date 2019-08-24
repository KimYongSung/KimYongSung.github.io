---
layout: post
title: "GC와 OS SWAP Memory"
date: 2019-08-20 23:26:40
image: '../assets/img/20190820/garbage_collection.png'
description: SWAP 메모리 사용시 GC 시간 증가 케이스
category: 'troubleshooting'
tags:
- troubleshooting
introduction: SWAP 메모리 와 GC
---

때는 저번 달 초. 갑자기 사내 인프라팀에서 연락이 왔습니다.

제가 담당하고 있던 서비스의 TOMCAT이 내려갔다고 확인 요청이 왔었죠.

우선 급하게 서버로 접속 후 프로세스 확인 및 로그를 확인하게 되었습니다.

하지만 살아있는 프로세스와 정상적으로 처리 중인 로그만 확인 할 수 있었죠.

기존에도 몇번의 오탐이 있었던지라 처음에는 대수롭지 않게 넘겼지만, 어느순간 부터 지속적으로 발생하게 됬었습니다.

서비스에는 이상은 없었지만, 지속적으로 발생하는 이유를 파악하기 위해 로그를 분석하게 되었습니다.

로그를 확인해보니 특정 시간동안 서비스가 멈춰있는걸 확인했습니다.

저는 우선 Full GC를 의심했습니다.

# Heap dump 를 떠보자!!

Full GC 가 왜 발생했고 어떤 부분에서 메모리를 사용하고 있는지 확인하기 위해 우선 heap dump를 확인해봤습니다.

eclipse memory analyzer를 사용하여 분석하였지만 특이사항을 찾을 수 없었죠..

![](https://www.eclipse.org/mat/about/overview.png)

( 뭐가 문제일까?! )

# GC 를 확인해보자!!

우선 GC가 언제 발생하고 시간이 정확히 얼마나 소요되는지 확인할 필요성이 느껴졌습니다.

기존 서비스 중인 tomcat에 GC 로그 설정을 추가 후 다시 확인해 보았습니다.

재기동 이후 일정시간 이후부터 minor GC 시간이 증가되기 시작하였고, minor GC만 5초 이상 소요되는걸 확인할 수 있었습니다.

Full GC 는 무려 1분 30초가.. 

메모리 상에 특이사항도 없는데 왜 GC가 오래걸리는건가 확인이 필요했죠. 

# SWAP Memory 와 GC

Head dump에서는 특이사항을 확인 할 수 없었고.. 그렇다면 뭐가 문제일까 고미하다가 구글링을 하게되었습니다. ( 갓구글 짱짱 )

그러다가 발견한 저와 유사한 케이스에 대한 질문을 확인 할 수 있었죠.

<https://stackoverflow.com/questions/32652585/why-this-particular-minor-gc-was-so-slow>

해당 답변을 보면, 아래와 같습니다.

>Sometimes the OS activities such as the swap space or networking activity happening at the time when GC is taking place can make the GC pauses last much longer. These pauses can be of the order of few seconds to some minutes.
>
>If your system is configured to use swap space, Operating System may move inactive pages of memory of the JVM process to the swap space, to free up memory for the currently active process which may be the same process or a different process on the system. Swapping is very expensive as it requires disk accesses which are much slower as compared to the physical memory access. So, if during a garbage collection the system needs to perform swapping, the GC would seem to run for a very long time.

스왑 공간이나 네트워크 활동과 같은 OS 활동으로 GC가 더 오래 지속될 수 있다는 내용을 확인했습니다.

해당 서비스의 경우 사용자가 얼마 없고, 노후화된 장비에서 실행되고 있었습니다.

실제 메모리는 8G에 Swap memory는 무려 16g... 

급하게 메모리를 증설하였고, swap memory 비율을 조정 후 해당 현상은 발생하지 않았습니다.

만약 위와 비슷한 케이스가 발생시 swap memory 사용을 한번 확인해보세요. 

-----












