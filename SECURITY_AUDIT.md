# 보안 점검 보고서 — maxun

- 점검일: 2026-11-08 (2026-11-08 후속 확인으로 등급 조정)
- 점검자: Won의 Instinct 에이전트 (자동 점검, 코드 수정 없이 문서화만 수행)
- 종합 등급: 🟡 참고 (포크 소유자 관점)
- 공개 범위: public

## 점검 방법
1. 전체 git 이력 대상 시크릿 스캔 (모든 브랜치·커밋의 모든 blob): AWS/GitHub/OpenAI/Slack/Google/Stripe/Telegram/Discord 토큰, 개인키 블록, DB 연결 문자열, 하드코딩 비밀번호 패턴, 민감 파일명(.env, .pem, .key 등).
2. 발견 항목은 원문을 열어 실제 비밀인지 테스트 픽스처/플레이스홀더인지 수동 분류.
3. 레포 설정 점검: Dependabot 알림, 브랜치 보호, 기존 SECURITY.md 유무.
4. 의존성 매니페스트 확인 + 주요 패키지(Next.js 등) 잠금 버전의 알려진 취약점 스팟 체크.

> 주의: 이 파일에는 어떤 비밀값도 전문으로 기록하지 않았습니다(마스킹 처리).

## 발견 사항
### 1. [참고] git 이력의 GCP 서비스 계정 개인키 — 업스트림(원본 프로젝트) 소유로 확인
- 이 레포는 `getmaxun/maxun`의 포크이며, 아래 키는 원본 레포의 커밋 `34e37033`(2024-09-17, 업스트림 작성자)에서 유래한 **업스트림 프로젝트의 자산**입니다. 포크 소유자의 GCP 자격증명이 아닙니다.
- 파일: `server/src/routes/service_account_credentials.json` — `type: service_account`, 프로젝트 `maxun12345`, `client_email: maxun-999@maxun12345.iam.gserviceaccount.com`, RSA 개인키 포함. 동일 키가 `integrations-Load More Test.json`에도 존재.
- 2024-10-02 커밋(`3ee30c04`)에서 삭제됐으나 이력에는 남아 있음. 2024년 9월부터 공개됐으므로 **이미 폐기(revoke)됐을 가능성이 높음**.
- 후속 결정(2026-11-08): 포크 소유자는 업스트림에 별도 신고하지 않기로 함 (이미 삭제·폐기된 API 키일 것으로 판단).
- 참고: 기존 `SECURITY.md`는 원본 오픈소스(Maxun OSS)의 신고 정책(support@maxun.dev)을 다룰 뿐이라 덮어쓰지 않고 이 파일(`SECURITY_AUDIT.md`)에 별도 기록함.

### 2. [참고] 그 외 스캔 결과
- DB 연결 문자열 다수: 로컬호스트/도커 서비스 기본값 (`postgres:***@localhost`, `pgbouncer` 등) — 개발용 기본값으로 판단됨.
- 테스트 코드의 API 키 문자열: 픽스처.
- Dependabot 알림 비활성화, 기본 브랜치(develop) 보호 규칙 없음.

## 권고 조치
1. 포크 소유자 측 조치는 선택 사항: 이력 노출이 신경 쓰이면 포크 삭제 후 재포크하거나 `git filter-repo`로 이력 정리 (업스트림이 키를 폐기했다면 실익은 적음).
2. 향후 업스트림 동기화 시 동일 파일이 다시 유입되지 않는지만 확인.
3. Dependabot alerts + security updates 활성화 권장.
