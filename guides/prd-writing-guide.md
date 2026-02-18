# PRD 작성 가이드

> AI(Claude Code)와 효과적으로 협업하기 위한 PRD(Product Requirements Document) 작성 가이드

---

## 1. PRD란?

### 정의

PRD(Product Requirements Document)는 제품이 **무엇을** 해야 하는지, **왜** 존재해야 하는지, **누구를** 위해 만들어지는지를 정의하는 문서입니다.

### 목적

- **단일 정보 출처(Single Source of Truth)**: 팀 전체가 참조하는 기준 문서
- **범위 확산 방지**: 포함/미포함 사항을 명확히 하여 불필요한 작업 방지
- **커뮤니케이션 효율화**: 끊임없는 논쟁 대신 문서 참조로 질문 해결

### 핵심 가치

| 가치 | 설명 |
|------|------|
| **명확성** | 모호함 없이 정확하게 기술 |
| **정렬** | 모든 이해관계자가 동일한 이해 공유 |
| **추적 가능성** | 요구사항의 출처와 변경 이력 추적 |

---

## 2. 좋은 PRD의 특징

### 2.1 명확성 (Clarity)

- 모호한 표현 없이 정확하게 기술
- 기술적 용어는 일관되게 사용
- 예시와 함께 설명

**Bad**: "빠른 응답 시간"
**Good**: "API 응답 시간 200ms 이내"

### 2.2 포괄성 + 유연성 (Comprehensive yet Flexible)

- 큰 그림과 세부사항을 모두 포함
- 적응과 반복의 여지 확보
- 구현 방법보다 **무엇을** 달성해야 하는지에 집중

### 2.3 우선순위 (Prioritization)

| 등급 | 의미 | 설명 |
|------|------|------|
| **P0** | 필수 (Must Have) | 이것 없이는 제품 출시 불가 |
| **P1** | 중요 (Should Have) | 사용자 경험에 큰 영향 |
| **P2** | 부가 (Nice to Have) | 있으면 좋지만 없어도 됨 |

### 2.4 사용자 중심 (User-Centric)

- "기능이 없다"가 아닌 "어떤 고통/비효율이 있는가" 관점
- 사용자 스토리 형식 활용: "~로서, ~하고 싶다, ~하기 위해"

### 2.5 MVP 집중 (MVP Focus)

- 채택에 필요한 핵심 기능에 집중
- 포괄적인 목록으로 부풀리지 않음
- "나중에 추가할 것"은 명시적으로 Non-Goals에 배치

---

## 3. AI(Claude Code) 최적화 PRD 작성법

### 3.1 핵심 원칙

#### 명시적이고 구체적으로 작성

> Claude 4.x는 **문자 그대로 해석**합니다.
> 이전 버전처럼 의도를 추론하지 않고, 요청한 것만 정확히 수행합니다.

**Bad**: "적절한 에러 처리를 추가해주세요"
**Good**: "400 Bad Request 응답 시 `{"code": "INVALID_INPUT", "message": "에러 메시지"}` 형식으로 반환"

#### Non-Goals(하지 않을 것) 필수 명시

> AI는 생략에서 추론하지 못합니다.

"이 단계에서 인증을 구현하지 않는다"를 명시하지 않으면, AI가 자의적으로 인증을 추가할 수 있습니다.

```markdown
## Non-Goals (이번 범위에서 하지 않을 것)

- JWT/Session 기반 인증
- 회원 탈퇴 기능
- 이메일 인증
```

#### 의존성 순서로 단계별 구성

하나의 거대한 명세(300개 요구사항) 대신 **5~6개 단계, 각 30~50개 요구사항**으로 구조화

```markdown
## Phase 1: 기본 CRUD (이번 주)
- 회원가입
- 내 정보 조회

## Phase 2: 인증 강화 (다음 주)
- JWT 토큰 발급
- 리프레시 토큰
```

#### 테스트 가능한 수용 기준

각 요구사항에 검증 가능한 기준 포함:

