---
layout: post
title: "[Spring]사내 채팅 서비스 설계"
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

스프링 기본 구조는 이렇다.
내가 왔다갔다 하면서 확인하기 위해 남겨둔다.

1. Controller
- 화면(프론트)에서 요청 받는다.
- Request DTO 나 파라미터를 Service 전달한다.
</br>
2. Request DTO
- 화면에서 넘어온 데이터를 담는 객체
- 조회조건, 등록정보 등을 한 번에 전달하기 위해 사용 (하나 이상??)
</br>
3. Service (? 아직 모르겠음)
- 업무로직 처리
- 검증
- 권한 확인
- 여러 Respository 호출
- Entity -> 
</br>
4. Respository
- DB 접근 담당
</BR>
---------------- DB ----------------
</BR>
6. Entity
- DB 테이블과 매핑되는 객체
- 조회 결과 담음
</BR>
7. Response DTO
- 화면에서 내려줄 데이터만 담는 객체
- Entity 전체를 노출하지 않기 위해 사용
</br>
8. Controller
- Response DTO를 JSON 형태로 반환
</BR>
9. JSON
- 화면(프론트)로 전달