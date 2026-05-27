# /cve

CVE ID로 취약점 상세 정보를 NVD 공식 API에서 조회한다.

## 실행 방법

`/cve [CVE-ID]`
- 예: `/cve CVE-2024-1234`

## 절차

### 1. ID 형식 검증

- `CVE-YYYY-NNNNN` 형식(연도 4자리, 숫자 4자리 이상)인지 확인
- 형식 불일치 시: "올바른 CVE ID 형식이 아닙니다 (예: CVE-2024-1234)" 출력 후 종료

### 2. API 키 로드

1. 설치 경로의 `config/.env` 파일에서 `NVD_API_KEY` 읽기
2. 없으면 시스템 환경변수 확인
3. 둘 다 없으면 키 없이 호출 (rate limit 5 req/30s 적용)

### 3. NVD 공식 API 호출

엔드포인트: `https://services.nvd.nist.gov/rest/json/cves/2.0?cveId={CVE-ID}`

- `NVD_API_KEY` 있으면: `apiKey: {NVD_API_KEY}` 헤더 추가 (rate limit 50 req/30s)
- 없으면: 헤더 없이 호출

**요청 딜레이 (연속 조회 시 필수)**

| 상태 | 제한 | 요청 간 최소 대기 |
|------|------|------------------|
| API 키 있음 | 30초당 50회 | **600ms** |
| API 키 없음 | 30초당 5회 | **6000ms** |

- `/cve`를 단발성으로 호출할 때는 딜레이 불필요
- `/cve`가 프로그래밍 방식으로 연속 호출될 경우 각 요청 사이에 위 대기 시간 적용
- HTTP 429 수신 시: 30초 대기 후 1회 재시도, 재시도도 429이면 오류 출력 후 종료

### 4. 응답 파싱

NVD 응답에서 아래 필드 추출:

| 항목 | 경로 |
|------|------|
| CVSS 점수·등급 | `metrics.cvssMetricV31[0]` → `cvssData.baseScore` / `baseSeverity` (없으면 V30 → V2 순 폴백) |
| 공격 벡터 | `cvssData.attackVector` + `cvssData.privilegesRequired` |
| 영향 범위 | `cvssData.confidentialityImpact` / `integrityImpact` / `availabilityImpact` |
| 설명 | `descriptions` → `lang: "en"` 항목 |
| 공개일 | `published` |
| 참조 URL | `references[].url` 전체 목록 |

CVSS 데이터 없는 CVE: 점수·벡터 항목을 "정보 없음"으로 표시하고 나머지는 정상 출력

### 5. 오류 처리

| 상황 | 출력 |
|------|------|
| CVE ID 존재하지 않음 (결과 0건) | "존재하지 않는 CVE ID입니다: {id}" |
| 네트워크 오류 | "NVD API 연결 실패. 네트워크 상태를 확인하세요." |
| HTTP 429 | "NVD API 요청 한도 초과. 잠시 후 다시 시도하세요." + API 키 미설정이면 발급 안내 출력 |
| HTTP 5xx | "NVD API 서버 오류 ({status}). 잠시 후 다시 시도하세요." |

## 출력 형식

```
CVE-2024-1234
──────────────────────────────────────────
CVSS 점수  : 9.8 (CRITICAL)
공격 벡터  : 네트워크 / 인증 불필요
영향 범위  : 기밀성 HIGH · 무결성 HIGH · 가용성 HIGH
설명       : 원격 공격자가 인증 없이 임의 코드를 실행할 수 있음
공개일     : 2024-03-15
참조       :
  https://nvd.nist.gov/vuln/detail/CVE-2024-1234
  https://example.com/vendor-advisory
```

## API 키 안내 (429 오류 또는 키 미설정 시 출력)

```
[안내] NVD API 키를 설정하면 요청 한도가 10배 향상됩니다.
       발급: https://nvd.nist.gov/developers/request-an-api-key
       설정: 설치 경로의 config/.env 파일에 NVD_API_KEY=발급받은키 입력
```
