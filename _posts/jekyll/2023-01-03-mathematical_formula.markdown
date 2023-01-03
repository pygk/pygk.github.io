---
layout: custom
title: jekyll 블로그에서 수식 사용하기
date: 2023-01-03 22:05:00 +0900
last_modified_at: 2023-01-03 22:05:00 +0900
category: jekyll
tags: ["minimal mistakes"]
published: True
use_math: true
---
> jekyll 블로그에서 수식 사용하기 (minimal mistakes 테마)

### 1. `/_config.yml` 파일 수정
```yml
# Conversion
markdown: kramdown
highlighter: rouge
lsi: false
excerpt_separator: "\n\n"
incremental: false
```

### 2. `/_includes/mathjax_support.html` 파일 생성
```html
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    TeX: {
        equationNumbers: {
            autoNumber: "AMS"
        }
    },
    tex2jax: {
        inlineMath: [ ['$', '$'] ],
        displayMath: [ ['$$', '$$'] ],
        processEscapes: true,
    }
});
MathJax.Hub.Register.MessageHook("Math Processing Error",function (message) {
        alert("Math Processing Error: "+message[1]);
	});
MathJax.Hub.Register.MessageHook("TeX Jax - parse error",function (message) {
        alert("Math Processing Error: "+message[1]);
	});
</script>
<script type="text/javascript" async
    src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
```

### 3. `_layout/default.html`파일의 `<head>` 부분에 코드 삽입
```html
{% if page.use_math %}
	{% include mathjax_support.html %}
{% endif %}
```

### 4. 수학식을 표시할 포스트의 front-matter에 `use_math: true` 적용
```md
---
title: test
use_math: true
---
```

### 5. 테스트
- $y=x+1$


### Reference
GitHub 블로그에서 마크다운 수학수식 사용하기, zzennin's DeepLearning[https://chaelin0722.github.io/blog/mathjex/](https://chaelin0722.github.io/blog/mathjex/)