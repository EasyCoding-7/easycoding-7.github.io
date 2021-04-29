---
layout: post
title:  "(Unreal : Basic) 18. Collision-2"
summary: ""
author: Unreal
date: '2021-04-29 0:00:00 +0000'
category: ['Unreal']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/Basic-18/
---

## Blueprint를 통해서 시작과 동시에 x축으로 힘을 받게 만들어보자

![](/assets/img/posts/Unreal/Basic-18-1.PNG){:class="img-fluid"}

테스트를 해보면 시작과 동시에 X축으로 10000000만큼의 힘을 받기에 튕겨져 나간다.

마치 중력이 작용한것과 같은 동작이다.

---

좀 더 극적인 시험을 위해 바닥을 엄청 넓게 하고<br>
앞에 의자를 둬서 Collision하게 만들어보자.

![](/assets/img/posts/Unreal/Basic-18-2.PNG){:class="img-fluid"}
