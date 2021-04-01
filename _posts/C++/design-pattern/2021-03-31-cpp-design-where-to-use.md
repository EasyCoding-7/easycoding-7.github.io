---
layout: post
title:  "(C++ : Design-pattern) 디자인 패턴은 언제쓸까?(edit control을 만들어보자)"
summary: ""
author: C++
date: '2021-03-31 0:00:00 +0000'
category: ['Cpp', 'Design-pattern']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/design-pattern/where-to-use/
---

```cpp
#include <iostream>
#include <string>
using namespace std;

class Edit
{
    string data;
public:
    string getData()
    {
        cin >> data;
        return data;
    }
};

int main()
{
    Edit edit;

    while(1)
    {
        string s = edit.getData();      
        // 문제) 만약 나이를 넣고 싶은데 string으로 받기에 모든 문자를 다 넣을 수 있다.
        // 숫자만 받아보자.
        cout << s << endl;
    }
}
```

```cpp
#include <conio.h>      // getch()

class Edit
{
    string data;
public:
    string getData()
    {
        data.clear();
        while(1)
        {
            char c = getch();
            if(c == 13) break;      // 이런식으로 문자입력을 막을 수 있다.

            if(isdigit(c))
            {
                data.push_back(c);
                cout << c;
            }
        }
        cout << endl;
        return data;
    }
};
```

만약 기획이 바뀌어 이번엔 다시 문자를 입력하고싶다면?<br>
그리고 매번 이런식으로 Edit을 바꿔야하나?<br>
그냥 Edit 정책을 변경가능하게 만들어보자.<br>

```cpp
class Edit
{
    string data;
public:
    virtual bool validate(char c)
    {
        return isdigit(c);
    }

    string getData()
    {
        data.clear();
        while(1)
        {
            char c = getch();
            if(c == 13) break;

            if(validate(c))
            {
                data.push_back(c);
                cout << c;
            }
        }
        cout << endl;
        return data;
    }
};

class AddressEdit : public Edit
{
    public:
    virtual bool validate(char c) override
    {
        return true;        
        // validate 정책을 항상 true로 둔다. -> 숫자, 문자 모두 받는다.
    }
};

int main()
{
    AddressEdit edit;

    while(1)
    {
        string s = edit.getData();
        cout << s << endl;
    }
}
```

이런식으로 코드의 구조에 따라 코드가 깔끔해 질수있다. 이런 패턴을 정리한 것이 디자인 패턴이다.

---

## 조금 더 진화시켜보자

조금 더 고민해보면 두 가지 방법이 있는데,<br>

1. 변하는 것을 가상 **함수**로 만들어서 파생클래스로 만들어 보는게 어떨까? -> Template method
2. 변하는 것을 **인터페이스화** 해서 파생클래스로 만들어 보는게 어떨까? -> Strategy Composition

더 자세한 설명은 Template, Strategy Pattern에서 설명