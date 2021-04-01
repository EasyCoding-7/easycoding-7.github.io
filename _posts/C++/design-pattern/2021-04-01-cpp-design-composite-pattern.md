---
layout: post
title:  "(C++ : Design-pattern) Composite Pattern"
summary: ""
author: C++
date: '2021-04-01 0:00:00 +0000'
category: ['Cpp', 'Design-pattern']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/design-pattern/composite-pattern/
---

## Composite 패턴

* 객체들이 트리 구조로 구성하여 부분과 전체를 나타내는 계층구조로 만들어진다.
* 개별 객체와 복합 객체를 구별하지 않고 동일한 방법으로 다루게 된다.

설명만 봐선 뭔... ;; 차라리 코드를 보자

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

class BaseMenu
{
    string title;
public:
    BaseMenu(string s) : title(s) {}
    string getTitle() const { return title; }
    virtual void command() = 0;
};

class MenuItem : public BaseMenu
{
    int id;
public:
    MenuItem(string s, int n) : BaseMenu(s), id(n) {}       
    // BaseMenu에 디폴트 생성자가 없기에 BaseMenu(s)를 반드시 넣어 줘야함.
    virtual void command() override {
        cout << getTitle() << endl;
        getchar();
    }
};

class PopupMenu : public BaseMenu
{
    vector<BaseMenu*> v;
public:
    PopupMenu(string s) : BaseMenu(s) {}
    void addMenu(BaseMenu* p) { v.push_back(p); }

    virtual void command() override {
        while(1)
        {
            system("cls");
            int sz = v.size();
            for(int i = 0; i < sz; i++)
            {
                cout <<  i + 1 << ". " << v[i]->getTitle() << endl;
            }
            cout << sz + 1 << ". 상위 메뉴로" << endl;
            int cmd;
            cout << "메뉴를 선택하세요 >>";
            cin >> cmd;

            if(cmd<1 || cmd > sz+1) continue;       // 잘못된 입력
            if(cmd == sz+1) break;

            // 선택된 메뉴 실행
            v[cmd-1]->command();        
            // 핵심 -> 다형성으로 MenuItem, PopupMenu의 command를 모두 처리가능
        }
    }
};


