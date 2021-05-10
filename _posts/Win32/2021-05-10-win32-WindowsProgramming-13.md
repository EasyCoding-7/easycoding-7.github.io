---
layout: post
title:  "(Win32 : WindowsProgramming-13) 가상주소 공간과 DLL"
summary: ""
author: win32
date: '2021-05-10 0:00:00 +0000'
category: ['win32', 'WindowsProgramming']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/thumbnail-win32-api.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/win32/WindowsProgramming-13/
---

## Virtual Address(가상주소)란?

프로그램은 실제로 메모리 공간에 올라가지만, 프로그램이 실제 물리적 메모리 주소를 사용한다면?<br>
프로그래머의 실수로 잘못된 메모리 공간에 직접 접근시 자신의 프로그램 뿐만 아니라 잘못 사용된 메모리공간에 있는 다른 프로그램 혹으 OS마져 오류를 이르킬수 있다.

해결책?<br>
Page Table을 사용하여 실제 메모리가 올라간 **물리주소**와 **가상주소**를 매핑시켜 프로그램은 가상의 주소만 사용하게 하자.

이런 가상주소로 하나의 프로세스는 다른 프로세스가 사용하는 메모리에 접근할 수 없게 된다.

---

## 가상주소의 메모리 공간

32bit 프로그램을 기준 총 사용가능한 메모리 공간은 4G<br>
상위 2G는 OS가 사용하는 메모리<br>
하위 2G는 실행파일, DLL, 스택, 힙이 올라가는 메모리

---

## A, B 프로세스에서 같은 DLL을 사용시 DLL은 메모리에 몇번 로드될까?

DLL은 물리 메모리에 한 번만 로드되고, 각 가상주소를 통해 접근한다.<br>
DLL로 공유 메모리 접근이 가능해진다.