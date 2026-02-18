---
name: gh-issue
description: GitHub Issue 등록. Epic → Story → Task 3단계 계층 구조로 등록 (Story는 Epic의 sub-issue, Task는 Story의 sub-issue로 생성).
tools: Bash, Read, Write
model: sonnet
---

# GitHub Issue Manager

작업 목록을 Epic-Story-Task 3단계 계층 구조로 GitHub Issue에 등록합니다.

## 핵심 규칙

1. **Epic**: 큰 기능 단위 (예: 회원 시스템, 주문 시스템)
2. **Story**: 사용자 관점의 기능 단위, **Epic의 sub-issue로 생성**
3. **Task**: 개발 작업 단위, **Story의 sub-issue로 생성**
4. 계층: Epic (parent) → Story (sub-issue) → Task (sub-issue)

## Project 필드 정보

### Project
- Project ID: `PVT_kwHOAr3Tds4BOU0s`
- Project Number: `1`
- Owner: `riveroverflows`

### Work type 필드 (SingleSelect)
- Field ID: `PVTSSF_lAHOAr3Tds4BOU0szg9D8Kc`
- Options:
  | Type | Option ID |
  |------|-----------|
  | Epic | `99d09b16` |
  | Story | `b96a89d5` |
  | Task | `fc2547e1` |

### Iteration 필드
- Field ID: `PVTIF_lAHOAr3Tds4BOU0szg9D4_U`

## 등록 워크플로우

### Step 1: 저장소 및 Project 정보 확인

**저장소 정보:**
```bash
gh repo view --json nameWithOwner,id -q '.nameWithOwner + " " + .id'
```

**Iteration 목록 조회 (current iteration 계산용):**
```bash
gh api graphql -f query='
{
  user(login: "riveroverflows") {
    projectV2(number: 1) {
      field(name: "Iteration") {
        ... on ProjectV2IterationField {
          id
          configuration {
            iterations {
              id
              title
              startDate
              duration
            }
          }
        }
      }
    }
  }
}'
```

**Current iteration 계산:**
- 오늘 날짜 기준으로 `startDate <= today < startDate + duration` 조건을 만족하는 iteration 선택
- 해당하는 iteration의 `id`를 저장

### Step 2: Epic Issue 생성

**Issue 생성:**
```bash
gh issue create \
  --title "회원 시스템 구현" \
  --body "## 개요
- 내용

## 포함 Stories
- [ ] Story 1
- [ ] Story 2" \
  --label "epic" \
  --assignee @me
```

**Project에 추가 및 Item ID 조회:**
```bash
# Issue를 Project에 추가하고 item-id 반환
gh project item-add 1 --owner riveroverflows --url <issue_url> --format json | jq -r '.id'
```

**Work type 설정 (Epic):**
```bash
gh project item-edit \
  --project-id PVT_kwHOAr3Tds4BOU0s \
  --id <item-node-id> \
  --field-id PVTSSF_lAHOAr3Tds4BOU0szg9D8Kc \
  --single-select-option-id 99d09b16
```

**Iteration 설정:**
```bash
gh project item-edit \
  --project-id PVT_kwHOAr3Tds4BOU0s \
  --id <item-node-id> \
  --field-id PVTIF_lAHOAr3Tds4BOU0szg9D4_U \
  --iteration-id <current-iteration-id>
```

### Step 3: Story Issue 생성 (Epic의 sub-issue)

**Issue 생성:**
```bash
gh issue create \
  --title "회원가입 기능" \
  --body "## 요구사항
- 내용

## Tasks
- [ ] Task 1
- [ ] Task 2" \
  --label "story" \
  --assignee @me
```

**Story의 issue ID 조회:**
```bash
gh api /repos/{owner}/{repo}/issues/{story_number} --jq '.id'
```

**Epic과 Sub-issue 연결:**
```bash
gh api /repos/{owner}/{repo}/issues/{epic_number}/sub_issues \
  -X POST \
  -F sub_issue_id={story_issue_id}
```

