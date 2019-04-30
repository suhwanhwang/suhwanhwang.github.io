---
layout: post
title:  "github + jekyll + disqus로 블로그 만들기"
date:   2019-04-28 14:45:00 +0900
categories: jekyll
---
[Soft Skills]책을 읽으면서 블로그를 다시 시작해야겠다는 생각으로 검색하는 도중에 [jekyll]에 대해서 알게되었다. 

[jekyll]이란 [markdown]형식의 문서로 정적 웹사이트나 블로그를 생성해주는 툴로
[github pages]에서 기본적으로 제공해주고 있기 때문에 git이나 텍스트 편집에 익숙하다면 블로깅하는데 간단히 사용하기 좋다.
[jekyll]이 정적 웹사이트만 지원가능하기 때문에 블로그에 필요한 댓글 기능은 [disqus]와 연동하면 된다.

자세한 방법은 구글링하면 자료가 많고, 간단히 정리해보겠다.

우선, [github]에 가입하고, Repository를 생성한다.
Repository명은 [username].github.io 로 하면 https://[username].github.io 로 접속 가능하다.

다음으로 [jekyll installation]를 참고해서 [jekyll]을 설치한다.

아래와 같이 [github] repository를 로컬에 clone하고 [jekyll] 프로젝트를 생성하고 실행한다. 
{% highlight shell %}
 $ git clone https://github.com/[username]/[username].github.io.git
 $ jekyll new [username].github.io
 $ cd [username].github.io
 $ bundle exec jekyll serve
{% endhighlight %}

브라우저에서 http://localhost:4000 을 열면 아래와 같이 생성된 블로그를 볼 수 있다.
![][jekyll run]

다음으로 댓글 기능을 위해서 [disqus]에 가입한다.
가입이후 I want to install Disqus my site 선택해서 new site를 생성한다.
Website Name에 입력하는 내용이 연동에 사용되기 때문에 잘 기억해 둔다.

[github] repository에 있는 _config.yml에 아래와 같은 내용을 추가하고 shortname에 [disqus] Website Name을 입력한다.

{% highlight yaml %}
# Disqus settings
disqus:
  shortname: websitename
{% endhighlight %}

아래와 같이 댓글을 입력할 수 있는 폼이 추가 되었다. facebook이나 twitter계정으로 로그인해서 입력 가능하다.
![][jekyll disqus]

[Soft Skills]:  https://www.amazon.com/dp/1617292397/ref=cm_sw_em_r_mt_dp_U_N5tXCbX7A79B1
[jekyll]: https://jekyllrb.com/
[markdown]: https://daringfireball.net/projects/markdown/
[github pages]: https://pages.github.com/
[disqus]: https://disqus.com/
[github]: https://github.com/
[jekyll installation]: https://jekyllrb.com/docs/installation/
[jekyll run]: https://user-images.githubusercontent.com/40444306/56998849-c64c0900-6be7-11e9-8f07-3e437849eca7.png
[jekyll disqus]:https://user-images.githubusercontent.com/40444306/57000309-eed70180-6bed-11e9-8c91-a549c7f3d748.png