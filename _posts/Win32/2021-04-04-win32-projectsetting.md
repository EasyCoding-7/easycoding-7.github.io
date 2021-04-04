---
layout: post
title:  "(Win32) ProjectSetting"
summary: ""
author: win32
date: '2021-04-04 0:00:00 +0000'
category: ['win32']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/projectsetting/
---

* [Get Code](https://github.com/EasyCoding-7/win32-example/tree/master/1)

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