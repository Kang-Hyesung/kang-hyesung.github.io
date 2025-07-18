---
title: Github 블로그 개시글 관리 팁
author: hyesung
date: 2025-07-18 12:00:00 +0900
description: _posts 내 폴더구조를 구현해보자
---

## 글을 작성하게 된 이유
GitHub 블로그를 구성하고 프로젝트를 에디터로 열어보면, 기본적으로 모든 글이 `_posts` 폴더에 모여 있다
처음에 아래와 같은 구조를 보면…
```
\_posts/
├── 2025-07-15-첫번째-글.md
├── 2025-07-16-두번째-글.md
├── 2025-07-17-세번째-글.md
└── ...
```
모든 주제의 글이 `_posts` 폴더 하위의 `.md` 파일로 존재하면 카테고리별로 분류도 안 되고 관리하기 너무나도 복잡해질 것 같다는 생각이 들었다.

나도 GitHub 블로그에 대해 잘 모르다보니 GPT 의 도움을 받아 내가 적용한 방법을 소개하고자 한다.

---

## 내가 적용한 방법

1. `_posts` 폴더 안에 주제별 서브폴더 생성
2. `_config.yml`에 경로별 메타데이터(scope) 등록

파일은 폴더별로 정리하면서, Jekyll 빌드 시에는 `categories`로 인식되도록 설정한다

---

### 1. 폴더 구조 예시
```
\_posts/
└── blogging/
└── github\_blogging/
├── 2025-07-15-github-setup.md
└── 2025-07-18-folder-category.md
```
* 최상위 `_posts` 폴더는 그대로 두고
* 그 안에 주제 → 세부 주제 형태로 폴더를 만든다
* 실제 경로는 필요에 따라 자유롭게 변경 가능하다

---

### 2. `_config.yml` 설정

```yaml
# _config.yml

collections:
  posts:
    output: true

defaults:
  - scope:
      path: "_posts/blogging/github_blogging"   # 실제 경로와 정확히 일치해야 함
      type: posts
    values:
      categories: ["Blogging", "Github 블로그"]  # 원하는 카테고리 이름으로 설정
```

* `path`에는 `_posts` 폴더 아래 실제 경로를 `/`로 연결해서 적는다
* `type: posts`는 Jekyll이 해당 경로를 포스트로 인식하게 한다
* `values.categories`에 작성한 배열이 각 글의 `categories`로 자동 적용된다

---

## 장점

* 파일 관리가 한눈에 보인다: 주제별 폴더로 분리되어 VSCode에서도 깔끔하다
* 자동 카테고리 적용: Front Matter에 매번 `categories`를 쓰지 않아도 된다
* 유연한 구조 확장: 새로운 주제를 폴더만 추가하면 즉시 반영된다

---

## 추가 팁

* 태그(tags)를 함께 쓰고 싶다면, `values`에 `tags: [...]`를 추가한다
* 퍼머링크(permalink)나 레이아웃(layout)도 scope로 설정할 수 있다
* 로컬에서 `jekyll serve`로 테스트해 보면 새 폴더와 설정이 잘 적용되는지 확인할 수 있다

---


구성하면서 든 생각이 옵시디언의 프론트메타 기능과 연계하면 아주 편리하게 구성할 수 있을 것 같은데.. 한번 날잡고 해봐야겠다..



