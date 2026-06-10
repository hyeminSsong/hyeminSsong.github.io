---
layout: post
title: "[DB]사내 채팅 서비스 설계"
subtitle: 
author: Hye min Song
categories: jekyll
banner:
  #video: https://vjs.zencdn.net/v/oceans.mp4
  loop: true
  volume: 0.8
  start_at: 8.5
  image: /assets/images/post/4_DB_Modeling.png
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: 사내채팅서비스 DB PostgreSQL
top: 1
sidebar: []
---
## 1. 프로젝트 소개

> PostgreSQL 기반 사내 채팅 서비스 설계

목표
이번 프로젝트의 목표는 PostgreSQL을 사용해서 사내 채팅 서비스에 필요한 기본 DB 구조를 설계해보는 것 !

- 사용자 관리
- 채팅방 관리
- 메시지 관리
- 읽음 처리 고려
- 이모지 반응 관리

---
## 2. 최종적으로 설계한 테이블

![example image](/assets/images/post/4_DB_Modeling.png "작고 소듕한..")
---


## 3. 고민했던 부분
#### 1) 테이블과 컬럼은 소문자?

후보

```
CHAT_ROOMROOM_ID
```

vs

```
chat_roomroom_id
```

결론

```
chat_roomroom_id
```

이유
- 회사에서 사용하는 테이블은 대문자라 대문자,
- 회사에서 사용하는 컬럼은 테이블 약어를 사용해서 붙이는 방식이었음


- 하지만 PostgreSQL 관련 자료를 찾아보면서 소문자와 스네이크 케이스를 많이 사용한다는 것을 알게 되었고
- 인 프로젝트에서는 PostgreSQL에서 일반적으로 사용되는 방식에 맞춰보기로
---
#### 2) 채팅룸의 멤버 테이블이 필요한가?

후보

```
chat_room 안에 member_ids = {'u001', 'u002', 'u003'} 으로 넣는 방법
```

vs

```
chat_room_member 테이블을 따로 빼는 방법
```

결론

```
chat_room_member > 따로 빼는 방법 선택
```

이유
- chat_room 안에 member_ids = {'u001', 'u002', 'u003'} 으로 넣는 방법은
	- 특정 사용자가 들어간 채팅방 조회가 불가능함
	- 참여자별 입장일시, 퇴장일시, 알림설정, 권한 저장이 어려움
	- 채팅방 나가기/재입장/방장/관리자/안읽은메세지 등 사용자 개별로 컨트롤하기 어려움
- chat_room_member 테이블을 따로 빼는 방법은
	- 기능이 조금 늘어나도 관리하기 편할 수 있다고 판단함

---

#### 3) 코드 테이블을 사용해야 하는가 ?

후보

```
//직접저장하는 방법
status = ONLINE
status = OFFLINE
```

vs

```
//코드 테이블 관리
status_cd = 1
room_type_cd = 2
message_type_cd = 3
```


결론

```
공통 코드 테이블 만들기로 함
```

이유

- 일단 실무 시스템 구조와 유사했음
- 공통코드 테이블을 만들었을 때 코드값 변경 시 관리가 쉬웠음
- 신규 상태 추가가 용이함
- 화면 표시명과 실제 저장값을 분리할 수 있음음


---
#### 4) "마지막 메세지" 저장 테이블 ?

후보

```
chat_message 테이블에 넣는 방법
```
vs

```
chat_room 테이블에 넣는 방법
```


결론

```
chat_room 에 넣기로 함

->
//컬럼 추가 결정
last_message_id
last_sender_id
last_send_dtm
```

이유

- 마지막 메세지니까 당연히 메세지 테이블에 넣는게 맞다고 생각했으나,
- 이때 매번 `chat_message` 테이블과 조인하거나 최신 메시지를 다시 조회하는 것보다, `chat_room`에 마지막 메시지 정보를 가지고 있으면 목록 조회가 더 단순
- 채팅방 목록 조회 성능과 정렬 편의성을 고려하여 `chat_room`에 마지막 메시지 정보를 저장하기로

---
#### 5) 메시지 안읽음 처리를 위해 SEQ가 필요한가?

후보

```
message_id로만 안읽음 처리 하자
```

vs

```
message_seq 컬럼을 추가하여 사용하자
```

결론

```
message_seq 컬럼 추가하기로 함
```


이유

- 비지니스 의미가 담긴 자연키를 사용하라고 들은 적이 있기에, 많이 고민했다
- `message_seq`가 안 읽음 처리 하나만을 위한 컬럼이라면, 그 기능 하나 때문에 컬럼을 추가하는 것이 과한 설계일 수도 있다고 생각

- 하지만 `message_seq`는 안 읽음 처리뿐만 아니라 메시지 정렬에도 사용할 수 있다고 판단
- 하지만 message_seq를 사용하면 send_dtm 기준 정렬시 같은 초에 여러 메세지가 등록되어 순서가 모호해질 수 있는 것을 해결할 수 있음
- 추후 안 읽은 메시지를 조회할 때도 마지막으로 읽은 `message_seq` 이후의 메시지를 기준으로 조회 가능능

---
#### 6) 이모지 반응은 메시지 테이블에 저장하면 안 될까?

후보

```
chat_message 테이블에 아래 컬럼 추가 
like_cnt
heart_cnt
laugh_cnt
```

vs

```
chat_reaction 테이블 생성
```

결론

```
chat_reaction 테이블 생성하기로 함
```


이유

- 이모지 종류가 계속 늘어날 수 있음
- 사용자별 누가 어떤 반응을 했는지 / 취소했는지 관리하기 쉬움움

---
#### 7) 이모지 테이블에는 수정일시(UPD_DTM)가 필요할까?

후보

```
INS_DTM
UPD_DTM
둘다 사용하자
```

vs

```
INS_DTM
```

결론

```
INS_DTM 만 사용
```



이유

- 반응은 일반적으로 수정하지 않고 취소 후 재등록 하는 기능
- 수정 일시가 사용되는 곳이 없음

---
## 4. 최종

처음에는 이런 설계는 금방 끝날 줄 알았다.

하지만 막상 직접 해보니 테이블 하나, 컬럼 하나를 정하는 데도 생각보다 많은 고민이 필요했다.

이 테이블이 왜 필요한지, 나중에 기능을 구현할 때 이 컬럼이 정말 필요한지, 지금은 없어도 되지만 나중에 확장할 때 문제가 되지는 않을지 계속 생각하게 되었다.

현업 DBA 분들이 보기에는 쉽고 어설픈 설계일 수도 있다.  
하지만 지금은 직접 고민하고 만들어보는 과정 자체가 중요하다고 생각한다.

한 번에 잘 만들 수 없다면 여러 번 만들면 된다.  
100번, 1000번 만들어보면 언젠가는 지금보다 훨씬 자연스럽게 설계할 수 있을 것이라고 믿는다.