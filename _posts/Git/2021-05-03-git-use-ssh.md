---
layout: post
title:  "(Git) SSH 사용법"
summary: ""
author: Git
date: '2021-05-03 0:00:00 +0000'
category: ['Git']
#tags: ['github-page', 'tag1-test1']
thumbnail: /assets/img/posts/Git-thumbnail.png
#keywords: ['github-page 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Git/how-to-use-ssh/
---

앞으로 ssh를 통해 깃 관리 예정.

## SSH Key 생성

sourcetree가 있다면 도구 -> SSH Key 생성 또는 불러오기

![](/assets/img/posts/Git/how-to-use-ssh-1.PNG){:class="img-fluid"}

Generate 선택 -> 생성 후 Save public Key, Save Private Key를 하고 잘 보관한다.<br>
(말 그대로 Key라서 잘보관해야한다 ...)

## Bitbucket SSH Key 등록

참고로 Public Key를 등록시 해당 계정에 Public Key를 등록과 해당 Repo의 Public Key 등록 두 가지가 있는데 계정에 Public Key를 등록하는게 편하다.

![](/assets/img/posts/Git/how-to-use-ssh-2.PNG){:class="img-fluid"}

public key를 넣는다.

![](/assets/img/posts/Git/how-to-use-ssh-3.PNG){:class="img-fluid"}

public key를 열어보면 저장된 양식이

```
---- BEGIN SSH2 PUBLIC KEY ----
Comment: "rsa-key-20210503"
<key내부>
---- END SSH2 PUBLIC KEY ----
```

이런식일텐데

Bitbucket에 넣을때는

```
ssh-rsa <key내부>
```

이렇게 넣어야 한다.

이렇게 하기 귀찮은 경우 ssh-key gen에서 private key를 열면 자동으로 저런 형식으로 보여준다

---

## SourceTree SSH 등록

SourceTree -> 도구 -> 옵션 -> 일반 -> SSH 클라이언트 설정

![](/assets/img/posts/Git/how-to-use-ssh-4.PNG){:class="img-fluid"}

---

## 원격지 주소를 SSH기반으로 변경

bitbucket에서 SSH주소를 받는다.

![](/assets/img/posts/Git/how-to-use-ssh-5.PNG){:class="img-fluid"}

소스트리에서 원격지 주소 변경

![](/assets/img/posts/Git/how-to-use-ssh-6.PNG){:class="img-fluid"}