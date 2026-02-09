# GitHub Project 활용 가이드

> 이 문서는 e-commerce 프로젝트의 GitHub Issues/Projects 기능 분석 및 활용 방안을 정리합니다.

## 1. 조사 요약

### 현재 설정 상태

| 항목 | 상태 | 비고 |
|------|------|------|
| Project | ✅ 생성 완료 | E-Commerce Platform |
| Work type | ✅ Custom field | Epic, Story, Task |
| Iteration | ✅ Custom field | 10주 교육기간 |

### 핵심 질문

- Label, Milestone이 현재 설정과 겹치는가?
- 어떻게 효율적으로 활용할 수 있는가?

## 2. 기능별 비교 분석

### 📊 비교표: 4가지 추적 도구

| 구분 | Label | Milestone | Work type (Custom field) | Iteration |
|------|-------|-----------|-------------------------|-----------|
| **범위** | Repository | Repository | Project | Project |
| **적용 개수** | 여러 개 | 1개 | 1개 | 1개 |
| **목적** | 분류/태깅 | 목표 지점 추적 | 작업 유형 분류 | 시간 블록 관리 |
| **진행률 표시** | ❌ | ✅ | ❌ | ✅ (로드맵) |
| **시각화** | 색상 | 완료율 바 | 색상 | 타임라인 |

### Label vs Work type 분석

- **Label**: 리포지토리 수준, 여러 개 적용 가능, 태깅/분류 목적
- **Work type**: 프로젝트 수준, 1개만 선택, 계층적 유형 분류
- **결론: 겹치지 않음** → 서로 보완적으로 사용 가능

### Milestone vs Iteration 분석

- **Milestone**: 리포지토리 수준, 특정 목표/릴리스 추적, 마감일 + 완료율
- **Iteration**: 프로젝트 수준, 반복 주기(스프린트) 관리, 시작일 + 종료일
- **결론: 부분적으로 겹침** → 교육과정에서는 Iteration으로 충분

## 3. 교육과정 맞춤 권장 구성

### ✅ 유지할 설정 (현재 상태)

#### 1. Work type: Epic / Story / Task

| Work type | 설명 | 예시 |
|-----------|------|------|
| **Epic** | 주차별 대과제 | "3주차 - 주문 시스템" |
| **Story** | 기능 단위 | "주문 생성 기능" |
| **Task** | 세부 작업 | "OrderService 구현" |

#### 2. Iteration: 10주 교육기간

- Week 1 ~ Week 10
- 로드맵 뷰에서 진행 상황 시각화

### ➕ 추가 권장: Labels

Labels는 **다차원 분류**에 유용. 하나의 이슈에 여러 Label을 붙일 수 있어 Work type과 함께 사용하면 효과적.

#### 권장 Label 체계

```yaml
# 우선순위 (Priority)
priority:high     🔴  # 즉시 처리 필요
priority:medium   🟡  # 일반적인 우선순위
priority:low      🟢  # 여유 있을 때

# 상태 (Status) - Work type과 다른 차원
status:blocked    🚫  # 다른 작업에 의해 막힘
status:review     👀  # 리뷰 대기 중
status:help-wanted 🆘  # 도움 필요

# 기술 분류 (Technical Type)
type:feature      ✨  # 새 기능
type:bug          🐛  # 버그 수정
type:refactor     ♻️  # 리팩토링
type:test         🧪  # 테스트 코드
type:docs         📚  # 문서화

# 도메인 (Domain) - e-commerce 맥락
domain:order      📦  # 주문 관련
domain:product    🏷️  # 상품 관련
domain:user       👤  # 회원 관련
domain:payment    💳  # 결제 관련
domain:cart       🛒  # 장바구니 관련
```

### ❓ Milestone 사용 여부: 선택적 사용

| 상황 | 권장 |
|------|------|
| 주차별 과제만 추적 | Iteration으로 충분 |
| 특정 마감 기한이 있는 목표 | Milestone 추가 사용 |
| PR 제출 마감 추적 | Milestone 유용 |

#### 예시 시나리오

```
Iteration: Week 3 (3/1 ~ 3/7)
├── Milestone: "Week 3 과제 제출" (마감: 3/7)
│   ├── Issue: 주문 생성 기능 구현 [Story]
│   └── Issue: 주문 조회 API 구현 [Task]
```

## 4. 실전 활용 예시

### 이슈 생성 시 설정 조합

```markdown
# Issue: 주문 생성 API 구현

## 메타데이터
- Work type: Task
- Iteration: Week 3
- Labels: type:feature, domain:order, priority:high
- Milestone: Week 3 과제 제출 (선택적)
```

### 필터링 활용

```
# 이번 주 고우선순위 작업 보기
iteration:@current label:priority:high

# 주문 도메인의 모든 기능 요청
label:domain:order label:type:feature

# 리뷰 대기 중인 작업
label:status:review
```

### 뷰 구성 권장

| 뷰 | 그룹화 기준 | 용도 |
|----|------------|------|
| **보드 뷰** | Status | To Do → In Progress → Done |
| **테이블 뷰** | - | Work type + Iteration + Labels 컬럼 표시 |
| **로드맵 뷰** | Iteration | 타임라인 시각화 |

## 5. 최종 결정사항

### 사용할 기능

| 기능 | 용도 | 설정값 |
|------|------|--------|
| ✅ **Work type** | 작업의 "크기/계층" | Epic, Story, Task |
| ✅ **Iteration** | 작업의 "시기" | Week 1 ~ Week 10 |

### 사용하지 않을 기능

| 기능 | 이유 |
|------|------|
| ❌ **Labels** | Work type으로 분류 충분 |
| ❌ **Milestone** | Iteration으로 시간 추적 충분 |

### 운영 원칙

- **Work type**: Epic → Story → Task 계층으로 작업 분해
- **Iteration**: Week 1 ~ Week 10으로 진행 상황 추적
- **Sub-issues**: 필요시 계층적 이슈 구조 활용 (Epic → Story → Task 연결)

## 6. 참고 문서

- [About Issues](https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues)
- [Managing Labels](https://docs.github.com/en/issues/using-labels-and-milestones-to-track-work/managing-labels)
- [About Milestones](https://docs.github.com/en/issues/using-labels-and-milestones-to-track-work/about-milestones)
- [About Iteration Fields](https://docs.github.com/en/issues/planning-and-tracking-with-projects/understanding-fields/about-iteration-fields)
- [Best Practices for Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/best-practices-for-projects)
