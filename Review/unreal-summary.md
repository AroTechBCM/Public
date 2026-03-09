# 언리얼 개발자를 위한 Claude Code 가이드 요약

> 원본: `unreal-claude-code-guide.md`
> 대상: Unreal Engine C++ 개발자 (경력 5년+)
> 버전: 1.0 (2026.03)

---

## 핵심 개념

AI가 판단력을 대체하는 것이 아니라 **작업의 밀도**를 바꾼다. `.h`/`.cpp` 쌍을 동시에 수정하고, 클래스 계층·리플리케이션 설정·GAS 구조를 파악한 상태에서 코드를 생성.

---

## 설치

1. VS Code → `Ctrl+Shift+X` → **Claude Code (Anthropic)** 설치
2. CLI (선택): `npm install -g @anthropic-ai/claude-code`
3. **`.uproject` 파일이 있는 프로젝트 루트** 열기 (Source/ 하위 폴더만 열면 안 됨)
4. `.claudeignore` 설정 (Binaries/, Intermediate/, Content/, *.uasset 등 제외)

> 💡 Rider/VS를 메인 IDE로 써도 Claude Code 작업은 VS Code에서 진행

---

## CLAUDE.md 핵심 내용 (UE 프로젝트)

| 항목 | 내용 |
|---|---|
| 엔진 | Unreal Engine 5.4 |
| 게임플레이 | GAS (GameplayAbilitySystem) |
| 네트워크 | Dedicated Server, Push Model 리플리케이션 |
| UI | UMG + MVVM |
| 입력 | Enhanced Input System |
| 컨테이너 | TArray/TMap/TSet (STL 금지) |
| 포인터 | TObjectPtr<> (raw pointer 지양) |

---

## 3개 도구 병행 패턴

| 도구 | 역할 |
|---|---|
| VS Code (Claude Code) | 코드 생성, .h/.cpp 동시 편집 |
| Unreal Editor | 컴파일, BP 작업, 플레이 테스트 |
| Rider / Visual Studio | 디버깅, 브레이크포인트, 심볼 탐색 |

---

## 실전 프롬프트 예시 (8가지)

| 작업 | 요약 |
|---|---|
| 새 시스템 | UActorComponent + GAS 연동 + Automation 테스트 |
| 리팩토링 | 500줄 함수 → Resolver 구조체 분리 |
| 크래시 디버깅 | 콜스택 붙여넣기 → TObjectPtr 전환 |
| GAS 어빌리티 | GameplayAbility + GameplayEffect + RPC |
| 리플리케이션 | FFastArraySerializer + Push Model + OnRep |
| 에디터 도구 | CSV 임포트 유틸리티 (EditorUtilityWidget) |
| 성능 최적화 | Tick 분석, TArray 재할당 제거, SCOPE_CYCLE_COUNTER |
| 테스트 생성 | IMPLEMENT_SIMPLE_AUTOMATION_TEST 매크로 |

---

## Unreal MCP

| 프로젝트 | 특징 |
|---|---|
| UnrealClaude (Natfii) | UE 5.7, 에디터 내장 채팅 |
| unreal-mcp (chongdashu) | 범용 에디터 제어 |
| ue-codegraph-mcp | C++ 코드 인덱싱, 콜 그래프 분석 |

```bash
# codegraph-mcp 활용 예
SetHP를 호출하는 모든 함수를 찾아줘
ABMMatchSimulator의 상속 계층을 보여줘
```

---

## AI 생성 UE C++ 검토 포인트

1. **UObject 라이프사이클** — GC 댕글링 포인터, TObjectPtr 사용
2. **UPROPERTY 스펙파이어** — Replicated 누락, EditAnywhere/EditDefaultsOnly 혼동
3. **GetLifetimeReplicatedProps** — 리플리케이션 프로퍼티 등록 누락
4. **STL 사용 여부** — std::vector, std::string → TArray, FString으로 교체
5. **예외처리** — try/catch → check/ensure/verify
6. **에디터 전용 코드** — `#if WITH_EDITOR` 가드 누락

---

## 주의사항

- 작업 전 반드시 소스 컨트롤 커밋
- 코드 변경이 BP 호환성을 깨뜨릴 수 있음 (.uasset은 Claude가 수정 안 함)
- Unreal MCP는 대부분 실험 단계 — 중요 맵 작업 전 커밋 필수
