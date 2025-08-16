---
layout: post
title: "공공데이터포털 SSL 오류(SSLV3_ALERT_ILLEGAL_PARAMETER) 간단 해결"
date: 2025-08-16 15:10:00 +0900
categories: internet
tags: [python, requests, tls, openssl, certifi, 공공데이터포털]
---

## 증상

`requests`로 `apis.data.go.kr` 호출 시 SSL 오류가 발생합니다:

```text
SSLError: [SSL: SSLV3_ALERT_ILLEGAL_PARAMETER] ...
```

## 원인

- 서버의 TLS 1.3/특정 확장 호환성 문제
- OpenSSL 기본 보안 레벨 상승으로 레거시 암호군 차단

## 해결 (요약)

- TLS 1.2로 고정
- OpenSSL 보안 레벨 완화: `@SECLEVEL=1`
- CA 번들 명시: `certifi`
- 세션 단위로만 적용(전역 설정 변경 금지)

## 최소 구현 예시

```python
import ssl, certifi, requests
from requests.adapters import HTTPAdapter

class TLS12Adapter(HTTPAdapter):
    def init_poolmanager(self, *args, **kwargs):
        ctx = ssl.create_default_context()
        try:
            ctx.minimum_version = ssl.TLSVersion.TLSv1_2
            ctx.maximum_version = ssl.TLSVersion.TLSv1_2
            ctx.set_ciphers("DEFAULT:@SECLEVEL=1")
        except Exception:
            pass
        kwargs["ssl_context"] = ctx
        return super().init_poolmanager(*args, **kwargs)

def http_get_json_compat(url: str, timeout: int = 10) -> dict:
    s = requests.Session()
    s.mount("https://", TLS12Adapter())
    try:
        r = s.get(
            url,
            headers={"Accept": "*/*", "Connection": "close"},
            timeout=timeout,
            allow_redirects=True,
            verify=certifi.where(),
        )
        r.raise_for_status()
        return r.json()
    finally:
        s.close()
```

## 메모

- 호환성 완화는 해당 도메인 세션에만 제한적으로 적용
- 실패 시 재시도/로깅 추가 권장


## 진단 팁

- curl로 TLS 버전 고정 비교: `curl --tlsv1.2 -v "https://apis.data.go.kr/..."`
- OpenSSL로 핸드셰이크 확인: `openssl s_client -tls1_2 -connect apis.data.go.kr:443 -servername apis.data.go.kr`
- 런타임 OpenSSL 버전 확인: `python -c "import ssl; print(ssl.OPENSSL_VERSION)"`

## 왜 이 설정이 효과적인가

- TLS 1.2 강제: 일부 서버가 TLS 1.3 확장/파라미터를 제대로 처리하지 못해 핸드셰이크가 실패할 수 있습니다.
- `@SECLEVEL=1`: OpenSSL의 기본 보안 레벨 상승으로 차단된 레거시 암호군을 제한적으로 허용합니다.
- `certifi`: 환경마다 다른 시스템 CA 대신 최신 루트 CA 번들을 명시적으로 사용합니다.

## 보안·운영 고려사항

- 전역이 아닌 "해당 세션"에만 완화 설정을 적용하세요.
- 서버가 개선되어 TLS 1.3이 안정화되면 완화 옵션을 제거하세요.
- 네트워크 이슈를 고려해 재시도와 상태/본문 로깅을 적절히 추가하세요.

## 대안 접근

- `httpx`로도 커스텀 TLS 컨텍스트를 구성할 수 있습니다.
- `pycurl`은 curl 스택을 그대로 활용하므로 호환성은 높지만 배포 복잡도가 증가합니다.


