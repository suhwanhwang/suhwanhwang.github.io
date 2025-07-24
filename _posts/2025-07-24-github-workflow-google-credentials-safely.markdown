---
layout: post
title:  "GitHub Workflow에서 Google 인증 정보를 안전하게 사용하기"
date:   2025-07-24 10:00:00 +0900
categories: github workflow gcp
---

GitHub Actions 워크플로우에서 Google Cloud Platform (GCP) 같은 외부 서비스에 접근해야 할 때가 많습니다. 이때 인증을 위한 JSON 키 파일을 저장소에 직접 추가하는 것은 매우 위험한 방법입니다. 개인 키가 그대로 노출되어 보안 사고로 이어질 수 있기 때문입니다.

올바르고 안전한 방법은 인증 정보를 Base64로 인코딩하여 GitHub Secrets에 저장하고, 워크플로우 실행 시점에 디코딩해서 사용하는 것입니다.

이 글에서는 그 방법을 단계별로 소개합니다.

### 왜 Base64 인코딩이 필요한가?

GCP 인증 키와 같은 JSON 파일의 내용을 그대로 복사하여 GitHub Secret에 넣고 워크플로우에서 직접 사용하면 왜 문제가 될까요?

```yaml
# 잘못된 예시: 이렇게 하면 에러가 발생할 수 있습니다.
- name: Authenticate to Google Cloud (Bad Practice)
  run: echo "${{ secrets.GCP_CREDENTIALS_JSON }}" > gcp-credentials.json
```

가장 큰 이유는 **JSON 파일에 포함된 특수문자와 줄바꿈** 때문입니다. JSON 형식은 중괄호(`{}`), 큰따옴표(`"`), 그리고 여러 줄로 구성된 복잡한 구조를 가집니다. GitHub Actions 워크플로우의 `run` 스크립트에서 이러한 문자열을 `echo`와 같은 셸 명령어로 처리하면, 셸이 특수문자를 해석하거나 줄바꿈을 제대로 처리하지 못해 **JSON 파일이 깨지는 현상**이 발생합니다. 결과적으로 망가진 인증 파일 때문에 인증 단계에서 "invalid format"과 같은 오류가 발생하게 됩니다.

**Base64 인코딩**은 이 문제를 해결하는 가장 확실한 방법입니다. 파일의 전체 내용을 ASCII 문자로만 이루어진 한 줄의 안전한 문자열로 변환해주기 때문입니다. 이 문자열은 셸 환경에서 어떠한 해석이나 변경 없이 그대로 전달될 수 있으며, 워크플로우 실행 시점에 다시 원본 파일로 완벽하게 디코딩할 수 있습니다.

따라서, 민감한 정보이면서 복잡한 구조를 가진 파일은 항상 Base64로 인코딩하여 Secrets에 저장하는 것이 안전하고 안정적입니다.

### 1. Google Cloud 서비스 계정 키 준비

먼저 GCP 콘솔에서 서비스 계정 키(JSON 형식)를 다운로드하여 로컬 컴퓨터에 저장합니다. 파일 이름은 `gcp-credentials.json`이라고 가정하겠습니다.

### 2. 서비스 계정 키를 Base64로 인코딩하기

다음으로, 터미널에서 아래 명령어를 사용해 JSON 키 파일을 Base64 문자열로 인코딩합니다.

```bash
base64 gcp-credentials.json
```

macOS에서는 `-i` 옵션을 추가해야 할 수 있습니다.

```bash
base64 -i gcp-credentials.json
```

명령을 실행하면 터미널에 긴 문자열이 출력됩니다. 이 문자열 전체를 복사해 둡니다.

### 3. GitHub Secrets에 인코딩된 값 저장하기

1.  인증 정보를 사용할 GitHub 리포지토리로 이동합니다.
2.  `Settings` > `Secrets and variables` > `Actions` 메뉴로 들어갑니다.
3.  `New repository secret` 버튼을 클릭합니다.
4.  `Name`에는 `GCP_CREDENTIALS_BASE64`와 같이 식별하기 쉬운 이름을 입력합니다.
5.  `Secret` 필드에는 위에서 복사한 Base64 인코딩된 문자열을 붙여넣고 `Add secret` 버튼을 눌러 저장합니다.

### 4. GitHub Workflow에서 디코딩하여 사용하기

이제 워크플로우 파일(`.github/workflows/ci.yml` 등)에서 이 시크릿을 디코딩하여 사용할 수 있습니다.

```yaml
name: Deploy to Google Cloud

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Authenticate to Google Cloud
      run: |
        # GitHub Secret에 저장된 Base64 값을 디코딩하여 파일로 저장
        echo "${{ secrets.GCP_CREDENTIALS_BASE64 }}" | base64 --decode > ${{ runner.temp }}/gcp-credentials.json
        
        # GCP 관련 CLI나 라이브러리가 이 파일을 사용하도록 환경 변수 설정
        export GOOGLE_APPLICATION_CREDENTIALS=${{ runner.temp }}/gcp-credentials.json
        
        # 예시: gcloud CLI 인증
        # gcloud auth activate-service-account --key-file=${{ runner.temp }}/gcp-credentials.json
        
        echo "Successfully authenticated to Google Cloud!"

    # - name: Deploy application
    #   run: |
    #     # 인증이 필요한 배포 스크립트 실행
    #     ./deploy.sh
```

위 예제는 `GCP_CREDENTIALS_BASE64` 시크릿을 `base64 --decode` 명령어로 디코딩하여 임시 파일(`gcp-credentials.json`)로 만듭니다. 그 후 `GOOGLE_APPLICATION_CREDENTIALS` 환경 변수를 이 파일 경로로 지정하면, `gcloud` CLI나 다른 GCP 클라이언트 라이브러리들이 자동으로 이 인증 정보를 사용하게 됩니다.

### 결론

이렇게 하면 민감한 인증 정보를 Git 저장소에 직접 노출하지 않고도 안전하게 GitHub Actions 워크플로우를 구성할 수 있습니다. 보안이 향상될 뿐만 아니라, 워크플로우 파일이 깔끔해지고 관리하기 쉬워지는 장점도 있습니다.
