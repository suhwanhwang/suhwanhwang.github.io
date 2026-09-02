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
  permalink가 `/<categories>/<YYYY>/<MM>/<DD>/<slug>.html` 형태로 생성되므로 카테고리는 URL의 일부가 된다.
- `tags`는 선택. 세부 키워드가 여러 개일 때만 붙인다.
- 참고할 실제 파일: `_posts/2025-08-16-fix-sslv3-illegal-parameter-on-data-go-kr.markdown`(tags 있는 형태),
  `_posts/2025-07-24-github-workflow-google-credentials-safely.markdown`

## 작성 스타일

- 한국어, 존댓말(기존 포스트들이 "~합니다" 체).
- 최근 포스트들의 구조를 따른다: 증상/문제 → 원인 → 해결 → (필요시) 정리.
  단순 소개성 글이면 이 구조를 억지로 맞추지 말고 자연스럽게.
- 코드블록에는 반드시 언어 태그를 붙인다.
- 대화 로그를 그대로 옮기지 말고, 결론과 근거만 남겨 글로 재구성한다.
  "제가 물어봤더니" 같은 대화 흔적, 시행착오 중 틀린 정보는 빼거나 정리한다.
- **붙여넣은 내용에 섞인 개인정보·API 키·토큰·내부 경로는 반드시 제거하거나 placeholder로 치환한다.**
  저장소는 public이고 secret scanning push protection이 켜져 있다.

## 절차

1. `git checkout master && git pull origin master`로 최신 상태에서 시작
2. `git checkout -b post/<slug>`
3. 포스트 파일 작성
4. `bundle exec jekyll build`로 검증 — 경고나 에러가 없어야 한다
   - Claude Code on the web 등 로케일이 UTF-8이 아닌 환경에서는 SCSS 변환 단계에서
     `Invalid US-ASCII character` 에러가 난다. `LANG=C.UTF-8 LC_ALL=C.UTF-8`를 지정하고 빌드한다.
   - `bundle exec jekyll`이 `command not found`면 gem은 설치돼 있어도 binstub이 없는 것이니
     `ruby $(gem contents jekyll | grep exe/jekyll) build`로 exe를 직접 실행한다.
   - 이 에러들은 테마/환경 문제이지 포스트 문제가 아니다. 포스트가 정상 렌더됐는지는
     `_site/<categories>/<YYYY>/<MM>/<DD>/<slug>.html`이 생성됐는지로 확인한다.
5. 커밋 후 `git push -u origin post/<slug>` (`_site` 등 빌드 산출물은 커밋에서 제외)
6. PR 생성. PR 본문에는 글의 주제와 카테고리를 한두 줄로 요약
   - `gh` CLI가 있으면 `gh pr create`, 없으면(원격 실행 환경 등) GitHub MCP
     `mcp__github__create_pull_request`를 쓴다. base는 `master`.
7. PR 링크를 사용자에게 알려주고, 검토 후 머지하면 된다고 안내

`master`는 브랜치 보호가 걸려 있다(CI `build` 통과 필수). 절대 master에 직접 push하지 말고 반드시 PR을 경유한다.
