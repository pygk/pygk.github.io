---
layout: custom
title: jekyll 블로그에서 수식 사용하기
date: 2023-01-03 22:05:00 +0900
last_modified_at: 2023-01-05 03:28:00 +0900
category: jekyll
tags: ["minimal mistakes", "MathJax", "jekyll"]
published: True
use_math: true
---
> jekyll 블로그에서 수식 사용하기 (MathJax V3)

---
jekyll 블로그에 MathJax를 적용하는 방법에 대한 많은 블로그 포스팅이 MathJax V2 버전을 사용하고 있고,   
내 블로그에 이 방법을 적용할 경우 `displayMath` 수식 렌더링에 에러가 발생했다. (예. `$$y=x+1\$$` $\to$ `\[y=x+1\]`)   
이 문제는 MathJax V3를 적용하여 해결하였고 관련 내용을 정리한다.  
MathJax V3 적용 방법도 MathJax V2 버전과 동일하다.

---

### 1. `/_config.yml` 파일 수정
```yml
# Conversion
markdown: kramdown
highlighter: rouge
lsi: false
excerpt_separator: "\n\n"
incremental: false
```

### 2. `/_includes/mathjax.html` 파일 생성
```html
<script type="text/javascript">
    window.MathJax = {
        tex: {
            packages: ['base', 'ams'],
            inlineMath: [ ['$', '$'], ['\\(', '\\)'] ],
            displayMath: [ ['$$', '$$'], ['\\[', '\\]'] ],
        },
        loader: {
            load: ['ui/menu', '[tex]/ams']
        }
    };
</script>
<script
    type="text/javascript" id="MathJax-script" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js">
</script>
```

### 3. `_layout/default.html`파일의 `<head>` 부분에 코드 삽입
```html
{% if page.use_math %}
	{% include mathjax.html %}
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

- inlineMath

```md
$y=x+1$
```  

$y=x+1$
  
```md
\\(y=x+1\\)
```   

\\(y=x+1\\)
  
- displayMath

```md
$$y=x+1$$
```  

$$y=x+1$$
  
```md
\\[y=x+1\\]
```  

\\[y=x+1\\]



### Reference
MathJax v3 in Jekyll, Arthur O’Dwyer \[[https://quuxplusone.github.io/blog/2020/08/19/mathjax-v3-in-jekyll/](https://quuxplusone.github.io/blog/2020/08/19/mathjax-v3-in-jekyll/)\]   
Add MathJax v3 Support to Jekyll and Hugo, std::bodun::blog \[[https://www.bodunhu.com/blog/posts/add-mathjax-v3-support-to-jekyll-and-hugo/](https://www.bodunhu.com/blog/posts/add-mathjax-v3-support-to-jekyll-and-hugo/)\]