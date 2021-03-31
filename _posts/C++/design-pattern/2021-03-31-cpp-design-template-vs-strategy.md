---
layout: post
title:  "(C++ : Design-pattern) Template Method, Strategy Pattern 비교"
summary: ""
author: C++
date: '2021-03-31 0:00:00 +0000'
category: ['Cpp', 'Design-pattern']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/hello.jpg
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/design-pattern/template-vs-strategy/
---

1. 변하는 것을 가상 **함수**로 만들어서 파생클래스로 만들어 보는게 어떨까? -> Template method
2. 변하는 것을 **인터페이스화** 해서 파생클래스로 만들어 보는게 어떨까? -> Strategy Composition

우선 아래는 Template Method 예제이다. 

```cpp
// validation을 위한 인터페이스
struct IValidator
{
    virtual bool validate(string s, char c) = 0;
    virtual bool iscomplete(string s) { return = true; }
    virtual ~IValidator() {}
};

class Edit
{
    string data;
    IValidator* pVal = 0;
public:
    void setValidator(IValidator* p) { pVal = p; }

    string getData()
    {
        data.clear();
        while(1)
        {
            char c = getch();
            if(c == 13 &&
                 pVal == 0 || pVal->iscomplete(data)) break;

            if( pVal == 0 || pVal->validate(data, c) )      // validation을 위임한다.
            {
                data.push_back(c);
                cout << c;
            }
        }
        cout << endl;
        return data;
    }
};

class LimitDigitValidator : public IValidator
{
    int value;
public:
    LimitDigitValidator(int n) : value(n) {}
    virtual bool validate( string s, char c )
    {
        return s.size() < value && isgidit(c);
        // 이런식으로 변하는 부분을 함수화 했다.
    }

    virtual bool iscomplete(string s)
    {
        return s.size() == value;
    }
};

int main()
{
    Edit edit;
    LimitDigitValidator v(5);
    edit.setValidator(&v);
}
```

## Template method

* 장점은 정책을 중간에 변경할 수 있다.

---

## Strategy 와 차이점?

**Strategy Composition** -> 상속 기반의 패턴들은 유연성이 떨어지고 실행 시간에 정책을 교체할 수 없다.(정책을 새로 만들어야한다.)

```cpp
int main()
{
    // AddressEdit edit;
    newAddressEdit edit;        // 정책변경을 위해 새로운 클래스를 만들어야함.

    while(1)
    {
        string s = edit.getData();
        cout << s << endl;
    }
}
```

단, 정책의 재 사용성이 필요없을 경우 template method이 훨씬 유리하다.

* template method : 상속을 통해 아에 재 정의
* Strategy pattern : 필요한 부분만 교체교체

---

## template method 정리

* 템플릿 메소드에서 알고리즘의 골격을 정의
* 알고리즘의 여러 단계중 일부는 서브 클래스에서 구현 -> 훅(hook) 메소드라 한다.
* 알고리즘의 전반적 구조를 유지하면서 서브 클래스에서 특정 단계를 재지정한다.

```cpp
class Shape
{
protected:
    virtual void DrawImp()      // hook method
    {
        cout << "Draw Shape" << endl;
    }
public:
    virtual void Draw() final
    {
        cout << "mutex lock" << endl;
        DrawImp();
        cout << "mutex unlock" << endl;
    }
    virtual Shape* Clone() { return new Shape(*this); }
};

class Rect : public Shape
{
    virtual void DrawImp()      // hook method
    {
        cout << "Rect Draw Shape" << endl;
    }
};
```

---

## Strategy pattern 정리

* 알고리즘의 군을 정의 하고, 각각을 캡슐화 해서 교환해서 사용할 수 있도록 만든다.
* Strategy 패턴을 활용하면 클라이언트와 독립적으로 알고리즘을 변경/생성 할 수 있다.

```cpp
struct IValidator
{
    virtual bool validate(string s, char c) = 0;
    virtual bool iscomplete(string s) { return = true; }
    virtual ~IValidator() {}
};

class Edit
{
    string data;
    IValidator* pVal = 0;
public:
    void setValidator(IValidator* p) { pVal = p; }

    string getData()
    {
        data.clear();
        while(1)
        {
            char c = getch();
            if(c == 13 &&
                 pVal == 0 || pVal->iscomplete(data)) break;

            if( pVal == 0 || pVal->validate(data, c) )      // validation을 위임한다.
            {
                data.push_back(c);
                cout << c;
            }
        }
        cout << endl;
        return data;
    }
};

class LimitDigitValidator : public IValidator
{
    int value;
public:
    LimitDigitValidator(int n) : value(n) {}
    virtual bool validate( string s, char c )
    {
        return s.size() < value && isgidit(c);
    }

    virtual bool iscomplete(string s)
    {
        return s.size() == value;
    }
};

int main()
{
    Edit edit;
    LimitDigitValidator v(5);
    edit.setValidator(&v);
}
```