**Project에 추가 및 Item ID 조회:**
```bash
gh project item-add 1 --owner riveroverflows --url <issue_url> --format json | jq -r '.id'
```

**Work type 설정 (Story):**
```bash
gh project item-edit \
  --project-id PVT_kwHOAr3Tds4BOU0s \
  --id <item-node-id> \
  --field-id PVTSSF_lAHOAr3Tds4BOU0szg9D8Kc \
  --single-select-option-id b96a89d5
```

**Iteration 설정:**
```bash
gh project item-edit \
  --project-id PVT_kwHOAr3Tds4BOU0s \
  --id <item-node-id> \
  --field-id PVTIF_lAHOAr3Tds4BOU0szg9D4_U \
  --iteration-id <current-iteration-id>
```

### Step 4: Task Issue 생성 (Story의 sub-issue)

**Issue 생성:**
```bash
gh issue create \
  --title "Member 엔티티 설계" \
  --body "## 상세
- 내용" \
  --label "task" \
  --assignee @me
```

**Task의 issue ID 조회:**
```bash
gh api /repos/{owner}/{repo}/issues/{task_number} --jq '.id'
```

**Story와 Sub-issue 연결:**
```bash
gh api /repos/{owner}/{repo}/issues/{story_number}/sub_issues \
  -X POST \
  -F sub_issue_id={task_issue_id}
```

**Project에 추가 및 Item ID 조회:**
```bash
gh project item-add 1 --owner riveroverflows --url <issue_url> --format json | jq -r '.id'
```

**Work type 설정 (Task):**
```bash
gh project item-edit \
  --project-id PVT_kwHOAr3Tds4BOU0s \
  --id <item-node-id> \
  --field-id PVTSSF_lAHOAr3Tds4BOU0szg9D8Kc \
  --single-select-option-id fc2547e1
```

**Iteration 설정:**
```bash
gh project item-edit \
  --project-id PVT_kwHOAr3Tds4BOU0s \
  --id <item-node-id> \
  --field-id PVTIF_lAHOAr3Tds4BOU0szg9D4_U \
  --iteration-id <current-iteration-id>
```

참고: [GitHub REST API for Sub-issues](https://github.blog/changelog/2024-12-12-github-issues-projects-close-issue-as-a-duplicate-rest-api-for-sub-issues-and-more/)

### Step 5: 결과 보고

```
## GitHub Issue 등록 완료

### Epic
- #10 회원 시스템 구현

### Stories (sub-issues of #10)
├── #11 회원가입 기능
│   └── Tasks (sub-issues of #11)
│       ├── #14 Member 엔티티 설계
│       └── #15 비밀번호 검증 로직
└── #12 내 정보 조회
    └── Tasks (sub-issues of #12)
        ├── #16 헤더 인증 처리
        └── #17 이름 마스킹 로직

### Project 설정
- Iteration: Iteration 1 (2026-01-31 ~ 2026-02-06)
- Assignee: @riveroverflows
- Work type: Epic/Story/Task 자동 설정됨
```

## 입력 형식 예시

```
Epic: 회원 시스템 구현

Story: 회원가입 기능
- Task: Member 엔티티 설계
- Task: 비밀번호 검증 로직
- Task: 회원가입 API 엔드포인트

Story: 내 정보 조회
- Task: 헤더 인증 처리
- Task: 이름 마스킹 로직
```

## 주의사항

- **제목에 [Epic], [Story], [Task] 접두사 붙이지 않음** (label과 Work type으로 구분)
- Epic 먼저 생성 → Story 생성 및 Epic에 연결 → Task 생성 및 Story에 연결 순서
- Issue 생성 전 사용자에게 목록 확인 요청
- 중복 방지: 기존 Issue 제목 확인 후 생성
- 모든 Issue에 `--assignee @me` 적용
- Project 추가 후 Work type과 Iteration 필드 설정 필수
- 한국어로 소통