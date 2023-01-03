---
layout: custom
title: Markdown 수식 정리 (LaTex)
date: 2023-01-04 00:38:00 +0900
last_modified_at: 2023-01-04 00:38:00 +0900
category: markdown
tags: ["markdown", "LaTex"]
published: true
use_math: true

---

> markdown 수식 작성법  
> ~~설정이 잘못되었는지 outline(displayMath)으로 작성한 수식이 적용이 될 때도 있고 안될 때도 있다... 버전 문제?~~  
> ~~velog 와 github 블로그에서 허용되는 문법이 다른 것 같다.. 추후 수정 필요!~~


## 1. 수식 작성
* `$` 사이에 수식 작성

```md
$y=x+1$
$y$는 $x+1$ 이다.
```

$y=x+1$  
$y$는 $x+1$ 이다.


### 2. 중앙정렬 (outline)
- `$$` 사이에 수식 작성
    - 

```md
$$
y=x+1$$
```

$$
y=x+1$$


### 3. 특정 문자를 기준으로 정렬
- `aligned` 심볼을 통한 특정 문자 기준 정렬. `&` 를 기준으로 정렬됨.

```md
$$
\begin{aligned}
f(x)&=ax^2+bx+c\\
g(x)&=Ax^4
\end{aligned}$$
```

$$
\begin{aligned}
f(x)&=ax^2+bx+c\\
g(x)&=Ax^4
\end{aligned}$$


### 4. 수식 내에서의 줄바꿈
- `\\`를 입력

```md
$$
x+y=3\\-x+3y=2$$
```

$$
x+y=3\\-x+3y=2$$


### 5. 수식 내에서의 띄어쓰기

```md
$local minimum$(띄어쓰기 적용 X)
$local\,minimum$(띄어쓰기 한 번)
$local\;minimum$(띄어쓰기 두 번)
$local\quad minimum$(띄어쓰기 네 번)
```

$local minimum$(띄어쓰기 적용 X)  
$local\,minimum$(띄어쓰기 한 번)  
$local\;minimum$(띄어쓰기 두 번)  
$local\quad minimum$(띄어쓰기 네 번)


### 6. 곱셈 기호

```md
$y = A \times x + B$
```
$y = A \times x + B$


### 7. 첨자
- 윗 첨자: `^` , 아랫 첨자: `_`

```md
$a_1, a^2, a_1^2$
$y_i=x_i^3+x_{i-1}^2+x_{i-2}$
```

$a_1, a^2, a_1^2$  
$y_i=x_i^3+x_{i-1}^2+x_{i-2}$


### 8. 분수 표기법
- `\over` 를 사용하는 방법: `\over`를 기준으로 왼쪽은 분자, 오른쪽은 분모
- `\frac` 을 사용하는 방법: 첫 번째 문자는 분자, 두 번째 문자는 분모. 두 문자 이상은 중괄호`{}` 활용
```md
$s^2+2s+s\over s+\sqrt s+1$
$\frac{1+s}{s(s+2)}$
```
$s^2+2s+s\over s+\sqrt s+1$
$\frac{1+s}{s(s+2)}$

### 9. 절대값 표기법
- 분수와 같이 큰 객체의 절대값 표기는 `|`를 사용할 수 없음
- `\vert`와 `\left`, `\right`를 통하여 좌우 기호를 명시

```md
$\vert x \vert$
$\left\lvert \frac{s^2+1}{s^3+2s^2+3s+1} \right\rvert$
```

$\vert x \vert$
$\left\lvert \frac{s^2+1}{s^3+2s^2+3s+1} \right\rvert$


### 10. sin, log와 같은 기호를 세워서 표기
- 단어 앞에 \를 붙이게 되면 똑바로 글자를 쓸 수 있음

```md
$log_{10}{(x+1)}$   
$Asin(bx+c)$  

$\log_{10}{(x+1)}$   
$A\sin(bx+c)$  
```

$log_{10}{(x+1)}$   
$Asin(bx+c)$  

$\log_{10}{(x+1)}$   
$A\sin(bx+c)$  


### 11. 극한/시그마 표기법
- `\sum`, `\lim` 심볼 사용

```md
$\lim_{s\rarr\infin}{s^2}$  
$\sum_{i=0}^{\infin}{(y_i-t_i)^2}$
```
$\lim_{s\rarr\infin}{s^2}$  
$\sum_{i=0}^{\infin}{(y_i-t_i)^2}$

