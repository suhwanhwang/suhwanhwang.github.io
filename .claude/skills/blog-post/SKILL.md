---
name: blog-post
description: 붙여넣은 대화 내용이나 주제 메모를 이 블로그의 컨벤션에 맞는 Jekyll 포스트 초안으로 만들고 PR까지 올린다. 사용자가 "이 대화 블로그 포스트로 만들어줘", "이거 블로그에 올려줘" 같은 요청을 하거나 /blog-post를 부를 때 사용.
---

# 블로그 포스트 작성

claude.ai 대화 내용이나 주제 메모를 받아 이 블로그(https://shwang.dev)의 포스트 초안을 만들고 PR을 연다.
최종 검토와 머지는 사용자가 GitHub에서 직접 하므로, PR을 열어 링크를 알려주는 것까지가 이 스킬의 범위다.

## 입력

사용자가 붙여넣은 대화 내용 또는 주제 메모. URL만 주어졌으면 WebFetch로 내용을 가져온다.
내용이 너무 짧거나 무엇에 대한 글인지 불분명하면, 추측해서 쓰지 말고 사용자에게 물어본다.

## 포스트 컨벤션

파일 경로: `_posts/YYYY-MM-DD-<영문-slug>.markdown` (오늘 날짜, slug는 소문자 하이픈)

Front matter:

```yaml
---
layout: post
title: "제목"
date: 2026-09-02 15:10:00 +0900
categories: internet
tags: [python, requests, tls]
---
```

- `categories`는 공백 구분. 기존에 쓰인 값: `internet`, `algorithm`, `android`, `jekyll`, `github workflow gcp`.
  기존 값 중 맞는 게 있으면 재사용하고, 없으면 새로 만들되 한 단어 소문자로.
  **반드시 소문자로 쓴다** — 과거 포스트 하나가 `categories: Android`로 대문자를 썼지만 URL은 소문자로 생성되므로 혼란만 준다.
  permalink가 `/<categories>/<YYYY>/<MM>/<DD>/<slug>.html` 형태로 생성되므로 카테고리는 URL의 일부가 된다.
- `tags`는 선택. 세부 키워드가 여러 개일 때만 붙인다.
- 참고할 실제 파일: `_posts/2025-08-16-fix-sslv3-illegal-parameter-on-data-go-kr.markdown`(tags 있는 형태),
  `_posts/2025-07-24-github-workflow-google-credentials-safely.markdown`

**2025년 포스트(위 두 개)만 스타일 참고 대상으로 삼는다.** 2019~2021년 포스트는 평서체(`~이다`)에
`{% highlight %}` Liquid 태그를 쓰는 옛 스타일이라 따라하면 안 된다.

## 글 구조

주제에 따라 아래 두 가지 중 맞는 쪽을 고른다. 기존 2025년 포스트가 각각 하나씩이다.

**문제 해결형** (`2025-08-16-fix-sslv3...` 참고) — 겪은 오류를 다룰 때

```
## 증상        오류 메시지, 재현 코드
### 특징       "브라우저에선 되는데 requests만 실패" 같은 관찰 포인트
## 원인 (추정)  가설을 불릿으로
## 진단 팁      확인에 쓴 명령어들
## 해결 방법
## 관련 리소스
```

**튜토리얼형** (`2025-07-24-github-workflow...` 참고) — 방법을 안내할 때

```
(도입부)       맥락 2~3문단 + 결론 한 줄 선요약, 헤딩 없이 시작
### 왜 ~가 필요한가?
### 1. 첫 단계
### 2. 다음 단계
### 결론
```

구조를 억지로 맞추지 말고, 글에 맞으면 섹션을 빼거나 더해도 된다.

## 작성 스타일

- 한국어, 존댓말("~합니다" 체).
- **헤딩은 `##`과 `###`만 쓴다. `#`은 절대 쓰지 않는다** — 포스트 제목이 레이아웃에서 이미 `<h1>`으로
  렌더링되므로 본문에 `#`을 쓰면 h1이 중복된다.
- 코드블록은 백틱 펜스에 언어 태그를 붙인다(```` ```python ````, ```` ```yaml ````, ```` ```bash ````,
  출력·로그는 ```` ```text ````). `{% highlight %}` Liquid 태그는 옛 포스트 스타일이므로 쓰지 않는다.
- 참고 링크가 있으면 마지막에 `## 관련 리소스` 섹션으로 모아 정리한다(선택).
- 제목은 무엇을 했는지 드러나게. 기존 예: "~ 해결기", "~ 안전하게 사용하기", "~ 연결하기".
- 대화 로그를 그대로 옮기지 말고, 결론과 근거만 남겨 글로 재구성한다.
  "제가 물어봤더니" 같은 대화 흔적, 시행착오 중 틀린 정보는 빼거나 정리한다.
- **붙여넣은 내용에 섞인 개인정보·API 키·토큰·내부 경로는 반드시 제거하거나 placeholder로 치환한다.**
  저장소는 public이고 secret scanning push protection이 켜져 있다.

## 이미지

저장소에 `assets/` 디렉터리가 없다. 기존 포스트의 이미지는 전부 `user-images.githubusercontent.com`
외부 URL이다(GitHub 이슈/코멘트에 드래그해서 올리면 URL이 생긴다).

이미지는 직접 만들 수 없으므로, 글에 이미지가 필요하다고 판단되면 사용자에게 URL을 요청한다.
없으면 없는 대로 글을 완성한다 — 이미지 자리를 빈 placeholder로 남기지 않는다.

## 절차

1. `git checkout master && git pull origin master`로 최신 상태에서 시작
2. `git checkout -b post/<slug>`
3. 포스트 파일 작성
4. `bundle exec jekyll build`로 검증 — 경고나 에러가 없어야 한다
5. 커밋 후 `git push -u origin post/<slug>`
6. `gh pr create`로 PR 생성. PR 본문에는 글의 주제와 카테고리를 한두 줄로 요약
7. PR 링크를 사용자에게 알려주고, 검토 후 머지하면 된다고 안내

`master`는 브랜치 보호가 걸려 있다(CI `build` 통과 필수). 절대 master에 직접 push하지 말고 반드시 PR을 경유한다.
