---
layout: custom
title: jekyll 블로그 카테고리 페이지 만들기
date: 2022-12-21 15:26:00 +0900
last_modified_at: 2022-12-21 15:26:00 +0900
category: jekyll
tags: ["minimal mistakes"]
published: True

---
> jekyll 블로그 카테고리&테크 페이지 만들기 (minimal mistakes 테마)

- 네비게이션 바 목록에 category 와 tag 추가하기
    - `/_data/navigation.yml` 수정
        ```yml
        main:
            - title: "Category"
                url: "/categories/"
            - title: "Tag"
                url: "/tags/"
        ```

    - 아직 카테고리 페이지가 없기 때문에 네비게이션 바에 추가된 "Category" 와 "Tag" 를 클릭해도  not found 페이지가 나옴
    - [https://school.programmers.co.kr/learn/courses/30/lessons/138475](https://school.programmers.co.kr/learn/courses/30/lessons/138475)

- config 파일 확인 및 카테고리&태그 페이지 생성

    - `_config.yml` 파일
        ```yml
        category_archive:
            type: liquid
            path: /categories/
        tag_archive:
            type: liquid
            path: /tags/
        ```

    - `/_pages/category_archive.md` 파일 생성
        ```md
        ---
        title: "Posts by Category"
        layout: categories
        permalink: /categories/
        author_profile: true
        ---
        ```

    - `/_pages/tag_archive.md` 파일 생성
        ```md
        ---
        title: "Posts by Tag"
        layout: tags
        permalink: /tags/
        author_profile: true
        ---
        ```
