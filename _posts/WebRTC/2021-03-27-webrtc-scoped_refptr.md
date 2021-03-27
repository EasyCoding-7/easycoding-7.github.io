---
layout: post
title:  "(WebRTC) scoped_refptr"
summary: ""
author: WebRTC
date: '2021-03-27 0:00:00 +0000'
category: ['WebRTC']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
keywords: ['rtc::scoped_refptr']
usemathjax: true
permalink: /blog/WebRTC/scoped-refptr/
---

* [참고사이트](https://dydtjr1128.github.io/chromium/2019/07/04/Chromium-base-scoped_refptr.html)

```cpp
rtc::scoped_refptr<webrtc::PeerConnectionInterface> peer_connection_;
rtc::scoped_refptr<webrtc::PeerConnectionFactoryInterface> peer_connection_factory_;
```

shared_ptr 레퍼런스 카운트를 make_shared를 하지 않아도 되게 변경했다고 생각하면 된다.