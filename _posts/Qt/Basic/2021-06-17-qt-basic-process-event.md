---
layout: post
title:  "(Qt : basic) main event queue에 쌓인 event 강제로 처리"
summary: ""
author: Qt
date: '2021-06-17 0:00:00 +0000'
category: ['Qt']
#tags: ['Qt', 'tag1-test1']
thumbnail: /assets/img/posts/qt-thumnail.png
#keywords: ['Qt 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Qt/basic/main-event-process/
---

```cpp
QCoreApplication::processEvents();
// 간단해서 별도 정리 없음.

// 사용은 QListWidget에 Item넣을때 Item넣을때마다 호출해주는게 좋음.
// Item이 많을경우 GUI가 멈추게 된다.
```