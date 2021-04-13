---
layout: post
title:  "(DirectX : Basic) 17.(수학) World, View 변환 행렬"
summary: ""
author: DirectX
date: '2021-04-13 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-17/
---

좌표계는 이런식으로 흘러간다

Local -> World -> View -> Projection -> Screen

* Local : 각 3D 오브젝트 하나하나의 좌표계
* World : 전체 월드에 오브젝트를 두고 그리는 좌표계(각 오브젝트에 월드행렬을 곱하면 자신의 위치가 나타난다)
* View : 카메라의 위치를 기준으로 찍히는 화면을 말함(뷰 행렬을 곱하면 나타남)
* Projection : 카메라가 찍는 영역을 표현
* Screen : 카메라에 나타난 그림을 투영시킨 화면

![](/assets/img/posts/directx/basic-17-1.png){:class="img-fluid"}

이래선 영... 뭔소린지...<br>
하나 하나 살펴보면 쉽다

## Local <-> World

![](/assets/img/posts/directx/basic-17-2.png){:class="img-fluid"}

---

## World <-> View

![](/assets/img/posts/directx/basic-17-3.png){:class="img-fluid"}

![](/assets/img/posts/directx/basic-17-4.png){:class="img-fluid"}

![](/assets/img/posts/directx/basic-17-5.png){:class="img-fluid"}

