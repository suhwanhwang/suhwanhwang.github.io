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


