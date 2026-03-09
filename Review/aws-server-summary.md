# AWS 서버 개발자를 위한 Claude Code 가이드 요약

> 원본: `aws-server-claude-code-guide.md`
> 대상: AWS 기반 서버/인프라 개발자 (경력 10년+, C#/.NET)
> 버전: 1.0 (2026.03)

---

## 핵심 개념

AI가 판단력을 대체하는 것이 아니라 **키보드 위에서 보내는 시간 대비 산출물의 양**을 바꾼다. 프로젝트 폴더 전체(`.sln`, `.csproj`, CDK 스택, Lambda 핸들러)를 컨텍스트로 이해하는 코딩 에이전트.

---

## 설치

1. VS Code → `Ctrl+Shift+X` → **Claude Code (Anthropic)** 설치
2. CLI (선택): `npm install -g @anthropic-ai/claude-code && claude auth login`
3. 프로젝트 루트 (`.sln` 파일 위치) 폴더 열기

---

## .claudeignore 필수 항목

```
**/bin/  **/obj/  cdk.out/  .env  *.pem  *.key  appsettings.*.json
```

> ⚠️ `.env`와 프로덕션 `appsettings` 파일은 반드시 제외

---

## CLAUDE.md 핵심 내용 (서버 프로젝트)

| 항목 | 내용 |
|---|---|
| 언어/런타임 | C# 12 / .NET 8, Lambda dotnet8 |
| IaC | AWS CDK v2 (C#) |
| 리전 | ap-northeast-2 (서울) |
| DB | DynamoDB 싱글 테이블 디자인 |
| 인증 | Cognito JWT |
| 직렬화 | System.Text.Json (Newtonsoft 금지) |
| DI | Lambda Annotations + Microsoft DI |

---

## 주요 워크플로우

```
작업 지시 (@멘션 활용) → diff 검토 → Accept/Reject → dotnet build/test → cdk synth → Git 커밋
```

---

## 실전 프롬프트 예시 (8가지)

| 작업 | 요약 |
|---|---|
| 새 API 추가 | CDK 라우트 + Lambda 핸들러 + DTO + 테스트 일괄 생성 |
| CDK 스택 작성 | 비동기 처리용 SQS + Lambda 스택 설계 |
| DynamoDB 패턴 | PK/SK 설계, GSI, 핫 파티션 방지 |
| Lambda 최적화 | SDK Singleton, N+1 쿼리, Native AOT |
| 트러블슈팅 | 에러 로그 붙여넣기 → 원인 분석 + 수정안 |
| IAM 최소화 | Grant* 메서드 활용, Resource `*` 제거 |
| 테스트 생성 | xUnit + Moq + FluentAssertions |
| Lambda 마이그레이션 | 수동 핸들러 → Lambda Annotations 전환 |

---

## AWS MCP 서버

| MCP 서버 | 용도 |
|---|---|
| `awslabs.dynamodb-mcp-server` | DynamoDB 조회/조작 (읽기 전용 권장) |
| `awslabs.lambda-mcp-server` | Lambda 관리, 로그 조회 |
| `awslabs.aws-docs-mcp-server` | AWS 공식 문서 검색 |

> ⚠️ prod 환경은 반드시 **읽기 전용** 프로파일 사용

---

## 절대 하지 말 것

- prod 프로파일로 MCP 쓰기 모드 연결
- `.env` / `appsettings.Production.json`을 `.claudeignore`에서 제외
- `cdk deploy`를 Claude에게 직접 실행시키기
- IaC 변경을 diff 검토 없이 Accept

---

## AI가 모르는 것

- 실제 트래픽 패턴과 피크 수치
- AWS 계정별 서비스 할당량
- 비용 최적화의 비즈니스 트레이드오프
- 조직 내부 컴플라이언스 요구사항
