---
layout: post
title:  "(OpenSource : WebRTC) 이론정리"
summary: ""
author: OpenSource
date: '2021-05-26 0:00:00 +0000'
category: ['OpenSource', 'WebRTC']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/opensource-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/opensource/WebRTC/theory/
---

* [참고사이트(간단)](https://millo-l.github.io/WebRTC-%EC%9D%B4%EB%A1%A0-%EC%A0%95%EB%A6%AC%ED%95%98%EA%B8%B0/)
* [참고사이트(상세)](https://brunch.co.kr/@linecard/156)

---

## WebRTC Protocol

### ICE(Interactive Connectivity Establishment)

* peer간 연결이 가능하도록 도와준다.
* 뭘 도와주나?
    * **방화벽을 통과**하는 방법
    * **public ip**가 할당되지 않았다면 ip할당
    * **라우터가 peer간 직접연결**을 할당하지 않는다면 그 후 처리
* ICE는 위 작업들을 위해서 STUN, TURN 서버 혹은 그 둘 다를 사용한다.

그럼 STURN, TURN을 알아야 겠지?

### NAT(Network Address Transilation)

* 단말에 공개 ip(public ip)를 할당하기 위해 사용
* NAT방법에는 두 가지가 존재(Cone, Symmetric)
    * Cone NAT : 송/수신을 같은 public ip로 할당 받음
    * Symmetric NAT : 송/수신이 다른 public ip로 할당될 수 있음
    * [참고사이트](http://forum.falinux.com/zbxe/?mid=network_programming&l=es&document_srl=795945)
* STUN에 등록된 ip라도 Symmetric NAT으로 모두 연결할 수 있는 것은 아니다.

```
[PC1 : 192.168.0.10] ---- [ROUTER : NAT동작] (public ip를 할당)
[PC2 : 192.168.0.20] -----/
```

### STUN(Session Traversal Utilities for NAT) 서버

* 클라이언트(자신)의 public ip(ip:port)를 알린다.
* 상대 peer의 접근 가능여부를 묻는다.

### TURN(Traversal Using Relays around NAT) 서버

* STUN으로 연결불가 판정을 받은경우 TURN으로 데이터를 릴레이
    * peer가 데이터를 TURN에게 전달후 원하는 peer에게 전송해달라고 요청하는 개념
* 오버헤드가 크기에 대안이 없을경우 사용한다.

## SDP(Session Description Protocol)

* 해상도나 형식, 코덱, 암호화 등 컨텐츠의 설명
* 정확히 말하면 프로토콜이 아님, peer간 연결을 위한 설명자료 정도이다.

---

## ICE 시나리오

```
        Alice                               Bob
          |                                  |
    <Create Offer>                           |
<Set Local Description>                      |
          |-----------(Send Offer)---------->|
          |                      <Set Remote Description>
          |                            <Create Answer>
          |                       <Set Local Description>
          |<----------(Send Answer)----------|
<Set Remote Description>                     |
          |                                  |
          |-------(Send Ice Candidates)----->|
          |<------(Send Ice Candidates)------|
          |                                  |
          |              (통신)              | 
```