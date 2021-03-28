---
layout: post
title:  "(DirectX : Tutorial) 1. Project Setting"
summary: ""
author: DirectX
date: '2021-03-28 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/tutorial-1/
---

* [DirectX Tutorial (YouTube)](https://www.youtube.com/watch?v=_4FArgOX1I4&list=PLqCJpWy5Fohd3S7ICFXwUomYW0Wv67pDD)
* [Get Project(GitHub)](https://github.com/EasyCoding-7/DirectX-basic-Tutorial)

---

## Get Code

* [Link](https://github.com/EasyCoding-7/DirectX-basic-Tutorial/tree/master/1)

---

## 프로젝트생성

![](/assets/img/posts/directx/dxd-basic-1-1.png){:class="img-fluid"}

![](/assets/img/posts/directx/dxd-basic-1-2.png){:class="img-fluid"}

![](/assets/img/posts/directx/dxd-basic-1-3.png){:class="img-fluid"}

말 그대로 여러 프로세스를 이용하여 컴파일, 컴파일 속도 증대

![](/assets/img/posts/directx/dxd-basic-1-4.png){:class="img-fluid"}

exe, dll에 데이터를 올리지않고 램에 올려 속도를 초대화

![](/assets/img/posts/directx/dxd-basic-1-5.png){:class="img-fluid"}

![](/assets/img/posts/directx/dxd-basic-1-6.png){:class="img-fluid"}

* /mt : 윈도우 dll을 내장
* /md : dll이 필요 (빠르다)

![](/assets/img/posts/directx/dxd-basic-1-7.png){:class="img-fluid"}

![](/assets/img/posts/directx/dxd-basic-1-8.png){:class="img-fluid"}

부동소수점의 계산을 어떻게 할지(정확하게 혹은 대충)

![](/assets/img/posts/directx/dxd-basic-1-9.png){:class="img-fluid"}

![](/assets/img/posts/directx/dxd-basic-1-10.png){:class="img-fluid"}

```cpp
/*
// Error
int main()
{
	return 0;
}
*/

#include <Windows.h>

int CALLBACK WinMain(
	HINSTANCE hInstance,
	HINSTANCE hPrevInstance,
	LPSTR lpCmdLine,
	int nCmdShow)
{
	while (true);

	return 0;
}
```

여기까지하면 프로세스생성까지 확인가능