int main()
{
    PopupMenu* menubar = new PopupMenu("MenuBar");
    PopupMenu* pm1 = new PopupMenu("화면설정");
    PopupMenu* pm2 = new PopupMenu("소리설정");

    MenuItem m1("정보 확인", 11);     


    menubar->addMenu(pm1);
    menubar->addMenu(pm2);
    menubar->addMenu(m1);

    pm1->addMenu(new MenuItem("해상도변경", 21);
    pm1->addMenu(new MenuItem("명암변경", 22);

    pm2->addMenu(new MenuItem("음량조절", 31);

    // 시작
    menubar->command();
}
```

아래와 같은 기능을 추가하고 싶다면?

```cpp
int main()
{
    PopupMenu* menubar = new PopupMenu("MenuBar");
    PopupMenu* pm1 = new PopupMenu("화면설정");
    PopupMenu* pm2 = new PopupMenu("소리설정");

    MenuItem m1("정보 확인", 11);     


    menubar->addMenu(pm1);
    menubar->addMenu(pm2);
    menubar->addMenu(m1);

    pm1->addMenu(new MenuItem("해상도변경", 21);
    pm1->addMenu(new MenuItem("명암변경", 22);
    pm2->addMenu(new MenuItem("음량조절", 31);

    // 하위 메뉴의 포인터를 받고 싶다
    BaseMenu* p = menubar->getSubMenu(1)->getSubMenu(0);

    menubar->command();
}
```

```cpp
class PopupMenu : public BaseMenu
{
    vector<BaseMenu*> v;
public:
    PopupMenu(string s) : BaseMenu(s) {}
    void addMenu(BaseMenu* p) { v.push_back(p); }
    BaseMenu* getSubMenu(int idx) { return v[idx]; }      
    // PopupMenu에만 넣으면 될까? -> Nope, BaseMenu에 넣어줘야함 SubMenu가 PopupMenu일지 MenuItem일지 모르기때문
```

```cpp
class BaseMenu
{
    string title;
public:
    BaseMenu(string s) : title(s) {}
    string getTitle() const { return title; }
    virtual void command() = 0;

    virtual BaseMenu* getSubMenu(int idx)
    {
        throw "unsupported function";        
        // MenuItem에서 호출시 throw를 던저버린다.
        return 0;
    }
};
```

---

## 기능 추가) 각 메뉴에서 새로운 기능을 추가하고자 한다면?

```cpp
#include "menu.hpp"

class MenuItem : public BaseMenu
{
    int id;
public:
    MenuItem(string s, int n) : BaseMenu(s), id(n) {}
    virtual void command()
    {
        // 여기는 실제 동작이 들어가는데 ...
        // 각 메뉴마다 다른 일을 수행해야한다. -> 어떻게 만들지??

        // 방법 1. - 가상 함수 처리
        doCommand();
    }
    virtual void doCommand() {}
};

class AddStudentMenu : public MenuItem
{
public:
    using MenuItem::MenuItem;   // 생성자 상속
    virtual void doCommand() { cout << "Add Student" << endl; }
};

int main()
{
    AddStudentMenu m1("Add Student ", 11);

    m1.command();
}
```

그런데 이런 방식은 메뉴가 50개라면 50개의 클래스를 만들어야한다.

```cpp
struct IMenuListner
{
    virtual void doCommand(int id) = 0;
    virtual ~IMenuListner() {}
};

class MenuItem : public BaseMenu
{
    int id;
    IMenuListner* pListner = 0;
public:
    MenuItem(string s, int n) : BaseMenu(s), id(n) {}
    void setLisnter(IMenuListner* p) {pListner = p;}
    virtual void command()
    {
        // 방법 2. - 변하는 것을 다른 클래스로 뽑자.
        if(pListner != 0)
            pListner->doCommand(id);
    }
};

class Dialog : public IMenuListner
{
public:
    virtual void doCommand(int id)
    {
        switch(id)
        {
        case 11: cout << "Diglog doCommand : 11" << endl; break;
        case 12: cout << "Diglog doCommand : 12" << endl; break;
        defualt: break;
        }
        
    }
}

int main()
{
    Diglog dlg;
    MenuItem m1("Add Student", 11);

    m1.setLisnter(&dlg);

    m1.command();
}
```

위 방식은 switch문이 너무 커지게 될꺼 같은데??

## 그래서 나온게 function 템플릿!

## 우선, 일반/멤버 함수 포인터를 하나의 변수에 담고 싶다면

```cpp
// 일단은 이 부분을 먼저 이해하고 들어가자
#include <iostream>
using namespace std;

void foo() { cout << "foo" << endl; }

class Dialog{
public:
    void Close() { cout << "Dialog Close" << endl; }
};

int main()
{
    // 이런게 하고싶다 void(*f1)()에는 &foo만 넣고 있고
    // void(Dialog::*f2)()에는 &Dialog::Close를 넣고있는데
    // 그냥 상관없이 &foo, &Dialog::Close 를 넣고 싶다면?

    // 정리하면 일반 함수 포인터(&foo)나 멤버 함수 포인터(&Dialog::Close)에 상관없이 모든 함수포인터를 넣고싶다는 말

    void(*f1)() = &foo;
    void(Dialog::*f2)() = &Dialog::Close;
}
```

```cpp
struct IAction
{
    virtual void Execute() = 0;
    virtual ~IAction() {}
};

class FuctionAction : public IAction
{
    typedef void(*FP)();
    FP handler;
public:
    FuctionAction(FP f) : handler(f) {}
    virtual Execute() { handler(); }
};

template<typename T>
class MemberAction : public IAction
{
    typedef void(T::*FP)();
    FP handler;
public:
    MemberAction(FP f, T* obj) : handler(f), target(obj) {}
    virtual Execute() { (target->*handler)(); }
};

int main()
{
    Dialog dlg;
    IAction* p = new FunctionAction(&foo);
    IAction* p2 = new MemberAction<Dialog>(&Dialog::Close, &dlg);

    p->Execute();       // foo 실행
    p2->Execute();      // Close 실행
}
```

코드를 좀 더 간단히 해보자.

```cpp
// 우선 아래를 이해해보자.
template<typename T> void square(T a) { return a * a; }

square<int>(3);
square(3);              // 3을 보고 컴파일러가 타입을 추론

list<int> s(10, 3);     // 클래스 템플릿은 타입추론이 안됨.
```

위 방법과 같이 타입추론을 컴파일러가 해주는데 `new MemberAction<Dialog>(&Dialog::Close, &dlg);`에서도 타입 추론이 안되나??

```cpp
// 함수 템플릿 -> 타입 추론 가능
template<typename T>
MemberAction<T>* action(void(T::*f)(), T* obj)
{
    return new MemberAction<T>(f, obj);
}

int main()
{
    Dialog dlg;
    IAction* p = new FunctionAction(&foo);
    // IAction* p2 = new MemberAction<Dialog>(&Dialog::Close, &dlg);
    IAction* p2 = action(&Dialog::Close, &dlg);     // ok

    p->Execute();       // foo 실행
    p2->Execute();      // Close 실행
}
```

코드에 일관성을 부여해보자.

```cpp
// 함수 템플릿 -> 타입 추론 가능
template<typename T>
MemberAction<T>* action(void(T::*f)(), T* obj)
{
    return new MemberAction<T>(f, obj);
}

FunctionAction* action( void(*f)() )
{
    return new FunctionAction(f);
}

int main()
{
    Dialog dlg;
    // IAction* p = new FunctionAction(&foo);
    IAction* p = action(&foo);
    // IAction* p2 = new MemberAction<Dialog>(&Dialog::Close, &dlg);
    IAction* p2 = action(&Dialog::Close, &dlg);     // ok

    p->Execute();       // foo 실행
    p2->Execute();      // Close 실행
}
```

이런 기능을 C++ 자체에서 제공한다.

## function 템플릿

C++11 부터 지원되는 일반화된 함수 포인터 역할 템플릿

```cpp
#include <iostream>
#include <functional>
using namespace std;

void foo() { cout << "foo" << endl; }
void goo(int) { cout << "goo" << endl; }

class Dialog
{
public:
    void Close() { cout << "Dialog Close" << endl; }
};

int main()
{
    function<void()> f;
    f = &foo;
    f();        // foo 호출

    Dialog dlg;

    f = bind(&Dialog::Close, &dlg);
    f();        // dlg.Close() 호출

    f = bind(&goo, 5);
    f();        // goo(5) 호출
}
```