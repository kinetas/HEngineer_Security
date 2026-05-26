# /scan

정적 분석 및 의존성 스캔 도구를 실행하여 취약점을 탐지한다.
KISA 체크리스트와 무관하게 도구 결과를 그대로 출력한다. 빠른 취약점 탐지용.

## 실행 방법

`/scan [target]`
- target: 스캔할 경로 (생략 시 현재 디렉터리)

## 절차

1. **언어/환경 감지**
   - package.json → Node.js
   - requirements.txt / pyproject.toml → Python
   - Dockerfile / 이미지 → 컨테이너

2. **도구 실행** (감지된 환경에 맞게 선택)
   - 공통: `semgrep --config p/security-audit [target]`
   - Python: `bandit -r [target]`
   - Node.js: `npm audit --json`
   - Python 패키지: `pip-audit`
   - 컨테이너/파일시스템: `trivy fs [target]`

3. **결과 분류**
   - CVSS 점수 및 severity 기준으로 CRITICAL / HIGH / MEDIUM / LOW 분류
   - policy/default.json의 임계값 적용

4. **미니PC API 전송** (API_BASE_URL 설정 시)
   - `POST /scan/result`로 결과 전송

5. **결과 출력**

## 출력 형식

```
[SCAN RESULT] target: ./src
──────────────────────────
CRITICAL (0)  HIGH (2)  MEDIUM (5)  LOW (3)

[HIGH] CVE-2024-XXXX — lodash 4.17.15
  의존성: package.json > lodash
  조치: 4.17.21 이상으로 업그레이드

[HIGH] semgrep: sql-injection — src/db.js:42
  패턴: 사용자 입력이 SQL 쿼리에 직접 삽입됨
  조치: parameterized query 사용
...
```
