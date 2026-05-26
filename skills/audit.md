# /audit

KISA 체크리스트 기반 심층 보안 감사를 실행한다.
Security AI가 검사 항목을 결정하고 복수의 Security Audit AI에게 병렬 배정한다.

## 실행 방법

`/audit [target] [mode]`
- target: 감사할 경로 (생략 시 현재 디렉터리)
- mode: `recommend` | `pick` | `full` (생략 시 recommend)

## 모드별 흐름

### recommend (기본)
1. 대상 코드를 읽어 언어/프레임워크/구조 파악
2. `GET /kisa/checklist`로 전체 항목 로드 (API_BASE_URL 미설정 시 내장 목록 사용)
3. 코드 특성에 맞는 항목 추천 + 각 항목마다 추천 근거 제시
   ```
   추천 항목 (7개):
   ✓ [KISA-INJ-01] SQL 인젝션  — DB 쿼리 코드 감지됨 (src/db.js)
   ✓ [KISA-AUTH-02] 세션 고정  — express-session 사용 감지됨
   ✓ [KISA-DEP-01] 취약 의존성 — package.json 존재
   ...
   승인하시겠습니까? (y / 항목 번호로 제외 / 추가)
   ```
4. 유저 승인 후 항목 확정

### pick
1. `GET /kisa/checklist`로 전체 항목 목록 출력
2. 유저가 번호로 원하는 항목 선택

### full
1. `GET /kisa/checklist`로 전체 항목 그대로 사용, 확인 없이 진행

## 공통 실행 절차 (항목 확정 후)

1. **Audit AI 수 결정 및 항목 분배**
   - 항목 5개 이하 → Audit AI 1개
   - 항목 6~15개   → Audit AI 2개
   - 항목 16개 이상 → Audit AI 3개 (teamLimit 고려)

2. **병렬 실행**
   - 각 Audit AI: 담당 항목에 맞는 도구 선택 → 코드 직접 읽기 → 항목별 판정
   - 판정: **PASS** / **FAIL** / **PARTIAL** / **N/A**
   - 각 Audit AI: 결과 → `POST /scan/result` → fragment 작성

3. **결과 종합**
   - Security AI가 모든 fragment 수집 → 위험 등급 판정 → 보고서

## 출력 형식

```
[KISA AUDIT RESULT] target: ./src  |  검사 항목: 7개  |  Audit AI: 2개 병렬
──────────────────────────────────────────────────────────────────────────
PASS: 4  FAIL: 2  PARTIAL: 1  N/A: 0

[FAIL] KISA-INJ-01 — SQL 인젝션 방어
  위치: src/api/user.js:88
  근거: `"SELECT * FROM users WHERE id=" + req.params.id`
  조치: parameterized query 사용

[FAIL] KISA-CRYPT-02 — 하드코딩된 비밀값 금지
  위치: config/database.js:5
  근거: `password: "admin1234"` 소스코드에 직접 기재
  조치: process.env.DB_PASSWORD 사용

[PARTIAL] KISA-AUTH-01 — 세션 관리
  근거: 세션 만료 설정은 있으나 세션 고정 공격 방어 미흡
  조치: 로그인 성공 시 session.regenerate() 호출 필요

──────────────────────────────────────────────────────────────────────────
최종 위험 등급: HIGH
```
