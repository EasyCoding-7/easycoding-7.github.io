---
layout: post
title:  "(DirectX : Basic) 16.(수학) 행렬(Scale, Rotation, Translation) 변환"
summary: ""
author: DirectX
date: '2021-04-12 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-16/
---

수학적 내용, 역시 아래 공식정도만 알고 있으면 된다.

Scale(사이즈), Rotation(회전), Translation(이동)에 대해 설명

## Translation

우선 Translation에 대해 설명한다.

예를들어 3차원 (0, 0, 0)에 한 물체가 있는데 (10, 15, 20)으로 이동을 하려 한다고 하자<br>
간단하게 (x, y, z)에 (10, 15, 20)만 더하면 된다고 하지만, 만약 특정 포털(매트릭스)에 들어가면 이동하게 만들고자 한다면? v*M 벡터와 매트릭스의 곱이 된다.

![](/assets/img/posts/directx/basic-16-1.png){:class="img-fluid"}

벡터와 매트릭스의 곱에선 (x, y, z)각 성분에 원하는 값을 각각 더하게 할 순 없다<br>
그럼 어떤방식으로 Translation을 구현해야 할까?

4차원(동차좌표계)으로 구현해보자.

![](/assets/img/posts/directx/basic-16-2.png){:class="img-fluid"}

4차원 좌표계의 특징은 마지막 항(w)을 1로 둘 경우 m41, m42, m43에 x, y, z값을 표현할 수 있다.

![](/assets/img/posts/directx/basic-16-3.png){:class="img-fluid"}

행렬의 m41, m42, m43에 translation 값을 넣으면 될꺼 같은데?

![](/assets/img/posts/directx/basic-16-4.png){:class="img-fluid"}

그리고 자신의 항의 값만 남겨두면 되니 단위행렬로 만든다.

![](/assets/img/posts/directx/basic-16-5.png){:class="img-fluid"}

Translation 행렬이 완성이 된다.

---

## Scale

(x, y, z)만큼의 배율로 커지게 만들고 싶다면?

이건 쉽기에 완성본만 보자

![](/assets/img/posts/directx/basic-16-6.png){:class="img-fluid"}

---

## Rotation

z축을 회전한다고 가정해보자.<br>
z축이 고정되어 x, y만 있다고 가정하고 A를 B로 옮긴다고 생각해보자.<br>
B의 X, Y는 아래와 같이 구할 수 있다.

![](/assets/img/posts/directx/basic-16-7.png){:class="img-fluid"}

![](/assets/img/posts/directx/basic-16-8.png){:class="img-fluid"}

좀더 공식을 단순화 해보자면

![](/assets/img/posts/directx/basic-16-9.png){:class="img-fluid"}

이걸 매트릭스에 반영하자면

![](/assets/img/posts/directx/basic-16-10.png){:class="img-fluid"}

---

## 주의할점?

Scale, Rotation은 원점을 기준으로 Scale되고 Rotation되기에 주의해야한다.<br>
따라서 Scale, Rotation, Translation을 적용시 Scale, Rotation, Translation순으로 적용해야 한다.<br>
일단 원점에 두고 Scale -> Rotation -> Translation 