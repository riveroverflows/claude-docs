---
name: tech-blog
description: "Claude 세션 분석 기반 기술 블로그 초안 생성 (Hugo 블로그용)"
argument-hint: "[week_identifier]"
disable-model-invocation: true
allowed-tools: Bash(hugo *), Bash(cd *), Read, Edit
---

# /tech-blog - 기술 블로그 초안 생성

## Triggers
- 주간 기술 블로그 작성 과제 시작
- 개발 경험 정리 및 회고 글 작성 요청
- Claude 세션 기반 글감 추출 요청

## Usage
```
/tech-blog [week_identifier]
/tech-blog week1
/tech-blog week1/agents-and-tdd
```

## Configuration
```yaml
blog_path: /Users/river/Developer/river/blog
sessions_path: ~/.claude/projects/-Users-river-Developer-loopers-e-commerce
hugo_content_path: /Users/river/Developer/river/blog/content/posts
```

## Behavioral Flow

### Phase 1: 세션 분석 (Subagent 호출)
1. `$ARGUMENTS`에서 week 식별자 추출
2. 브랜치명 패턴 생성: `week{N}/*` 또는 직접 전달된 브랜치명
3. **session-analyzer** subagent 호출하여 세션 로그 분석
4. 요약 결과 수신 (주요 작업, 결정, 고민 포인트)

### Phase 2: 글감 도출 (사용자 협의)
technical-writing.dev 프레임워크 적용:

| 문서 유형 | 목적 | 적합한 상황 |
|----------|------|------------|
| **Learning** | 새로운 기술 습득 도움 | TDD 첫 경험, 새 언어/프레임워크 |
| **Problem-Solving** | 문제 해결 도움 | 삽질 기록, 디버깅 과정 |
| **Explanation** | 개념 깊은 이해 | Mock/Stub/Fake 비교 |
| **Reference** | 정보 빠른 검색 | 설정 가이드, 치트시트 |

**사용자에게 제시**:
```markdown
## 📝 이번 주 글감 후보

### 1. [추천] Learning + Problem-Solving
**제목**: "Java 개발자의 Kotlin TDD 첫 도전기"
- 핵심: TDD 3A 원칙 학습, Mock/Stub 혼란 해소 과정
- 세션: ae061f64, a05d57da

### 2. Explanation
**제목**: "Mock, Stub, Fake: 언제 어떤 걸 써야 할까?"
- 핵심: 테스트 더블 역할 구분, 실전 예시
- 세션: a05d57da, 4c9ab079

어떤 방향으로 진행할까요?
```

### Phase 3: Hugo 파일 생성
1. 사용자 선택 확인 후 진행
2. Hugo 명령어로 새 포스트 생성:
   ```bash
   cd /Users/river/Developer/river/blog
   hugo new content posts/{한글-폴더명}/index.md
   ```
3. 폴더명 규칙:
   - 한글 제목 사용 (띄어쓰기 → 대시)
   - 예: `Java-개발자의-Kotlin-TDD-도전기`

### Phase 4: Front Matter 수정
archetype에서 생성된 기본 내용 수정:
```yaml
---
title: '한글 제목'
date: 2026-02-04T00:00:00+09:00
lastmod: 2026-02-04T00:00:00+09:00
slug: 'english-url-slug'  # 영어로 수정
resource: 'uuid-auto-generated'
tags: ["tag1", "tag2"]
draft: true
---
```

### Phase 5: 본문 초안 작성
technical-writing.dev 구조 + Loopers 가이드 원칙 적용:

**Learning 유형 구조**:
```markdown
## 배경 (왜 이 글을 쓰게 되었나)
- 상황 설명
- 해결하고자 한 문제

## 핵심 내용
### 1. [첫 번째 주제]
- 개념 설명
- 코드 예시 (Java vs Kotlin 비교)
- 판단 근거

### 2. [두 번째 주제]
...

## 시행착오 (고민과 해결 과정)
- 처음 시도한 방법
- 왜 안 됐는지
- 어떻게 해결했는지

## 마무리
- 배운 점
- 다음 단계
```

**핵심 원칙** (Loopers 가이드):
- "무엇을 했다" < **"왜 그렇게 판단했는가"**
- 고민이 읽히는 글 (자만하지 않고)
- 포트폴리오로 활용 가능한 수준

## Tool Coordination
- **Task (session-analyzer)**: 세션 로그 분석 subagent 호출
- **Bash**: `hugo new content` 실행
- **Read**: 생성된 파일 확인
- **Edit**: Front matter 및 본문 수정
- **AskUserQuestion**: 글감 선택 및 방향 협의

## Examples

### 기본 사용
```
/tech-blog week1
```

### 특정 브랜치 지정
```
/tech-blog week1/agents-and-tdd
```

## Boundaries

**Will:**
- Claude 세션 로그 분석하여 글감 추출
- technical-writing.dev 프레임워크 기반 구조 제안
- Hugo 블로그 초안 생성 (Page Bundle 패턴)
- 사용자와 글 방향 협의

**Will Not:**
- 완성된 블로그 글 작성 (초안만 제공)
- 이미지/썸네일 생성
- 블로그 배포 또는 git push
- 세션 로그 외 정보 임의 추가
