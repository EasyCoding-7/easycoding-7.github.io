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

## 좌표계 변환

```
(y)
 | 
 |     M(x, y)
 |
 |
 |
 |------------------- (x)
 A
```

원점을 A라 가정하고 x, y축의 단위 백터를 `u1, v1`이라 가정하자<br>
AM의 벡터를 표현 하는 방법은 `AM = x*u1 + y*v1` 이라 표현할 수 있다.

여기서 문제는 원점을 다시 표현하고 싶을때 (가령 카메라의 좌표, 각도가 변경되었을 경우)<br>
상대적으로 M의 좌표를 어떻게 표현할꺼냐에 대한 문제이다.<br>
(우리가 갖고있는 값은 M의 좌표 및 새로운 원점의 각도 정도이다.)

좀 정리하자면 우린 `AM = x*u1 + y*v1`을 알고 있고<br>
`BM`을 구하고 싶은데 `x, u1, y, v1`을 통해 표현해야한다.

```
(y)
 | 
 |     M(x, y)\   /
 |             \ /
 |              B
 |
 |------------------- (x)
 A
```

B가 새로운 원점좌표이고 B를 기준으로 M을 표현(`BM`)하고자 한다면??<br>
`BM = x2*u2 + y2*v2` 여기서 `(x2, y2)`는 B에서 M을 봤을때 상대적 위치 `u2, v2`는 B원점좌표의 단위 벡터이다.

```
// u1, v1 단위 벡터를 u2, v2로 표현이 가능하다
// ux, uy, vx, vy는 임의의 값이라 생각하자
u1 = ux*u2 + uy*v2  // u1을 u2, v2 성분의 합으로 표현이 가능
v1 = vx*u2 + vy*v2
```

```
// 일단 하려는게 u1, v1으로 표현된 AM을 u2, v2로 표현을 변경하려는게 목적이다.
AM = x*u1 + y*v1
AM = x*(ux*u2+uy*v2) + y*(vx*u2 + vy*v2)
AM = u2*(x*ux+y*vx) + v2*(x*uy+y*vy)
// 여기까지하면 AM을 u2, v2로 표현이 가능해진다.
```

```
// 벡터상으로 볼때
AM = AB + BM
// 우리가 알고 싶은 것은 BM이기에
BM = AM + BA
// 라고 표현이 가능해진다.
// AM = u2*(x*ux+y*vx) + v2*(x*uy+y*vy) 라고 위에서 증명했기에
BM = u2*(x*ux+y*vx) + v2*(x*uy+y*vy) + BA
// 라고 도출이 된다.
```

`BA`또한 B를 기준으로 A를 바라보는 벡터이기에

```
BA = qx*u2 + qy*v2
// 라고 표현해보자면
BM = u2*(x*ux+y*vx) + v2*(x*uy+y*vy) + qx*u2 + qy*v2
// 좀 정리하자면
BM = u2*(x*ux+y*vx*qx) + v2*(x*uy+y*vy*qy)
// 잘 보면 x 곱하기 ux
// y 곱하기 vx
// qx는 더하기
// 느낌상 z는 wx일듯?
```

정리하면 아래와 같은 Matrix가 도출된다.

```
ux uy uz 0
vx vy vz 0
wx wy wz 0
qx qy qz 0
```

즉, 내 좌표계 `[x, y, z, 1]`에 Matrix를 곱할경우 변경된 좌표계에서 내 위치(x, y, z)를 구할 수 있다.<br>
그럼 이제 Matrix구하는 방법만 알면되겠군?? (맞다. 사실 Matrix구하는 방법만 알면되고 위의 설명은 이런식으로 좌표계 변환이 이루어진다고 알려주는 것이다.)

---

## 3D Game Engine에서 사용되는 좌표계 종류

* Local : 각 3D 오브젝트 하나하나의 좌표계
* World : 전체 월드에 오브젝트를 두고 그리는 좌표계(각 오브젝트에 월드행렬을 곱하면 자신의 위치가 나타난다)
* View : 카메라의 위치를 기준으로 찍히는 화면의 좌표계
* Projection(투영) : 카메라에 의해 나타날 부분/나타나지 않을 부분, 작게/크게 표현할 부분을 담당하는 좌표계
* Screen : 실제로 모니터에 그려질 부분의 좌표계

3D Game Engine은 보통 위의 5개의 좌표계를 쓰며 서로서로간 좌표계 변환이 필요하다<br>
사실 이게 설명으로 보면 이해가 어려움. 오히려 수식이 쉽다

---

일단 좌표계의 각 축 이름을 좀 더 직관적으로 변경하겠다.

```
(up)
 |     (look)
 |     /
 |    /
 |   /
 |  /
 | /
 |------------------------- (right)
```

## Local <-> World

Local 좌표계에서 Rotation만 up축의 a도 돌리고 World 좌표계에서 (x, y, z)만큼 이동한 물체의 World 좌표계 변환을 구해보자

```
(up)
 |     (look)
 |     /
 |    /
 |   /         O(x, y, z)
 |  /
 | /
 |------------------------- (right)
```

Local 좌표계 `x, y, z`에 있던 물체를 World 좌표계로 옮기고 싶다면?

```
right.x right.y right.z 0
up.x    up.y    up.z    0
look.x  look.y  look.z  0
x       y       z       1
```

위 Matrix에서 우리가 하는 부분은 x, y, z뿐이고 right, up, look은 어떻게 구해야할까?

앞에서 배운 SRT(Scale, Rotation, Translation)가 Matrix의 right, up, look이게 된다.

---

## World <-> View

```
\              /
 \  O(물체)   /
  \          /
    (카메라)
```

위 그림처럼 보자면 카메라가 오른쪽으로 돌면 물체는 왼쪽으로 간다고 생각<br>
왼쪽으로 돌면 물체는 오른쪽으로 간다고 생각 할 수 있다.<br>

이게 View 좌표계를 구하는 힌트이다.

카메라의 World Matrix는 아래와 같고

```
right.x right.y right.z 0
up.x    up.y    up.z    0
look.x  look.y  look.z  0
x       y       z       1
```

사실상 반대로 동작해야하니 역행렬을 구하면 된다.<br>
`SRT`가 카메라의 World Matrix이고 카메라는 S(Scale)이 없기에 `RT`가 World Matrix가 된다.<br>
결국 `RT`의 역행렬을 구하면 되고 `(RT)^-1 = T^-1*R^-1`<br>
right, up, look은 각 축이기에 직교성이 역행렬이 단순하다

```
// T^-1
1  0  0  0 
0  1  0  0
0  0  1  0
-x -y -z 1

// 이렇게도 표현이 가능
1    0    0    0 
0    1    0    0
0    0    1    0
-c.x -c.y -c.z 1
```

c(camera) vector이고 c.x는 c의 x축을 의미함 기존의 x와 완전동일

```
// R^-1
right.x up.x look.x 0
right.y up.y look.y 0
right.z up.z look.z 0
0      0     0      1
```

```
// T^-1*R^-1
right.x  up.x  look.x  0
right.y  up.y  look.y  0
right.z  up.z  look.z  0
-c*right -c*up -c*look 1
```