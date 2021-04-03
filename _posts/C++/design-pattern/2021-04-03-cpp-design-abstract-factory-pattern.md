---
layout: post
title:  "(C++ : Design-pattern) Abstract Factory Pattern"
summary: ""
author: C++
date: '2021-04-03 0:00:00 +0000'
category: ['Cpp', 'Design-pattern']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/design-pattern/abstract-factory-pattern/
---

## Abstract Factory

여러 객체의 군을 생성하기 위한 인터페이스를 제공

```cpp
#include <iostream>
using namespace std;

struct WinButton { void Draw() { cout << "Draw WinButton" << endl; }};
struct GTKButton { void Draw() { cout << "Draw GTKButton" << endl; }};

struct WinEdit { void Draw() { cout << "Draw WinEdit" << endl; }};
struct GTKEdit { void Draw() { cout << "Draw GTKEdit" << endl; }};

int main(int argv, char** argc)
{
    if(strcmp(argc[1], "Windows") == 0)
        ? pBtn = new WinButton;
    else
        ? pBtn = new GTKButton;
        // 공동의 기반클래스가 필요하다

    pBtn->Draw();
}
```

```cpp
struct IEdit
{
    virtual void Draw() = 0;
    virtual ~IEdit() {}
};

struct IButton
{
    virtual void Draw() = 0;
    virtual ~IButton() {}
};

struct WinButton : public IButton { void Draw() { cout << "Draw WinButton" << endl; }};
struct GTKButton : public IButton { void Draw() { cout << "Draw GTKButton" << endl; }};

struct WinEdit : public IEdit { void Draw() { cout << "Draw WinEdit" << endl; }};
struct GTKEdit : public IEdit { void Draw() { cout << "Draw GTKEdit" << endl; }};

int main(int argv, char** argc)
{
    IButton * pBtn;
    if(strcmp(argc[1], "Windows") == 0)
        pBtn = new WinButton;
    else
        pBtn = new GTKButton;
        // 버튼이 추가될때마다 if문이 추가되어야하나?

    pBtn->Draw();
}
```

```cpp
struct IFactory
{
    virtual IButton* CreateButton() = 0;
    virtual IEdit* CreateButton() = 0;
    virtual ~IFactory() {}
};

struct WinFactory : public IFactory
{
    WinButton* CreateButton() { return new WinButton; }
    WinEdit* CreateEdit() { return new WinEdit; }
};

struct GTKFactory : public IFactory
{
    GTKButton* CreateButton() { return new GTKButton; }
    GTKEdit* CreateEdit() { return new GTKEdit; }
};

int main(int argv, char** argc)
{
    * pFactory;
    if(strcmp(argc[1], "Windows") == 0)
        pFactory = new WinFactory;
    else
        pFactory = new GTKFactory;
  
    IButton* pBtn = pFactory->CreateButton();
    pBtn->Draw();
}
```
