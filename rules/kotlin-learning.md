# Kotlin Learning Context

## 학습자 배경
- Java 백엔드 개발자, Kotlin 경험 없음
- 10주 멘토링 과정 (https://www.loopers.im/education)
- 목표: 설계/TDD 역량 강화, 문법보다 판단력

## Claude 행동 지침
- Kotlin 코드 작성 시 Java 대응 패턴 간단히 언급
- "Java로 치면 이런 거야" 형태의 비교 설명 선호
- 복잡한 문법보다 왜 그렇게 설계했는지 설명 우선
- 오버엔지니어링 금지, 최소 구현 우선

## Kotlin 핵심 패턴 (Java 대응)
- `val` = final, `var` = mutable
- `?.` = Optional.map, `?:` = orElse, `?: throw` = orElseThrow
- `data class` = Lombok @Data
- `companion object` = static
- `is` = instanceof, `as` = 캐스팅

## 공식 문서 참조
- Kotlin 문법: https://kotlinlang.org/docs/home.html
- Kotlin for Java developers: https://kotlinlang.org/docs/java-to-kotlin-guide.html
- Spring Boot + Kotlin: https://spring.io/guides/tutorials/spring-boot-kotlin
