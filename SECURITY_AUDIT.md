# 보안 점검 보고서 — maxun

- 점검일: 2026-11-08
- 점검자: Won의 Instinct 에이전트 (자동 점검, 코드 수정 없이 문서화만 수행)
- 종합 등급: 🚨 긴급
- 공개 범위: public

## 점검 방법
1. 전체 git 이력 대상 시크릿 스캔 (모든 브랜치·커밋의 모든 blob): AWS/GitHub/OpenAI/Slack/Google/Stripe/Telegram/Discord 토큰, 개인키 블록, DB 연결 문자열, 하드코딩 비밀번호 패턴, 민감 파일명(.env, .pem, .key 등).
2. 발견 항목은 원문을 열어 실제 비밀인지 테스트 픽스처/플레이스홀더인지 수동 분류.
3. 레포 설정 점검: Dependabot 알림, 브랜치 보호, 기존 SECURITY.md 유무.
4. 의존성 매니페스트 확인 + 주요 패키지(Next.js 등) 잠금 버전의 알려진 취약점 스팟 체크.

> 주의: 이 파일에는 어떤 비밀값도 전문으로 기록하지 않았습니다(마스킹 처리).

## 발견 사항
### 1. [긴급] 공개 레포 git 이력에 GCP 서비스 계정 개인키 커밋됨
- 파일: `server/src/routes/service_account_credentials.json` (커밋 `34e37033` 등에 존재, 이후 삭제됐지만 **public 이력에 그대로 노출**)
- 내용: `type: service_account`, 프로젝트 `maxun12345`, `client_email: maxun-999@maxun12345.iam.gserviceaccount.com`, **실제 RSA 개인키 포함**
- 동일 키가 `server/src/workflow-management/integrations/integrations-Load More Test.json`에도 존재
- 위험: 누구나 이력에서 키를 추출해 해당 GCP 프로젝트에 서비스 계정 권한으로 접근 가능. 이미 크롤러에 수집됐을 가능성 있음.
- 참고: 기존 `SECURITY.md`는 원본 오픈소스(Maxun OSS)의 신고 정책(support@maxun.dev)일 뿐, 이 포크의 키 노출 문제는 다루지 않음. 덮어쓰지 않고 이 파일(`SECURITY_AUDIT.md`)에 별도 기록함.

### 2. [참고] 그 외 스캔 결과
- DB 연결 문자열 다수: 로컬호스트/도커 서비스 기본값 (`postgres:***@localhost`, `pgbouncer` 등) — 개발용 기본값으로 판단됨.
- 테스트 코드의 API 키 문자열: 픽스처.
- Dependabot 알림 비활성화, 기본 브랜치(develop) 보호 규칙 없음.

## 권고 조치 (우선순위 순)
1. **즉시** GCP 콘솔(IAM → 서비스 계정 → maxun-999)에서 해당 키 삭제/키 교체. 사용 로그(감사 로그) 확인.
2. 키가 필요 없으면 서비스 계정 자체 비활성화 검토.
3. 이력에서 파일 제거(`git filter-repo`) 또는 레포 재생성. 공개 이력은 이미 노출된 것으로 간주하고 키 폐기가 우선.
4. Dependabot alerts + security updates 활성화 (Settings → Advanced Security).
5. 포크라면 향후 동기화 시 동일 파일이 다시 들어오지 않는지 확인.
