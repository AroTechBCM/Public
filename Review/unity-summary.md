# Unity 개발자를 위한 Claude Code 가이드 요약

> 원본: `unity-claude-code-guide.md`
> 대상: Unity C# 개발자 (경력 5년+)
> 버전: 1.0 (2026.03)

---

## 핵심 개념

AI가 작업을 대체하는 것이 아니라 **작업의 밀도**를 바꾼다. 프로젝트 폴더 전체를 컨텍스트로 이해하며, 기존 아키텍처·네이밍 컨벤션·의존성을 파악한 상태에서 코드를 생성.

---

## 설치

1. VS Code → `Ctrl+Shift+X` → **Claude Code (Anthropic)** 설치
2. Unity **프로젝트 루트** (Assets, Packages, ProjectSettings 폴더 위치) 열기
3. `.claudeignore` 설정 (Library/, Temp/, *.png, *.fbx 등 제외)

---

## CLAUDE.md 핵심 내용 (Unity 프로젝트)

| 항목 | 내용 |
|---|---|
| 엔진 | Unity 2022.3 LTS (URP) |
| 플랫폼 | Android / iOS |
| DI | VContainer (Zenject 아님) |
| 비동기 | UniTask (Task 사용 금지) |
| Reactive | R3 (UniRx 아님) |
| UI | UI Toolkit (UGUI 아님) |
| namespace | `BaseballManager.[모듈명]` 필수 |

---

## 주요 워크플로우

```
Claude Code (코드 생성/수정) → Accept → Unity 에디터 (리컴파일 확인) → 플레이 테스트
```

VS Code + Unity 에디터를 나란히 띄워두고 작업.

---

## 실전 프롬프트 예시 (8가지)

| 작업 | 요약 |
|---|---|
| 새 시스템 | IInterface + Service + ScriptableObject + EditMode 테스트 |
| 리팩토링 | 메서드 분리, Resolver 클래스 추출, DI 구조 유지 |
| 버그 디버깅 | NullRef 콜스택 붙여넣기 → 근본 원인 분석 |
| 테스트 생성 | NUnit + Unity Test Framework, 확률/천장 검증 |
| ScriptableObject | 커스텀 에디터 포함, 스탯 합계 실시간 표시 |
| 에디터 도구 | CSV 임포트 위젯, EditorUtility.DisplayProgressBar |
| 성능 최적화 | GC Alloc 제거, LINQ → for 루프, StringBuilder |
| AWS 연동 | UniTask 기반 HTTP 클라이언트, 재시도/JWT 갱신 |

---

## Unity MCP

```bash
claude mcp add --scope user --transport stdio unity-mcp -- uvx --python ">=3.11" unity-mcp-server@latest
```

**가능한 작업**: GameObject 생성/삭제, 컴포넌트 추가, 머티리얼 적용, 씬 관리, 테스트 실행

> ⚠️ 아직 베타 - 프로덕션 씬 작업 전 반드시 Git 커밋

---

## AI 생성 코드 검토 포인트

1. **GC Alloc** — LINQ, string 연결, 람다 캡처 매 프레임 발생 여부
2. **스레드 안전성** — UniTask와 메인 스레드 접근
3. **null 안전성** — `?.`와 `??` 적절 사용
4. **네이밍** — 기존 컨벤션 일치 여부
5. **의존성 방향** — 아키텍처 레이어 역참조 없는지
6. **에디터 전용 코드** — `#if UNITY_EDITOR` 가드 누락 여부

---

## 주의사항

- 작업 전 반드시 Git 커밋 (diff Accept 후 잘못 발견 시 되돌리기 어려움)
- Claude는 Unity 런타임을 실행하지 않음 (Inspector, 물리, 성능 체감 모름)
- 핵심 게임 로직과 아키텍처 결정은 직접 설계, 구현 속도에만 활용
