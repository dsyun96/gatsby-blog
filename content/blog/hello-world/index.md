---
title: Hello, world!
date: 2021-08-02
description: "첫 글은 헬로월드가 국룰"
tags: [Blog]
---

# 헤더 1
## 헤더 2
### 헤더 3
#### 헤더 4
##### 헤더 5
###### 헤더 6

이것은 한글이다.

And, this is English.

두 줄을 붙여서 쓰면 어떻게 보일까?
이 줄은 위 줄과 바로 붙여서 쓴 상태이다.

    두 줄을 붙여서 쓰면 어떻게 보일까?
    이 줄은 위 줄과 바로 붙여서 쓴 상태이다.

> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Ut volutpat mollis bibendum. Aenean sit amet justo viverra, finibus magna quis, luctus tellus.
>
> Maecenas ex sem, finibus nec tortor vitae, pretium aliquam erat. Proin ultricies ex at risus consectetur, sit amet posuere felis sagittis. Fusce eleifend, arcu vitae dictum cursus, mauris ligula elementum eros, sit amet lobortis augue dui sed ante. 

이미지도 잘 올라가는지 확인해 보자.

![개발자 공감](https://cdn.clien.net/web/api/file/F01/2986983/def325ffb07a4fc1920.JPG?w=780&h=30000)

C언어로 헬로월드는 다음과 같이 짠다.

```C
#include <stdio.h>

int main(void) {
    printf("Hello, world!\n");
    return 0;
}
```

적당한 표

| 제목 1 | 제목 2    | 제목 3 |
| :----- | :-------:| ----: |
| 1      | Windows  | 2001  |
| 2      | Linux    | 2002  |
| 3      | Mac      | 2004  |

문단 1

    문단 2
    문단 3


> ## 인용문 안의 헤더
>
> 1. 안녕하세요
> 2. 안녕히가세요
>
>

<br>
<br>
<br>
<br>
<br>
<br>

# 여백의 미

<br>
<br>
<br>
<br>
<br>
<br>

- `간단한 코드`
- **볼드체**
- __볼드체2__
- *기울임체*
- _기울임체2_
<br>

    - `간단한 코드`
    - **볼드체**
    - __볼드체2__
    - *기울임체*
    - _기울임체2_

<!-- 여기엔 주석이 숨겨져 있다. -->

<br>

---

그리고 [KaTeX](https://katex.org/) 문법을 지원한다!

피타고라스의 정리는 $a^2 + b^2 = c^2$ 이다.

$$
\begin{CD}
A @>a>> B \\
@VbVV @AAcA \\
C @= D
\end{CD}
$$

$$
\sum_{i=1}^{n} i^2 = \frac{n(n+1)(2n+1)}{6}
$$

