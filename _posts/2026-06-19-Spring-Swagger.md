---
layout: post
title: "[사내 채팅 서비스] Swagger"
subtitle: 
author: Hye min Song
categories: jekyll
banner:
  #video: https://vjs.zencdn.net/v/oceans.mp4
  loop: true
  volume: 0.8
  start_at: 8.5
  image: /assets/images/post/4_Spring_Coment.png
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: 사내채팅서비스 DB Spring DTO Entity Controller
top: 1
sidebar: []
---
![example image](/assets/images/post/5_Spring.png "작고 소듕한..")

채팅 프로젝트 API를 테스트하기 위해 Swagger를 적용해보기로 했다.

남자친구 회사에서는 툴을 이용해서 검증한다 했다.

원래는 Postman으로 호출하려고 했는데 API가 늘어날수록 Swagger가 훨씬 편하다고 해서 적용을 시작했다.

## 1. 시작부터 빨간줄

Swagger 의존성을 추가했는데 바로 Controller에서 빨간줄이 발생했다.

처음에는 의존성 버전 문제인가 싶어서 이것저것 찾아봤는데 원인은 전혀 다른 곳에 있었다.

프로젝트에 gradle-wrapper가 없었다.

Wrapper가 없는 상태에서 계속 빌드만 시도하고 있었던 것이다.

Wrapper를 설치하고 다시 Gradle Refresh를 진행하니 빨간줄은 사라졌다.

## 2. 이번에는 실행이 안 된다

이제 끝난 줄 알고 bootRun을 실행했다.

그런데 이번에는 Repository에서 오류가 발생했다.

확인해보니 Repository에서 사용하는 필드명과 Entity가 서로 맞지 않았다.

Entity 기준으로 다시 맞춰주고 재실행.

## 3. 또 다른 Entity 문제..제발 그만

이번에는 컬럼명 때문에 오류가 발생했다.

```
pinned_yn
```

으로 작성했던 부분을

```
pinnedYn
```

으로 수정해야 했다.

JPA는 Entity의 필드명을 기준으로 조회하기 때문에 DB 컬럼명이 아니라 Entity 변수명을 사용해야 했다.

## 4. EmbeddedId의 함정

수정 후 다시 실행.

또 오류 발생.

이번에는 Repository 쿼리에서 사용한 필드가 문제였다.

처음에는 아래처럼 작성되어 있었다.

```
crm.userId
```

그런데 해당 Entity는 EmbeddedId를 사용하고 있었다.

```
@EmbeddedId
private ChatRoomMemberId id;
```

구조였기 때문에 실제 접근은

```
crm.id.userId
```

로 해야 했다.

EmbeddedId 내부 필드라는 것을 놓치고 있었던 것이다.

## 5. 드디어 성공

모든 오류를 수정한 뒤 다시 실행.

Swagger UI에 정상적으로 API가 노출되었다.

현재 확인 가능한 API는

```
채팅방 조회
채팅방 고정
디버그 조회
```

정도이며 브라우저에서 바로 테스트가 가능해졌다.

## 느낀 점

Swagger 붙이는 작업 자체는 10분이면 끝날 줄 알았다.

하지만 실제로는

Gradle Wrapper 누락
Repository ↔ Entity 불일치
필드명(camelCase) 문제
EmbeddedId 접근 문제

를 하나씩 해결하면서 생각보다 오래 걸렸다.

그래도 덕분에 JPA가 Entity 필드명을 기준으로 동작한다는 점과 EmbeddedId 사용 시 필드 접근 방법을 다시 정리할 수 있었다.