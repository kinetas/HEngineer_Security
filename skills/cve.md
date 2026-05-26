# /cve

CVE ID로 취약점 상세 정보를 조회한다.

## 실행 방법

`/cve [CVE-ID]`
- 예: `/cve CVE-2024-1234`

## 절차

1. **ID 형식 검증**
   - `CVE-YYYY-NNNNN` 형식인지 확인. 아니면 "올바른 CVE ID 형식이 아닙니다" 출력 후 종료.

2. **조회**
   - `$env:API_BASE_URL` 설정 시: `GET /cve/{id}` 호출 (미니PC 로컬 NVD 미러)
   - 미설정 시: NVD 공식 API `https://services.nvd.nist.gov/rest/json/cves/2.0?cveId={id}` 호출

3. **결과 출력**

## 출력 형식

```
CVE-2024-1234
──────────────
CVSS 점수  : 9.8 (CRITICAL)
공격 벡터  : 네트워크 / 인증 불필요
영향 범위  : 기밀성·무결성·가용성 모두 HIGH
영향 패키지: example-lib < 2.1.0
설명       : 원격 공격자가 인증 없이 임의 코드를 실행할 수 있음
권고 조치  : 2.1.0 이상으로 업그레이드
공개일     : 2024-03-15
참조       : https://nvd.nist.gov/vuln/detail/CVE-2024-1234
```
