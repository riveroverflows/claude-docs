# claude-docs

- 프로젝트별 Claude Code 설정 파일을 하나의 저장소에서 관리하기 위해 만든 저장소
- 프로젝트별 브랜치를 생성하여 관리
- 프로젝트에서 프로젝트 브랜치를 submodule로 구성하여 운영

## 사용 방법

프로젝트 루트에서 `.claude/` 경로에 서브모듈로 추가합니다.

```bash
git submodule add -b <브랜치명> https://github.com/riveroverflows/claude-docs.git .claude
```

예시 (e-commerce 프로젝트):

- 프로젝트: [e-commerce](https://github.com/riveroverflows/e-commerce)
- claude-docs 저장소 브랜치: `loopers`

```bash
git submodule add -b loopers https://github.com/riveroverflows/claude-docs.git .claude
```

## 디렉토리 구조

```
.claude/
├── agents/       # Claude Code 커스텀 에이전트 정의
├── guides/       # 개발 가이드 문서 (Codex 협업, GitHub, 기술 문서 작성 등)
├── learning/     # 학습 관련 문서 및 마인드셋
├── rules/        # Claude Code 행동 규칙 (TDD, 테스트 레벨, PR 가이드 등)
├── skills/       # 재사용 가능한 슬래시 커맨드 스킬
└── settings.local.json  # 프로젝트별 로컬 권한 설정
```