---
author_name: 장상진
github_nickname: AlbertJang
title: 이펙티브 코틀린 - 18. 코딩 컨벤션을 지켜라
excerpt: 한 사람이 작성한 것처럼 작성되어야 한다.
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 코딩 컨벤션이란?
코딩 컨벤션은 읽고, 관리하기 쉬운 코드를 작성하기 위한 코딩 스타일 규약

## 코딩 컨벤션을 지켰을 경우
- 어떤 프로젝트를 접해도 쉽게 이해할 수 있습니다.
- 다른 외부 개발자도 프로젝트의 코드를 쉽게 이해할 수 있습니다.
- 다른 개발자도 코드의 작동 방식을 쉽게 추측할 수 있습니다.
- 코드의 유지보수가 쉽습니다.

## 기본적인 코딩 컨벤션
- 코틀린 문서의 ‘Coding Convensions’
- IntelliJ formatter
- ktlint

## 코딩 컨벤션 예시

```kotlin
class Person(
    val id: Int = 0,
    val name: String = "",
    val surname: String = ""
) : Human(id, name) {
    //...
}
```

```kotlin
// 이렇게 하지 마세요.
class Person(val id: Int = 0,
    val name: String = "",
    val surname: String = "") : Human(id, name) {
    //...
}
```
- 이런 형태로 작성하면, 클래스 이름을 변경할 때 모든 기본 생성자 파라미터의 들여쓰기를 조정해야 합니다.
- 처음 class 키워드가 있는 줄도 너비가 너무 크고, 이름이 가장 긴 마지막 파라미터와 슈퍼클래스 지정이 함께 있는 줄도 너무 큽니다.

## 요약정리
1. 코딩 컨벤션을 지키고 적극 사용하자!
