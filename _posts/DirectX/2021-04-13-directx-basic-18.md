---
layout: post
title:  "(DirectX : Basic) 18.(수학) Projection, Screen 변환 행렬"
summary: ""
author: DirectX
date: '2021-04-13 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-18/
---

## View <-> Projection

* Projection 좌표계 : 멀리있는 아이템을 작게 가까이 있는 아이템을 크게 등등 이런 작업을 할 좌표계

```
  (y)
   | 
   |     /
   |    /
   |   /             o(x, y, z)
   |  /
   | /
 <카메라>--(1)----(n)----(f)--------- (z)
   | \
   |  \
   |   \
   |    \
   |     \
   |   
```

카메라의 각도가 90도이고, n~f사이에 있는 아이템만 그려준다고 가정하자

우선, Projection의 핵심은 카메라와의 거리에 따라 얼마나 작게/크게 그려질것인가를 계산하는데 있다.<br>
비례식을 통해서 구해야한다.

```
// 구하고자 하는 X, Y는 아래와 같이 표현할 수 있다.
// z는 깊이 값이다.
X = (1/r) * (x/z) * (1/tan(a/2))
Y = (y/z) * (1/tan(a/2))
```

* `(x/z)` : 기존 x에 깊이값 z만큼을 반영해야하고
* `1/tan(a/2)` : 카메라 각도만큼도 반영해 줘야하며
* `(1/r)` : 화면의 비율또한 반영해 줘야한다.

```
// 참고1 : 화면의 비율
r(화면의 비율)=w(width)/h(height)
```

```
// 참고2 : 높이를 tan로 구할수 있음.

    /|
   / |
  /  tan(a/2)
 / a | 
/----|

```

우선 깊이를 고려하지 않고 식을 간단히 해보자면

```
1/(r*tan(a/2)) 0            0 0
0              1/(tan(a/2)) 0 0
0              0            1 1 // z를 보존하기 위해서 여기 1을 넣음
0              0            0 0
```

이 Matrix를 사용시

`[1/(r*tan(a/2)), 1/(tan(a/2)), z, z]` 를 구할수 있는데 세 번째 z값은 기존의 깊이값인 z가 필요한 것이아니라 n~f사이의 깊이값 z가 필요하다<br>
Matrix를 좀 더 수정해야 한다.

```
1/(r*tan(a/2)) 0            0 0
0              1/(tan(a/2)) 0 0
0              0            A 1
0              0            B 0
```

`[1/(r*tan(a/2)), 1/(tan(a/2)), Az+B, z]` 이 값이 나오면<br>
마지막 z로 앞의 세 항을 나눈다 <br>
`[1/(r*z*tan(a/2)), 1/(z*tan(a/2)), A+(B/z)]`

마지막으로 A, B만 구하면 된다.

```
// n(near)일때 0이고
// f(far)일때 1이기에
G(n) = A + (B/n) = 0
G(f) = A + (B/f) = 1

// ...

B = -An
A-(An/f) = 1

// ...

A = f/(f-n)
B = -nf/(f-n)
```

완성

```
1/(r*tan(a/2)) 0            0         0
0              1/(tan(a/2)) 0         0
0              0            f/(f-n)   1
0              0            -nf/(f-n) 0
```

---

## Projection <-> Screen

Projection은 -1~1의 값을 갖는데 그 값을 실제 Screen(Viewport)의 Width, Height에 맞춰 주면 끝이다.

```
(left, top)
      -------------------------
      |                        |
      |                        |
      |                        |
      |        ViewPort        | h(height)
      |                        |
      |                        |
      |                        |
      |                        |
      -------------------------
              w(width)

X = Left + ((x+1)/2) * w
Y = TOP + ((1-y)/2) * h
```

```
w/2      0       0 0
0        -h/2    0 0
0        0       0 0
w/2+left h/2+top 0 1
```

여러 ViewPort를 사용할 수 있어서 Viewport자체의 깊이 값도 생각해야 한다.(많이 쓰진 않음)

```
w/2      0       0       0
0        -h/2    0       0
0        0       max-min 0
w/2+left h/2+top min     1
```

