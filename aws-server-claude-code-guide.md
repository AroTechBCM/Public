# ☁️ AWS 서버 개발자를 위한 Claude Code 가이드
## AI 코딩 도구를 처음 도입하는 경력 개발자용

> **대상**: AWS 기반 서버/인프라 개발자 (경력 10년 이상, C#/.NET, AI 코딩 도구 미경험)  
> **프로젝트**: 야구매니저  
> **버전**: 1.0 (2026.03)  
> **작성**: CEO 미니

---

## 목차

1. [왜 AI 코딩 도구인가](#1-왜-ai-코딩-도구인가)
2. [Claude Code 개요](#2-claude-code-개요)
3. [설치 및 세팅](#3-설치-및-세팅)
4. [CLAUDE.md — AI에게 프로젝트 컨텍스트 주입하기](#4-claudemd--ai에게-프로젝트-컨텍스트-주입하기)
5. [AWS 개발 워크플로우](#5-aws-개발-워크플로우)
6. [실전 활용 예시](#6-실전-활용-예시)
7. [AWS MCP 서버 — AI가 AWS 리소스를 직접 다루게 하기](#7-aws-mcp-서버--ai가-aws-리소스를-직접-다루게-하기)
8. [효과적인 프롬프트 작성법](#8-효과적인-프롬프트-작성법)
9. [주의사항과 한계](#9-주의사항과-한계)
10. [레퍼런스](#10-레퍼런스)

---

## 1. 왜 AI 코딩 도구인가

### 10년 경력자에게 AI 도구가 의미하는 것

여러분은 CloudFormation을 YAML 한 줄 안 보고도 머릿속으로 리소스 그래프를 그릴 수 있고, DynamoDB 핫 파티션 문제를 직감으로 잡아내는 분들입니다. AI가 여러분의 판단력을 대체하는 것이 아닙니다.

AI 코딩 도구가 바꾸는 것은 **키보드 위에서 보내는 시간 대비 산출물의 양**입니다:

| 작업 유형 | 기존 방식 | Claude Code 활용 |
|---|---|---|
| IaC 작성 | CDK/SAM 문서 참조하며 타이핑 | 아키텍처 설명 → CDK C# 스택 초안 생성 |
| Lambda 핸들러 | 보일러플레이트 복사 후 로직 작성 | 이벤트 소스 + 비즈니스 로직 설명 → C# 핸들러 생성 |
| IAM 정책 | 최소 권한 원칙에 맞게 수동 작성 | 리소스 접근 패턴 설명 → CDK Grant 메서드 + 최소 권한 정책 생성 |
| 트러블슈팅 | CloudWatch Logs 직접 탐색 | 에러 로그 붙여넣기 → 원인 분석 + 수정안 |
| API 스펙 | OpenAPI YAML 수작업 | 엔드포인트 설계 설명 → 스펙 + 모델 + 검증 코드 생성 |
| 마이그레이션 | 수동 계획서 작성 + 스크립트 | 현행/목표 아키텍처 설명 → 마이그레이션 코드 생성 |
| 코드 리뷰 | PR 열어서 파일별 확인 | 프로젝트 전체 대상 패턴 위반/보안 취약점 스캔 |

### 어떤 것이 아닌지

- ❌ AWS 콘솔을 대체하는 도구 → 아닙니다
- ❌ 자동완성 수준의 도우미 → 그것보다 훨씬 큰 범위입니다
- ❌ 아키텍처 판단을 위임할 대상 → 의사결정은 여전히 여러분의 몫입니다

### 어떤 것인지

- ✅ 프로젝트 폴더 전체(`.sln`, `.csproj`, CDK 스택, Lambda 핸들러)를 컨텍스트로 이해하는 **코딩 에이전트**
- ✅ 기존 CDK 스택, Lambda 함수, DTO 구조를 파악한 상태에서 일관된 코드를 생성
- ✅ 여러 `.cs` 파일을 동시에 수정하고, diff를 보여주고, 승인을 받는 방식
- ✅ MCP를 통해 실제 AWS 리소스에 접근하여 조회/조작 가능

---

## 2. Claude Code 개요

### Claude Code란

Anthropic이 만든 **에이전틱 코딩 도구**입니다. VS Code 확장 또는 CLI로 사용하며, 프로젝트 디렉토리 전체를 컨텍스트로 활용합니다.

일반적인 AI 채팅(Claude 웹, ChatGPT)과의 핵심 차이:

- **파일 시스템 접근**: `.sln`, `.csproj`, `.cs` 파일을 비롯한 프로젝트 전체를 직접 읽고 쓸 수 있음
- **CLAUDE.md**: 프로젝트 루트의 규칙 파일을 자동으로 읽어 매번 컨텍스트 설명 불필요
- **diff 기반 수정**: 모든 파일 수정을 diff로 보여주고 Accept/Reject 선택 가능
- **MCP 연동**: AWS MCP 서버를 통해 실제 AWS 리소스 조회/조작 가능
- **셸 실행**: `dotnet build`, `cdk synth` 등 터미널 명령어를 직접 실행 가능 (사용자 승인 필요)

### 비용

Claude Max 구독 또는 Anthropic API 키가 필요합니다.  
AWS Bedrock을 통한 사용도 가능합니다 (자세한 내용은 [레퍼런스](#10-레퍼런스) 참고).  
세션 중 `/cost`를 입력하면 현재까지의 토큰 사용량과 비용을 확인할 수 있습니다.

---

## 3. 설치 및 세팅

### 3-1. VS Code 확장 설치

1. VS Code에서 `Ctrl + Shift + X` → **"Claude Code"** 검색 → Publisher: **Anthropic** Install
2. 왼쪽 사이드바에 ✱ (Spark) 아이콘 확인
3. 최초 클릭 시 Anthropic OAuth 로그인

> 💡 Unity 개발에 Rider를 쓰시더라도, 서버 프로젝트의 Claude Code 작업은 **VS Code에서** 진행하세요.  
> Claude Code 확장은 현재 VS Code 전용입니다.

### 3-2. CLI 설치 (선택사항이지만 권장)

서버 개발자라면 CLI를 병행하는 것이 편리합니다. 스크립팅, 파이프, `--print` 모드 등 VS Code 확장에서는 제공하지 않는 기능이 있습니다.

```bash
npm install -g @anthropic-ai/claude-code
claude auth login
```

CLI와 VS Code 확장은 동일한 백엔드를 사용합니다. 상황에 따라 혼용 가능합니다.

### 3-3. 프로젝트 폴더 열기

```
File → Open Folder → 서버 프로젝트 루트 (.sln 파일이 있는 디렉토리)
```

모노레포 구조라면 전체 루트를 여세요. Claude가 CDK 스택(C#)과 Lambda 핸들러(C#)를 동시에 참조할 수 있습니다.

### 3-4. .claudeignore

불필요한 파일을 제외하여 토큰 효율을 높이세요:

```
# 빌드 산출물
**/bin/
**/obj/
cdk.out/
.aws-sam/

# IDE 설정
.vs/
.idea/
*.user
*.suo

# 환경 변수 (보안)
.env
.env.*
*.pem
*.key
appsettings.*.json
!appsettings.json
!appsettings.Development.json

# NuGet 패키지 캐시
packages/

# 바이너리
*.zip
*.nupkg
```

> ⚠️ `.env` 파일과 프로덕션 `appsettings` 파일은 반드시 제외하세요.  
> Claude Code는 읽을 수 있는 모든 파일을 컨텍스트로 활용합니다.

---

## 4. CLAUDE.md — AI에게 프로젝트 컨텍스트 주입하기

### CLAUDE.md가 하는 일

프로젝트 루트에 `CLAUDE.md`를 놓으면, Claude Code가 **모든 대화의 시작 시점에 이 파일을 자동으로 읽습니다**. 매번 "우리는 CDK v2를 C#로 쓰고, Lambda도 .NET 8이고..." 같은 설명을 반복할 필요가 없습니다.

이것은 **AI에 대한 프로젝트 온보딩 문서**입니다.  
새로운 팀원이 합류했을 때 읽게 할 문서를 떠올리세요. 그것과 비슷합니다.

### 야구매니저 서버 프로젝트용 CLAUDE.md 예시

```markdown
# 야구매니저 — 서버 프로젝트 규칙

## 프로젝트 정보
- 서비스명: 야구매니저 백엔드
- 언어: C# 12 / .NET 8
- IaC: AWS CDK v2 (C#)
- Lambda 런타임: dotnet8 (Managed Runtime)
- 리전: ap-northeast-2 (서울)
- 계정 구조: dev / staging / prod (AWS Organizations)

## 솔루션 구조
```
BaseballManager.Server/
├── BaseballManager.Server.sln
│
├── src/
│   ├── BaseballManager.Cdk/               ← CDK 인프라 (C#)
│   │   ├── Program.cs
│   │   ├── Stacks/
│   │   │   ├── NetworkStack.cs
│   │   │   ├── DatabaseStack.cs
│   │   │   ├── ApiStack.cs
│   │   │   ├── QueueStack.cs
│   │   │   └── MonitoringStack.cs
│   │   └── Constructs/                    ← 재사용 가능한 Construct
│   │
│   ├── BaseballManager.Functions/          ← Lambda 핸들러
│   │   ├── Handlers/
│   │   │   ├── Player/
│   │   │   │   ├── GetPlayerHandler.cs
│   │   │   │   ├── UpdatePlayerHandler.cs
│   │   │   │   └── ...
│   │   │   ├── Match/
│   │   │   ├── Economy/
│   │   │   └── Team/
│   │   ├── Startup.cs                     ← DI 등록
│   │   └── BaseballManager.Functions.csproj
│   │
│   ├── BaseballManager.Core/              ← 비즈니스 로직 (순수 C#)
│   │   ├── Services/
│   │   ├── Models/
│   │   ├── Interfaces/
│   │   └── BaseballManager.Core.csproj
│   │
│   └── BaseballManager.Infrastructure/    ← AWS SDK 래퍼, DynamoDB 접근
│       ├── Repositories/
│       ├── Clients/
│       └── BaseballManager.Infrastructure.csproj
│
└── tests/
    ├── BaseballManager.Core.Tests/
    ├── BaseballManager.Functions.Tests/
    └── BaseballManager.Integration.Tests/
```

## 아키텍처 개요
- API: API Gateway (HTTP API) → Lambda
- 인증: Cognito User Pool + JWT
- 데이터: DynamoDB (싱글 테이블 디자인)
- 파일: S3 + CloudFront (에셋 번들, 정적 리소스)
- 큐: SQS (매치 시뮬레이션 요청, 보상 처리)
- 이벤트: EventBridge (일일 리셋, 시즌 전환 등 스케줄)
- 캐시: ElastiCache (Redis, 랭킹/리더보드)
- 모니터링: CloudWatch + X-Ray

## DynamoDB 설계
- 싱글 테이블 패턴 사용
- 테이블명: BaseballManager-{stage}
- PK/SK 패턴:
  - 유저: PK=USER#{userId}, SK=PROFILE
  - 선수: PK=USER#{userId}, SK=PLAYER#{playerId}
  - 구단: PK=USER#{userId}, SK=TEAM#ROSTER
  - 매치: PK=MATCH#{matchId}, SK=META
  - 랭킹: PK=SEASON#{seasonId}, SK=RANK#{score}#{userId}
- GSI1: GSI1PK/GSI1SK (역인덱스, 선수 전체 목록 등)
- TTL: ExpiresAt 필드 (매치 기록 90일 보관)

## C# 코드 컨벤션
- 네이밍: Microsoft C# 코딩 규칙 준수
  - private 필드: _camelCase
  - public 프로퍼티: PascalCase
  - 메서드: PascalCase
  - 인터페이스: IPrefix
  - 상수: PascalCase (C# 관례)
- DI: Microsoft.Extensions.DependencyInjection (Lambda Annotations 모델)
- 비동기: async/await + CancellationToken 전파 필수
- null 안전: nullable reference types 활성 (<Nullable>enable</Nullable>)
- 직렬화: System.Text.Json (Newtonsoft 사용하지 않음)
- 로깅: AWS Lambda Powertools for .NET (ILogger)
- 검증: FluentValidation
- 매핑: Mapster (AutoMapper 사용하지 않음)

## Lambda 컨벤션
- 핸들러 구조: Lambda Annotations 프레임워크 사용
  - [LambdaFunction] 어트리뷰트로 핸들러 선언
  - [HttpApi] 어트리뷰트로 API Gateway 라우트 매핑
  - Startup.cs에서 DI 등록
- SDK 클라이언트는 DI를 통해 Singleton으로 주입 (콜드 스타트 최적화)
- 환경변수: TABLE_NAME, STAGE, REGION은 CDK에서 주입
- 타임아웃: API 핸들러 30초, 백그라운드 처리 900초
- 메모리: API 핸들러 512MB, 시뮬레이션 1024MB
- Native AOT: 콜드 스타트 최적화가 필요한 핸들러에 선택적 적용

## CDK 컨벤션 (C#)
- NuGet 패키지: Amazon.CDK.Lib
- 스택 분리: NetworkStack, DatabaseStack, ApiStack, QueueStack, MonitoringStack
- Construct 단위로 재사용 가능하게 설계
- 환경별 설정: cdk.json context + C# record로 타입 안전하게 관리
- Lambda 빌드: BundlingOptions + Amazon.Lambda.Tools
- 리소스 네이밍: {서비스명}-{스택}-{리소스}-{stage}
- 태깅: Project, Stage, Owner 태그 필수
- Removal Policy: prod는 RETAIN, dev/staging은 DESTROY

## API 설계 규칙
- RESTful: /api/v1/{resource}
- 인증: Authorization 헤더에 Cognito JWT
- 페이지네이션: cursor 기반 (DynamoDB LastEvaluatedKey를 Base64 인코딩)
- 요청: record DTO + FluentValidation
- 응답: ApiResponse<T> 래퍼 (data, meta.requestId)
- 에러: ProblemDetails 표준 형식 (RFC 7807)
- Rate Limit: API Gateway 스테이지 레벨 스로틀링

## 보안 규칙
- IAM: 최소 권한 원칙 (Lambda별 개별 역할, CDK의 Grant* 메서드 적극 활용)
- 시크릿: Secrets Manager 사용 (하드코딩, appsettings에 넣기 금지)
- 암호화: DynamoDB 서버사이드 암호화 활성
- VPC: Redis만 VPC 내, Lambda는 VPC 밖 (콜드 스타트 고려)
- WAF: API Gateway에 WAF 연결

## 테스트
- 프레임워크: xUnit + Moq + FluentAssertions
- AWS 서비스 목: LocalStack 또는 AWS SDK 목킹
- 테스트 위치: tests/ 하위에 프로젝트별 미러 구조
- 커버리지 목표: 핵심 비즈니스 로직(Core) 80% 이상

## 응답 규칙
- 모든 응답, 주석, 문서는 한국어
- 코드 내 변수명/메서드명은 영어, XML 주석은 한국어
- 새 파일 생성 시 파일 상단에 XML summary 주석 포함
```

### CLAUDE.md 관리 팁

- **Git에 커밋**하여 팀원 모두가 동일한 규칙을 공유하세요
- DynamoDB 테이블 설계, API 스펙 같은 **핵심 설계 결정**을 담아두면 AI가 일관된 코드를 생성합니다
- 수정 후에는 Claude Code에서 **새 대화를 시작**해야 반영됩니다

---

## 5. AWS 개발 워크플로우

### 기본 작업 흐름

```
VS Code에서 Claude Code 패널 열기 (✱ 아이콘 또는 CLI에서 claude)
        ↓
자연어로 작업 지시 (+ 관련 파일 @멘션)
        ↓
Claude가 기존 코드/IaC를 분석하고 수정안 생성
        ↓
diff 화면에서 변경 사항 검토
        ↓
Accept / Reject 선택
        ↓
로컬 검증 (dotnet build, dotnet test, cdk synth)
        ↓
Git 커밋 → CI/CD 파이프라인
```

### @ 멘션으로 파일 참조하기

채팅 입력창에서 `@`를 입력하면 프로젝트 내 파일 목록이 나타납니다.

```
@src/BaseballManager.Cdk/Stacks/ApiStack.cs 와 
@src/BaseballManager.Functions/Handlers/Player/GetPlayerHandler.cs 를 참고해서
선수 업데이트 API (PUT /api/v1/players/{playerId})를 추가해줘.
CDK 라우트 + Lambda 핸들러 + 요청/응답 DTO + xUnit 테스트 모두 만들어줘.
```

Claude Code는 기존 CDK 스택의 Construct 패턴, Lambda 핸들러의 DI 구조, DTO 모델을 분석한 뒤 일관된 형태로 새 코드를 생성합니다.

### CLI에서의 활용

```bash
# 프로젝트 디렉토리에서 Claude Code 시작
cd ~/projects/BaseballManager.Server
claude

# 비대화형 모드 (CI/CD나 스크립트에서 활용)
claude --print "이 솔루션의 프로젝트 참조 그래프를 분석해줘"

# 특정 파일에 대한 질문
claude --print "src/BaseballManager.Infrastructure/Repositories/PlayerRepository.cs의 DynamoDB 쿼리 패턴에 핫 파티션 위험이 있는지 분석해줘"
```

---

## 6. 실전 활용 예시

### 예시 1: 새 API 엔드포인트 추가

```
가챠(뽑기) API를 추가해줘.

엔드포인트:
- POST /api/v1/gacha/pull (단일 뽑기)
- POST /api/v1/gacha/pull-multi (10연차)

비즈니스 로직:
- 재화(다이아) 잔액 확인 → 차감 → 뽑기 실행 → 결과 저장
- 등급별 확률: N:60%, R:25%, SR:10%, SSR:4%, UR:1%
- 천장: SSR 이상 미획득 80연차 시 SSR 보장
- 10연차 할인: 300*10=3000 → 2700 다이아

DynamoDB:
- 유저 재화: PK=USER#{userId}, SK=CURRENCY
- 뽑기 이력: PK=USER#{userId}, SK=GACHA#{timestamp}
- 천장 카운터: PK=USER#{userId}, SK=GACHA#PITY

구현 위치:
- 서비스: BaseballManager.Core/Services/GachaService.cs (IGachaService)
- 리포지토리: BaseballManager.Infrastructure/Repositories/GachaRepository.cs
- 핸들러: BaseballManager.Functions/Handlers/Economy/GachaPullHandler.cs
- DTO: BaseballManager.Core/Models/Gacha/ 에 요청/응답 record
- 검증: FluentValidation으로 요청 검증
- TransactWriteItems로 재화 차감 + 선수 추가 + 이력 저장을 원자적으로 처리

기존 @src/BaseballManager.Functions/Handlers/Player/GetPlayerHandler.cs 의 
Lambda Annotations 패턴을 따라줘.
CDK 라우트 + 핸들러 + 서비스 + 리포지토리 + xUnit 테스트 모두 만들어줘.
```

### 예시 2: CDK 스택 작성

```
매치 시뮬레이션용 비동기 처리 인프라를 CDK C#으로 만들어줘.

아키텍처:
- API Gateway → Lambda (요청 접수, SQS 전송) → SQS → Lambda (시뮬레이션 실행)
- DLQ 설정 (3회 실패 시)
- 시뮬레이션 Lambda: 메모리 1024MB, 타임아웃 300초
- SQS: 가시성 타임아웃 360초, 배치 사이즈 1

CDK:
- @src/BaseballManager.Cdk/Stacks/DatabaseStack.cs 에서 테이블 참조 가져오기
- @src/BaseballManager.Cdk/Stacks/ApiStack.cs 의 Construct 패턴 따르기
- MatchStack으로 별도 스택 분리
- BundlingOptions로 Lambda .NET 빌드 설정 포함
- MonitoringStack에 DLQ 메시지 수 CloudWatch 알람 추가
```

### 예시 3: DynamoDB 접근 패턴 설계

```
시즌 랭킹 시스템의 DynamoDB 접근 패턴을 설계해줘.

요구사항:
- 시즌별 전체 유저 랭킹 조회 (상위 100명)
- 특정 유저의 시즌 순위 조회
- 시즌 종료 시 보상 지급 대상 조회 (상위 10%)
- 실시간 점수 업데이트

PK/SK 패턴, GSI 설계, C# 모델 클래스, 리포지토리 구현을 포함해줘.
핫 파티션 위험이 있는 패턴은 피하고, Write Sharding이 필요하면 그 방법도.
기존 @src/BaseballManager.Infrastructure/Repositories/PlayerRepository.cs 의 
IAmazonDynamoDB 사용 패턴을 따라줘.
```

### 예시 4: Lambda 성능 최적화

```
@src/BaseballManager.Functions/Handlers/Match/SimulateMatchHandler.cs 를 분석해서 
성능 최적화해줘.

확인할 것:
- DI를 통한 AmazonDynamoDBClient Singleton 주입이 되어 있는지
- 불필요한 DynamoDB 호출 (N+1 쿼리 등)
- BatchGetItem/BatchWriteItem으로 묶을 수 있는 호출
- System.Text.Json 직렬화 설정 최적화 (Source Generator 적용 가능 여부)
- 콜드 스타트 개선 여지 (Native AOT 적용 가능 여부)
- X-Ray 서브세그먼트 설정
- 불필요한 async state machine 생성 (ValueTask 적용 가능 여부)
- string 연결 → StringBuilder 또는 string interpolation

최적화 전후 예상 영향도 설명해줘.
```

### 예시 5: 트러블슈팅

```
프로덕션에서 아래 에러가 간헐적으로 발생해:

Amazon.DynamoDBv2.AmazonDynamoDBException: 
  Rate exceeded for table BaseballManager-prod
  RequestId: abc123
  StatusCode: 400
  ErrorCode: ProvisionedThroughputExceededException
  
  at Amazon.Runtime.Internal.HttpErrorResponseExceptionHandler.HandleExceptionStream(...)
  at BaseballManager.Infrastructure.Repositories.RankingRepository.UpdateScoreAsync(...)
  at BaseballManager.Core.Services.RankingService.ProcessMatchResultAsync(...)

발생 시간: 매일 21:00~22:00 KST (피크 시간대)
관련 파티션: PK=SEASON#2026-1, SK=RANK#*

현재 설정:
- 테이블 모드: 프로비저닝 (읽기 500 RCU, 쓰기 200 WCU)
- Auto Scaling: 타겟 70%, 쿨다운 60초

@src/BaseballManager.Cdk/Stacks/DatabaseStack.cs 와 
@src/BaseballManager.Infrastructure/Repositories/RankingRepository.cs 를 분석해서:
1. 근본 원인 분석
2. 단기 해결책 (즉시 적용 가능)
3. 장기 해결책 (아키텍처 개선)
을 제시해줘.
```

### 예시 6: IAM 정책 최소화

```
@src/BaseballManager.Cdk/Stacks/ApiStack.cs 에서 
Lambda 함수들에 부여된 IAM 정책을 분석해줘.

확인할 것:
- GrantFullAccess 같은 과도한 권한이 있는지
- Resource가 * 로 되어 있는 곳이 있는지
- 실제 핸들러 코드에서 사용하는 DynamoDB 액션만 허용하고 있는지

각 Lambda별로 CDK의 Grant* 메서드(GrantReadData, GrantWriteData 등)를 
최대한 활용하고, 커스텀 정책이 필요한 경우만 PolicyStatement으로 작성해줘.
```

### 예시 7: 테스트 코드 생성

```
@src/BaseballManager.Core/Services/GachaService.cs 에 대한 
단위 테스트를 작성해줘.

테스트 케이스:
- 정상 단일 뽑기 (재화 충분)
- 재화 부족 시 InsufficientCurrencyException 발생
- 천장 도달 시 SSR 보장
- 10연차 할인 가격 검증
- DynamoDB TransactWriteItems 실패 시 예외 전파 확인
- 확률 분포 검증 (10만회 시뮬레이션, 오차 범위 ±1%)

xUnit + Moq + FluentAssertions 사용,
기존 @tests/BaseballManager.Core.Tests/ 의 패턴을 따라줘.
IGachaRepository를 Moq으로 목킹해줘.
```

### 예시 8: 기존 핸들러를 Lambda Annotations로 마이그레이션

```
@src/BaseballManager.Functions/Handlers/Player/ 폴더의 핸들러들이
아직 수동 직렬화 + FunctionHandler 패턴으로 되어 있어.

이걸 Lambda Annotations 프레임워크 패턴으로 마이그레이션해줘:
- [LambdaFunction] + [HttpApi] 어트리뷰트 사용
- Startup.cs에 DI 등록
- 입력은 record DTO로, 출력은 ApiResponse<T>로 통일
- FluentValidation 검증 추가
- 기존 비즈니스 로직은 변경하지 않고 구조만 변환

@src/BaseballManager.Functions/Handlers/Economy/GachaPullHandler.cs 를 
목표 패턴의 레퍼런스로 참고해줘.
변환 전후 diff를 보여주고, 기존 테스트가 깨지지 않는지도 확인해줘.
```

---

## 7. AWS MCP 서버 — AI가 AWS 리소스를 직접 다루게 하기

### MCP란

MCP(Model Context Protocol)는 AI 도구와 외부 시스템을 연결하는 프로토콜입니다.  
**AWS MCP 서버**를 연결하면 Claude Code가 실제 AWS 리소스를 조회하고 조작할 수 있습니다.

즉, "dev 테이블의 GSI 목록을 보여줘"라고 말하면 Claude가 **실제 DynamoDB를 조회**합니다.

### AWS 공식 MCP 서버 목록

AWS Labs에서 공식 제공하는 MCP 서버들입니다:

| MCP 서버 | 용도 |
|---|---|
| `awslabs.core-mcp-server` | 범용 AWS API (모든 서비스 접근) |
| `awslabs.dynamodb-mcp-server` | DynamoDB 테이블 조회/조작 |
| `awslabs.lambda-mcp-server` | Lambda 함수 관리, 로그 조회 |
| `awslabs.cloudformation-mcp-server` | CloudFormation/CDK 문서, 검증 |
| `awslabs.aws-serverless-mcp-server` | 서버리스 패턴, SAM/CDK 가이드 |
| `awslabs.iam-mcp-server` | IAM 정책 분석/생성 |
| `awslabs.aws-docs-mcp-server` | AWS 공식 문서 검색 |

### 설치 방법

VS Code 터미널(`` Ctrl + ` `` 또는 상단 메뉴 **터미널 → 새 터미널**)에서:

**DynamoDB MCP (읽기 전용 권장):**
```bash
claude mcp add awslabs.dynamodb-mcp-server -s project \
  -e AWS_PROFILE=dev \
  -e AWS_REGION=ap-northeast-2 \
  -e DDB-MCP-READONLY=true \
  -- uvx awslabs.dynamodb-mcp-server@latest
```

**Lambda MCP:**
```bash
claude mcp add awslabs.lambda-mcp-server -s project \
  -e AWS_PROFILE=dev \
  -e AWS_REGION=ap-northeast-2 \
  -e FUNCTION_PREFIX="BaseballManager-dev-" \
  -- uvx awslabs.lambda-mcp-server@latest
```

**서버리스 패턴 MCP:**
```bash
claude mcp add awslabs.aws-serverless-mcp-server -s project \
  -- uvx awslabs.aws-serverless-mcp-server@latest
```

**AWS 문서 MCP:**
```bash
claude mcp add awslabs.aws-docs-mcp-server -s project \
  -- uvx awslabs.aws-docs-mcp-server@latest
```

### 환경별 프로파일 분리

```
# ~/.aws/config
[profile dev]
region = ap-northeast-2
output = json

[profile staging-readonly]
region = ap-northeast-2
output = json

[profile prod-readonly]
region = ap-northeast-2
output = json
```

> ⚠️ **중요**: prod 환경은 반드시 **읽기 전용** 프로파일을 사용하세요.  
> DynamoDB MCP는 `-e DDB-MCP-READONLY=true` 를 반드시 설정하세요.

### MCP 설정을 Git으로 관리하기

`-s project` 옵션으로 추가하면 프로젝트 루트에 `.mcp.json`이 생성됩니다.

```bash
git add .mcp.json
git commit -m "Add AWS MCP server configuration"
```

### 활용 예시

```
DynamoDB MCP를 사용해서 dev 환경 BaseballManager-dev 테이블의
GSI1 인덱스 소모 용량을 확인해줘.
핫 파티션이 의심되는 파티션 키가 있는지 분석해줘.
```

```
Lambda MCP로 BaseballManager-dev-SimulateMatch 함수의
최근 1시간 에러 로그를 조회해줘.
에러 패턴을 분류하고 가장 빈번한 에러의 원인을 분석해줘.
```

---

## 8. 효과적인 프롬프트 작성법

### 원칙: 아키텍처 리뷰를 요청하듯 지시하라

여러분이 시니어 동료에게 설계 리뷰를 요청할 때처럼, **제약 조건과 맥락을 명확히** 전달하세요.

### 나쁜 프롬프트 vs 좋은 프롬프트

**❌ 나쁨:**
```
SQS 큐 만들어줘
```

**✅ 좋음:**
```
매치 시뮬레이션 결과를 비동기 처리하는 SQS 큐를 CDK C#으로 만들어줘.

요구사항:
- 표준 큐 (FIFO 불필요)
- DLQ: 3회 실패 시 이동
- 메시지 보존: 7일
- 가시성 타임아웃: 360초 (시뮬레이션 Lambda 타임아웃 300초 + 여유)
- 암호화: AWS managed key (SQS)

Consumer Lambda:
- 배치 사이즈: 1 (시뮬레이션 1건당 처리 시간이 길어서)
- ReservedConcurrentExecutions: 10 (DynamoDB WCU 고려)

기존 @src/BaseballManager.Cdk/Stacks/QueueStack.cs 패턴을 따르고,
MonitoringStack에 DLQ ApproximateNumberOfMessagesVisible 알람도 추가해줘.
```

### 프롬프트 체크리스트

1. **무엇을** — 만들거나 수정할 대상
2. **왜** — 비즈니스 맥락 또는 기술적 이유
3. **제약 조건** — 성능, 비용, 보안, 호환성
4. **참조** — 기존 코드 패턴 (`@` 멘션)
5. **산출물** — CDK 스택 + 핸들러 + 서비스 + 테스트 등 원하는 범위 명시

### 점진적 작업 패턴

```
1단계: "매치 시뮬레이션 서비스의 아키텍처를 설계해줘. 코드 없이 설계 문서만."
2단계: "CDK 스택을 만들어줘."
3단계: "Core 프로젝트에 서비스와 모델을 구현해줘."
4단계: "Infrastructure에 리포지토리를 구현해줘."
5단계: "Functions에 Lambda 핸들러를 만들어줘."
6단계: "xUnit 테스트를 작성해줘."
```

---

## 9. 주의사항과 한계

### Git은 생명줄입니다

Claude Code는 파일을 직접 수정합니다.

- **작업 전 반드시 커밋**하세요
- 대규모 변경은 별도 브랜치에서 진행하세요
- IaC 변경은 `cdk diff`로 반드시 검증하세요

### 절대 하지 말 것

| 위험 행위 | 이유 |
|---|---|
| prod 프로파일로 MCP 쓰기 모드 연결 | 실수로 프로덕션 데이터 변경 가능 |
| `.env` / `appsettings.Production.json`을 .claudeignore에서 빠뜨리기 | 시크릿이 AI 컨텍스트에 포함됨 |
| IaC 변경을 diff 검토 없이 Accept | 의도치 않은 리소스 삭제 가능 |
| `cdk deploy`를 Claude에게 직접 실행시키기 | 예상치 못한 인프라 변경 |
| 비용 관련 설정(Provisioned Capacity, Auto Scaling)을 검증 없이 적용 | 과금 폭발 |

### AI가 모르는 것들

- 실제 트래픽 패턴과 피크 타임의 구체적 수치
- AWS 계정별 서비스 할당량(quotas)과 실제 사용량
- .NET Lambda의 실제 콜드 스타트 시간 (메모리/패키지 크기에 따라 크게 다름)
- 현재 프로덕션에서 발생 중인 인시던트 상황
- 비용 최적화의 비즈니스적 트레이드오프 (RI vs On-Demand, Provisioned vs PAY_PER_REQUEST)
- 조직 내부의 컴플라이언스 요구사항

### AI 생성 C# 코드의 검토 포인트

1. **IAM 권한**: `GrantFullAccess` 사용 여부, Resource가 `*`인 PolicyStatement
2. **DynamoDB**: 핫 파티션 위험, Scan 사용 여부, 적절한 인덱스 활용
3. **Lambda**: 타임아웃/메모리 설정, SDK 클라이언트 Singleton 주입 여부
4. **비동기**: `async void` 사용 금지, CancellationToken 전파 여부
5. **null 안전성**: nullable reference types 경고 무시하지 않았는지
6. **직렬화**: System.Text.Json 설정, record 불변성 유지
7. **비용**: Provisioned 값, Auto Scaling 설정이 합리적인지

### 비용 감각을 유지하세요

Claude Code 사용 자체의 토큰 비용뿐 아니라, AI가 제안하는 AWS 아키텍처의 비용도 주의하세요. Claude는 "기술적으로 가장 좋은 구조"를 제안하는 경향이 있어, 비용 대비 효율이 떨어지는 설계가 나올 수 있습니다. 항상 비용 추정을 함께 요청하세요:

```
위 아키텍처를 월 100만 요청 기준으로 예상 비용을 산출해줘.
Lambda(.NET 8, 512MB), DynamoDB, SQS, API Gateway 각각 분리해서.
```

---

## 10. 레퍼런스

### 공식 문서

- Claude Code 공식 문서: [https://code.claude.com/docs](https://code.claude.com/docs)
- VS Code 확장: [https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code)
- CLAUDE.md 작성 가이드: [https://code.claude.com/docs/en/claude-md](https://code.claude.com/docs/en/claude-md)

### AWS + .NET

- AWS Lambda C# 개발 가이드: [https://docs.aws.amazon.com/lambda/latest/dg/lambda-csharp.html](https://docs.aws.amazon.com/lambda/latest/dg/lambda-csharp.html)
- AWS CDK C# 가이드: [https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-csharp.html](https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-csharp.html)
- CDK로 .NET Lambda 빌드/배포: [https://docs.aws.amazon.com/lambda/latest/dg/csharp-package-cdk.html](https://docs.aws.amazon.com/lambda/latest/dg/csharp-package-cdk.html)
- AWS MCP 서버: [https://github.com/awslabs/mcp](https://github.com/awslabs/mcp)
- AWS Bedrock에서 Claude Code 사용: [https://aws.amazon.com/blogs/machine-learning/claude-code-deployment-patterns-and-best-practices-with-amazon-bedrock/](https://aws.amazon.com/blogs/machine-learning/claude-code-deployment-patterns-and-best-practices-with-amazon-bedrock/)

### 사내 자료

- 기획팀 Claude Code 매뉴얼: `vscode.md` (비개발자용)
- Unity 개발자 가이드: `unity-claude-code-guide.md`
- 프로젝트 CLAUDE.md: 프로젝트 루트에 위치

---

> 📝 이 가이드에 대한 질문이나 개선 사항은 CEO 미니에게 전달해주세요.  
> AI 도구 도입 과정에서 발견한 팁이나 문제가 있으면 팀 내 공유 부탁드립니다.
