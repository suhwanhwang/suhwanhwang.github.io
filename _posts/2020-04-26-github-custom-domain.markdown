---
layout: post
title:  "내 도메인으로 GitHub Pages 연결하기"
date:   2020-04-26 17:05:00 +0900
categories: internet
---

GitHub Pages를 사용하면 **http://[Username].github.io** 로 연결할 수 있는데, 간단한 설정으로 내가 만든 도메인으로 접속할 수 있다. 

# Google Domains 설정

먼저 도메인을 만들어야하는데, **.dev**를 사용하고 싶어서 [Google Domains](https://domains.google.com/m/registrar/search)를 사용했다. 
도메인 구입시 국가는 미국으로 설정해야 가능하고, 원하는 도메인 주소를 검색해서 장바구니에 담고 결제하면 된다. 
(참고로 dev 도메인 가격은 1년에 $12이다.)
![](https://user-images.githubusercontent.com/40444306/80305910-d5fe2d80-87fa-11ea-8eb3-ffa2229d0283.png)

결제시 나오는 연락처 정보는 대한민국으로 변경해서 입력하고, 결제시 우편번호는 미국 우편번호 아무거나 입력하면 된다.

그리고 DNS 메뉴에서 **맞춤 리소스 레코드**에서 아래 4개의 GitHub Pages와 연결하는 IP주소를 추가해주면 된다. 

    185.199.108.153    
    185.199.109.153    
    185.199.110.153    
    185.199.111.153    

![](https://user-images.githubusercontent.com/40444306/80305995-5e7cce00-87fb-11ea-83ad-62f2225d366b.png)


# GitHub Pages 설정

GitHub Pages로 사용하는 Repository에서 Settings로 들어가면 GitHub Pages 관련 항목들이 있는데,
**Custom domain**에 본인의 도메인을 입력하고, **Enforce HTTPS**는 체크 해준다. (신규 도메인의 경우 조금 시간이 지나야 체크 가능하다.) 

**Your side is published at https://...** 라고 나오면 설정이 끝난 것이다.

![](https://user-images.githubusercontent.com/40444306/80306567-1364ba00-87ff-11ea-8bb4-2361a2d76993.png)