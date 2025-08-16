---
layout: post
title: "Python requests로 공공데이터포털 SSL 오류(SSLV3_ALERT_ILLEGAL_PARAMETER) 해결기"
date: 2025-08-16 15:10:00 +0900
categories: internet
tags: [python, requests, tls, openssl, certifi, 공공데이터포털]
---

## 증상

공공데이터포털(`apis.data.go.kr`) API를 Python `requests`로 호출할 때 아래와 같은 SSL 오류가 발생했습니다.

```text
requests.exceptions.SSLError: HTTPSConnectionPool(host='apis.data.go.kr', port=443): Max retries exceeded ...
(Caused by SSLError(SSLError(1, '[SSL: SSLV3_ALERT_ILLEGAL_PARAMETER] ssl/tls alert illegal parameter (_ssl.c:1006)')))
```

### 특징
- 동일 요청이 브라우저나 `curl`에서는 정상 동작
- `requests`만 TLS 핸드셰이크 단계에서 실패

## 원인 추정

- **TLS 버전 협상 문제**: 일부 서버가 TLS 1.3 또는 특정 확장을 안정적으로 처리하지 못함
- **OpenSSL 보안 레벨 상승 영향**: OpenSSL 3 기본 `SECLEVEL`이 높아져 레거시 암호군/해시가 거절될 수 있음
- **클라이언트 기본값 차이**: `curl`과 `requests`가 사용하는 TLS 스택/암호군/확장이 상이

## 재현 코드 (문제 상황)

```python
import requests

url = "https://apis.data.go.kr/1160100/service/GetGeneralProductInfoService/getGoldPriceInfo?..."
requests.get(url, timeout=10).json()  # SSL 오류 발생
```

## 진단 팁

- curl로 TLS 버전 고정 비교: `curl --tlsv1.2 -v "https://..."`
- OpenSSL로 핸드셰이크 확인: `openssl s_client -tls1_2 -connect apis.data.go.kr:443 -servername apis.data.go.kr`
- Python 런타임의 OpenSSL 버전: `python -c "import ssl; print(ssl.OPENSSL_VERSION)"`

## 해결 방법

핵심은 서버가 수용 가능한 TLS 파라미터로 낮춰 호출하는 것입니다.

- TLS 1.2로 고정
- OpenSSL 보안 레벨 완화: `DEFAULT:@SECLEVEL=1`
- 신뢰 저장소 명시: `certifi`의 CA 번들 사용
- (선택) 연결 안정화를 위해 `Connection: close`

## 안전한 구현 예시

도메인 전용 세션을 만들어 필요한 범위에서만 정책을 완화합니다. 전역 설정은 바꾸지 않습니다.

```python
import ssl
import certifi
import requests
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
    headers = {"Accept": "*/*", "Connection": "close"}
    session = requests.Session()
    session.mount("https://", TLS12Adapter())
    try:
        resp = session.get(
            url,
            headers=headers,
            timeout=timeout,
            allow_redirects=True,
            verify=certifi.where(),
        )
        resp.raise_for_status()
        return resp.json()
    finally:
        session.close()
```


## 관련 리소스

- 공공데이터포털: [`https://www.data.go.kr/`](https://www.data.go.kr/)
- Requests 문서: [Advanced Usage — SSL](https://requests.readthedocs.io/en/latest/user/advanced/#ssl-cert-verification)
- Certifi 문서: [`https://certifiio.readthedocs.io/en/latest/`](https://certifiio.readthedocs.io/en/latest/)
- OpenSSL 보안 레벨: [SSL_CTX_set_security_level](https://www.openssl.org/docs/man3.0/man3/SSL_CTX_set_security_level.html)

