---
layout: post
title:  "(Qt : Example) Frameless 분석(Window Style)"
summary: ""
author: Qt
date: '2021-06-04 0:00:00 +0000'
category: ['Qt']
#tags: ['Qt', 'tag1-test1']
thumbnail: /assets/img/posts/qt-thumnail.png
#keywords: ['Qt 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Qt/Example/frameless/
---

* [GetCode](https://github.com/EasyCoding-7/FramelessHelper)

* [참고사이트1](https://kaspyx.tistory.com/32)
* [참고사이트2](https://mindgear.tistory.com/142)

---

자세한건 코드를 살펴보면되고 중요한 개념은 윈도우 스타일을 어떻게 설정하느냐

## SetWindowLong

![](/assets/img/posts/qt/basic-frameless-1.png){:class="img-fluid"}

* `WS_CAPTION` : 제목 표시줄
* `WS_BORDER` : 단선으로 된 경계선을 만들며 크기 조정은 할 수 없다.
* `WS_THICKFRAME` : 크기 조정 가능한 두꺼운 경계선을 가진다.

더 많은데 자세한건 참고사이트 참고

---

## SetWindowPos

* `SWP_NOOWNERZORDER` : Z순서를 변경하지 않음
* `SWP_FRAMECHANGED` : SetWindowLong으로 경계선 스타일을 변경했을 경우 새 스타일을 적용, 

---

```cpp
void NativeWindowHelperPrivate::updateWindowStyle()
{
    Q_Q(NativeWindowHelper);

    Q_CHECK_PTR(window);

    HWND hWnd = reinterpret_cast<HWND>(window->winId());
    if (NULL == hWnd)
        return;
    else if (oldWindow == hWnd) {
        return;
    }
    oldWindow = hWnd;

    NativeWindowFilter::deliver(window, q);

    QOperatingSystemVersion currentVersion = QOperatingSystemVersion::current();
    if (currentVersion < QOperatingSystemVersion::Windows8) {
        return;
    }

    LONG oldStyle = WS_OVERLAPPEDWINDOW | WS_THICKFRAME | WS_CAPTION | WS_SYSMENU | WS_MAXIMIZEBOX | WS_MINIMIZEBOX;
    LONG newStyle = WS_POPUP            | WS_THICKFRAME;

    if (QtWin::isCompositionEnabled())
        newStyle |= WS_CAPTION;

    if (window->flags() & Qt::CustomizeWindowHint) {
        if (window->flags() & Qt::WindowSystemMenuHint)
            newStyle |= WS_SYSMENU;
        if (window->flags() & Qt::WindowMinimizeButtonHint)
            newStyle |= WS_MINIMIZEBOX;
        if (window->flags() & Qt::WindowMaximizeButtonHint)
            newStyle |= WS_MAXIMIZEBOX;
    } else {
        newStyle |= WS_SYSMENU | WS_MINIMIZEBOX | WS_MAXIMIZEBOX;
    }

    LONG currentStyle = GetWindowLong(hWnd, GWL_STYLE);
    SetWindowLong(hWnd, GWL_STYLE, (currentStyle & ~oldStyle) | newStyle);

    SetWindowPos(hWnd, NULL, 0, 0, 0 , 0,
                 SWP_NOOWNERZORDER | SWP_NOZORDER |
                 SWP_FRAMECHANGED | SWP_NOMOVE | SWP_NOSIZE);

    if (QtWin::isCompositionEnabled())
        QtWin::extendFrameIntoClientArea(window, 1, 1, 1, 1);
}
```