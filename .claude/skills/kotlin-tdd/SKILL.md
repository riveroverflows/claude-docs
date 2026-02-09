---
name: kotlin-tdd
description: TDD 사이클로 Kotlin 기능 구현 (Java 개발자용)
argument-hint: "[feature_description]"
---

# TDD로 기능 구현

$ARGUMENTS 기능을 TDD(Red→Green→Refactor) 순서로 구현합니다.

## 진행 순서

1. **Red**: 실패하는 테스트 먼저 작성
   - @DisplayName 한국어로 테스트 의도 명시
   - 3A 패턴: Arrange → Act → Assert

2. **Green**: 테스트 통과하는 최소 코드 작성
   - 오버엔지니어링 금지
   - 요구사항에 없는 예외처리 추가 금지

3. **Refactor**: 코드 품질 개선
   - 중복 제거, 가독성 개선
   - 모든 테스트 통과 확인

## 설명 방식
- Java 개발자가 이해할 수 있게 Kotlin 문법 간단히 설명
- 설계 결정의 이유와 대안 제시

## 참조 문서
- Kotlin 공식: https://kotlinlang.org/docs/home.html
- Kotlin for Java: https://kotlinlang.org/docs/java-to-kotlin-guide.html
- Spring + Kotlin: https://spring.io/guides/tutorials/spring-boot-kotlin
- TDD Best Practice: https://martinfowler.com/bliki/TestDrivenDevelopment.html