```markdown
### 수용 기준 (Acceptance Criteria)

- [ ] POST /api/v1/users 요청 시 201 Created 반환
- [ ] 중복 loginId 요청 시 409 Conflict 반환
- [ ] 저장된 비밀번호는 BCrypt로 암호화됨
```

### 3.2 Claude 특화 팁

| 팁 | 설명 |
|-----|------|
| **긴 문서는 상단에** | Claude는 긴 컨텍스트에서 문서를 프롬프트 상단에 배치할 때 더 잘 이해 |
| **계약서처럼 작성** | 명시적, 범위가 한정적, 검증 가능한 형태 |
| **오버엔지니어링 금지 명시** | "요청된 변경만 수행" 문구 포함 |
| **예시 포함** | 요청/응답 JSON 예시, 에러 케이스 예시 |

---

## 4. AI-Ready PRD 템플릿

```markdown
# [기능명] PRD

## 개요
[1-2문장으로 기능 설명]

## 개발 환경 명령어

| 명령어 | 설명 |
|--------|------|
| `./gradlew build` | 빌드 |
| `./gradlew test` | 테스트 실행 |

## 프로젝트 구조

| 레이어 | 패키지 경로 | 생성할 파일 |
|--------|------------|------------|
| Controller | `com.example.interfaces` | XxxController |
| Domain | `com.example.domain` | Xxx, XxxRepository |

## Non-Goals (이번 범위에서 하지 않을 것)

- [명시적으로 하지 않을 것 나열]

---

## 기능 1: [기능명]

### 엔드포인트
- **Method**: POST
- **Path**: /api/v1/xxx
- **인증**: 필요/불필요

### 요청 (Request)
```json
{
  "field": "value"
}
```

### 유효성 규칙

| 필드 | 규칙 | 검증 위치 |
|------|------|----------|
| field | 조건 | DTO/Domain |

### 응답 (Response)

**성공 (201 Created)**
```json
{
  "id": "string"
}
```

**실패 (400 Bad Request)**
```json
{
  "code": "ERROR_CODE",
  "message": "에러 메시지"
}
```

### 수용 기준 (Acceptance Criteria)

- [ ] 조건 1
- [ ] 조건 2

---

## 에러 코드 정의

| 코드 | HTTP Status | 설명 |
|------|-------------|------|
| ERROR_CODE | 400 | 설명 |

## 기술 스택

| 항목 | 기술 |
|------|------|
| 암호화 | BCrypt |
```

---

## 5. PRD 작성 체크리스트

### 기본 요소

- [ ] 개요: 기능의 목적과 배경 설명
- [ ] Non-Goals: 하지 않을 것 명시
- [ ] 우선순위: P0/P1/P2 분류

### AI 최적화 요소

- [ ] Commands: 빌드/테스트 명령어 포함
- [ ] Project Structure: 파일 생성 위치 명시
- [ ] 수용 기준: 각 기능별 검증 가능한 체크리스트
- [ ] 에러 케이스: 실패 응답 형식과 코드 명시

### 품질 요소

- [ ] 모호한 표현 없음 ("적절한", "빠른" 등 제거)
- [ ] JSON 예시 포함
- [ ] 테스트로 검증 가능한 조건

---

## 6. 참고 자료

- [코드스테이츠 - PRD의 모든 것](https://www.codestates.com/blog/content/prd-%EC%A0%9C%ED%92%88%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD%EC%A0%95%EC%9D%98%EC%84%9C)
- [How to Write PRDs for AI Coding Agents](https://medium.com/@haberlah/how-to-write-prds-for-ai-coding-agents-d60d72efb797)
- [Addy Osmani - Good Spec for AI Agents](https://addyosmani.com/blog/good-spec/)
- [CLAUDE.md Best Practices](https://arize.com/blog/claude-md-best-practices-learned-from-optimizing-claude-code-with-prompt-learning/)
- [Claude Prompt Engineering Best Practices](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices)
- [AI PRD Template by OpenAI PM](https://www.productcompass.pm/p/ai-prd-template)
