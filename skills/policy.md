# /policy

보안 정책을 조회하거나 변경한다.

## 실행 방법

`/policy [action]`
- action: `show` | `set [key] [value]` | `reset`
- 생략 시 show

## 절차

### show
policy/default.json 내용을 사람이 읽기 쉬운 형식으로 출력.

```
[현재 보안 정책]
─────────────────────────────
위험 등급 임계값
  CRITICAL : CVSS 9.0 이상
  HIGH     : CVSS 7.0 ~ 8.9
  MEDIUM   : CVSS 4.0 ~ 6.9
  LOW      : CVSS 4.0 미만

KISA 감사 설정
  필수 항목 미준수 시 FAIL 처리 : 활성화
  권고 항목 미준수 FAIL 기준    : 5건 이상

스캔 도구 설정
  semgrep 룰셋   : p/security-audit
  trivy 심각도   : CRITICAL, HIGH, MEDIUM

API 설정
  API_BASE_URL : (미설정)
  타임아웃     : 30초
```

### set [key] [value]
key 경로(점 표기법)와 값을 받아 policy/default.json 수정.
예: `/policy set risk_thresholds.critical 8.5`

변경 전 현재값과 변경 후 값을 함께 출력하고 확인 요청.

### reset
policy/default.json을 기본값으로 복원. 실행 전 확인 요청.