- `\displaystyle` 명시

```md
$\displaystyle\lim_{s\rarr\infin}{s^2}$  
$\displaystyle\sum_{i=0}^{\infin}{(y_i-t_i)^2}$
```
$\displaystyle\lim_{s\rarr\infin}{s^2}$  
$\displaystyle\sum_{i=0}^{\infin}{(y_i-t_i)^2}$

### 12. 벡터 표기법

```md
$\vec{a}$
$\overrightarrow{a}$
```
$\vec{a}$  
$\overrightarrow{a}$

### 13. 행렬 표기법
- `matrix` 심볼 사용
- 열 구분: `&`, 행 구분: `\\`

```md
$\begin{matrix}1&2\\3&4\\ \end{matrix}$
$\begin{pmatrix}1&2\\3&4\\ \end{pmatrix}$
$\begin{bmatrix}1&2\\3&4\\ \end{bmatrix}$
$\begin{Bmatrix}1&2\\3&4\\ \end{Bmatrix}$
$\begin{vmatrix}1&2\\3&4\\ \end{vmatrix}$
$\begin{Vmatrix}1&2\\3&4\\ \end{Vmatrix}$
```
$\begin{matrix}1&2\\3&4\\ \end{matrix}$  
$\begin{pmatrix}1&2\\3&4\\ \end{pmatrix}$  
$\begin{bmatrix}1&2\\3&4\\ \end{bmatrix}$  
$\begin{Bmatrix}1&2\\3&4\\ \end{Bmatrix}$  
$\begin{vmatrix}1&2\\3&4\\ \end{vmatrix}$  
$\begin{Vmatrix}1&2\\3&4\\ \end{Vmatrix}$  

### 14. 조각함수와 같은 case 표기법
- `cases` 심볼 사용

```md
$\vert x\vert=
\begin{cases}
-x,\;if\;x<0\\
+x,\;if\;x\geq0
\end{cases}$
```

$\vert x\vert=
\begin{cases}
-x,\;if\;x<0\\
+x,\;if\;x\geq0
\end{cases}$


### 15. 화살표

```md
$\larr$
$\rarr$ or $\to$
```

$\larr$  
$\rarr$ or $\to$


### 16. 부등호, 등호 등

|심볼|기호|심볼|기호|
|:-------------------|:------:|:-------------------|:------:|
|`$\gt$`|$\gt$|`$\lt$`|$\lt$|
|`$\geq$`|$\geq$|`$\leq$`|$\leq$|
|`$\approx$`|$\approx$|`$\sim$`|$\sim$|
|`$\cong$`|$\cong$|`$\approxeq$`|$\approxeq$|
|`$\simeq$`|$\simeq$|`$\eqsim$`|$\eqsim$|
|`$\doteq$`|$\doteq$|||
|`$\coloneq$`|$\coloneq$|`$\eqcolon$`|$\eqcolon$|
|`$\fallingdotseq$`|$\fallingdotseq$|`$\risingdotseq$`|$\risingdotseq$|
|`$\pm$`|$\pm$|`$\mp$`|$\mp$|


### 17. 점

|이름|마크다운|기호|
|:-----------------|:-----------------|:------:|
|점 문자|`$\dot{a}$`|$\dot{a}$|
|중간 점|`$\cdot$`|$\cdot$|
|콜론|`$\colon$`|$\colon$|
|하단 점|`$\ldotp$`|$\ldotp$|
|말줄임표|`$\cdots$`|$\cdots$|
|대각 말줄임표|`$\ddots$`|$\ddots$|
|수직 말줄임표|`$\vdots$`|$\vdots$|
|하단 말줄임표|`$ldots$`|$ldots$|
|왜냐하면|`$\because$`|$\because$|
|그러므로|`$\therefore$`|$\therefore$|

### 18. 단위 기호

|이름|마크다운|기호|
|:-----------------|:-----------------|:------:|
|도|`$\degree$`|$\degree$|
|섭씨|`$\celsius$`|$\celsius$|
|옴|`$\Omega$`|$\Omega$|
|위상각|`$\phase{}$`|$\phase{}$|


### Reference
[LaTex] Markdown 수식 작성법, Air on the C String "[https://velog.io/@d2h10s/LaTex-Markdown-수식-작성법](https://velog.io/@d2h10s/LaTex-Markdown-수식-작성법)"