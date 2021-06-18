---
layout: post
title:  "(Qt : basic) worker, gui thread"
summary: ""
author: Qt
date: '2021-06-18 0:00:00 +0000'
category: ['Qt']
#tags: ['Qt', 'tag1-test1']
thumbnail: /assets/img/posts/qt-thumnail.png
#keywords: ['Qt 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Qt/basic/worker-gui-thread/
---

* [참고사이트1](https://doc.qt.io/qt-5/qcoreapplication.html#quit)
* [참고사이트2](https://forum.qt.io/topic/56134/event-loop/3)
* [참고사이트3](https://doc.qt.io/qt-5/qthread.html)

---

* Qt는 기본적으로 worker, gui Thread로 구성됨
* QThread는 내부적으로 event loop가 있다.
* worker Thread의 경우 `app.exec()`호출시 event loop가 동작하게된다.
* gui Thread에서도 내부적으로 event loop가 돌고 있기에 `app.exec()`가 호출되지 않더라도 show 및 paint가 가능하다
* 단, `QCoreApplication::quit()`과 같은 특정함수는 `app.exec()`이 호출되기 전 즉 메인 이벤트루프가 돌기전엔 동작하지 않는다.