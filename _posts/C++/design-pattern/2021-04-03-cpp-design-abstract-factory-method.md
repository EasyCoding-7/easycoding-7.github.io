---
layout: post
title:  "(C++ : Design-pattern) Factory Method"
summary: ""
author: C++
date: '2021-04-03 0:00:00 +0000'
category: ['Cpp', 'Design-pattern']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/design-pattern/factory-method/
---

## Factory Method 의도
* 객체를 생성하기 위해 인터페이스를 정의 하지만, 어떤 클래스의 인스턴스를 생성할 지에 대한 결정은 서브 클래스가 한다.
* Factory Method 패턴에서는 클래스의 인스턴스를 만드는 시점을 서브클래스로 미룬다.
* template method와 유사한 형태이다

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

class WinDialog
{
public:
    void Init()
    {
        WinButton* pBtn = new WinButton;
        WinEdit* pEdit = new WinEdit;

        pBtn->Draw();
        pEdit->Draw();
    }
};

class GTKDialog
{
public:
    void Init()
    {
        GTKButton* pBtn = new GTKButton;
        GTKEdit* pEdit = new GTKEdit;

        pBtn->Draw();
        pEdit->Draw();
    }
};
// 공통된 부분을 기반클래스로 뽑아보자.


int main(int argv, char** argc)
{
    WinDialog dlg;
    dlg.Init();
}
```

```cpp
class BaseDialog
{
public:
    void Init()
    {
        IButton* pBtn = CreateButton();
        IEdit* pEdit = CreateEdit();

        pBtn->Draw();
        pEdit->Draw();
    }
    virtual IButton* CreateButton() = 0;
    virtual IEdit* CreateEdit() = 0;
};

class WinDialog : public BaseDialog
{
public:
    virtual IButton* CreateButton() { return new WinButton; }
    virtual IEdit* CreateEdit() { return new WinEdit; }
};
```