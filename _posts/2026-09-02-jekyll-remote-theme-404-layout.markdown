---
layout: post
title: "remote_theme을 쓰는 Jekyll에서 404 페이지만 레이아웃이 빠지는 문제"
date: 2026-09-02 15:55:00 +0900
categories: jekyll
tags: [jekyll, github-pages, remote-theme, minima]
---

## 증상

GitHub Pages 블로그를 빌드할 때마다 아래 경고가 계속 찍히고 있었습니다.

```text
Build Warning: Layout 'default' requested in 404.html does not exist.
```

빌드는 정상적으로 끝나고(exit code 0) 404 페이지도 생성되기 때문에, 한동안 그냥 지나쳤던 경고입니다.
그런데 실제로 생성된 `_site/404.html`을 열어보니 단순한 경고가 아니었습니다.

```html
<style type="text/css" media="screen">
  .container { ... }
</style>

<div class="container">
  <h1>404</h1>
  <p><strong>Page not found :(</strong></p>
</div>
```

`<html>`, `<head>`도 없고 사이트 헤더와 푸터도 없습니다.
레이아웃이 적용되지 않은 채로 본문만 그대로 출력되고 있었습니다.
즉 방문자가 없는 주소로 들어오면, 사이트 네비게이션도 없는 알몸 HTML을 보게 되는 상태였습니다.

## 확인

블로그의 설정은 이랬습니다.

```yaml
# _config.yml
remote_theme: jekyll/minima
plugins:
  - jekyll-feed
  - jekyll-sitemap
  - jekyll-remote-theme
```

`_layouts/` 디렉터리 없이 remote theme의 레이아웃을 그대로 쓰는 구조입니다.
그리고 `404.html`의 front matter는 `layout: default`였습니다.

여기서 이상한 점은, minima 테마에 `default` 레이아웃이 **분명히 존재한다**는 것입니다.

```
minima-2.5.1/_layouts/default.html
minima-2.5.1/_layouts/home.html
minima-2.5.1/_layouts/page.html
minima-2.5.1/_layouts/post.html
```

게다가 다른 페이지들은 멀쩡했습니다. 생성된 결과물에서 사이트 헤더/푸터를 세어보면 이렇습니다.

```bash
$ grep -c "site-header\|site-footer\|site-nav" _site/index.html _site/about/index.html _site/404.html
_site/index.html:3
_site/about/index.html:3
_site/404.html:0
```

`index.md`는 `layout: home`, `about.md`는 `layout: page`를 쓰는데 둘 다 정상입니다.
그런데 minima의 `home.html`과 `page.html`은 각자 front matter에서 이렇게 선언합니다.

```yaml
---
layout: default
---
```

**`home`과 `page` 레이아웃이 상속하는 `default`는 잘 찾아지는데, 페이지가 직접 `layout: default`를 요청하면 못 찾는 상황**이었습니다.

## 해결

원인을 더 파고들 수도 있지만, 실용적인 해결책은 간단했습니다.
정상 동작이 확인된 `page` 레이아웃을 쓰면 됩니다.
`page`는 내부적으로 `default`를 상속하므로 결과물은 동일한 사이트 구조를 갖습니다.

```yaml
# 404.html
---
layout: page
---
```

이렇게 바꾸고 다시 빌드하면 경고가 사라집니다.

```bash
$ bundle exec jekyll build
      Remote Theme: Using theme jekyll/minima
       Jekyll Feed: Generating feed for posts
                    done in 0.481 seconds.

$ grep -c "site-header\|site-footer\|site-nav" _site/404.html
3
```

이제 404 페이지에도 다른 페이지와 동일하게 헤더와 푸터가 붙습니다.

## 정리

- 빌드가 성공(exit 0)했다고 결과물이 의도대로 나온 것은 아닙니다. Jekyll의 레이아웃 관련 경고는 조용히 넘어가지만, 실제로는 페이지 하나가 통째로 깨져 있을 수 있습니다.
- `remote_theme`을 쓸 때는 테마에 레이아웃 파일이 존재하는 것과, 내 페이지에서 그 레이아웃 이름이 해석되는 것이 별개일 수 있습니다.
- 확인은 경고 메시지가 아니라 `_site/`에 생성된 결과물로 하는 게 확실합니다. `grep`으로 헤더/푸터 유무만 세어봐도 금방 드러납니다.
