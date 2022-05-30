---
author_name: 승열
github_nickname: seung-yeol
title: 이펙티브 코틀린 - 14.변수 타입이 명확하지 않은 경우 확실하게 지정하라
excerpt: 변수의 타입을 확실히 알아 볼 수 있게 하자.
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## 타입추론 
코틀린은 타입을 지정하지 않아도, 알아서 지정해주는 높은 수준의 타입 추론 시스템을 갖추고 있습니다.

```kotlin
val num = 10
val name = "seung yeol"
val ids = listOf(1,2,3,4,455)
```
이와 같이 유형이 **명확할 때**에는 코드가 짧아져 가독성이 향상이 됩니다.

```kotlin
val data = getSomeData()
```

위의 코드는 타입을 숨기고 있으며, 메서드의 네이밍도 모호하게 되어있어 타입의 유형이 명확하지 않습니다.
유형을 확인하기 위해서는 코드를 읽어 보아야 하며, 특히 github과 같은 환경에서는 코드이동이 불편하여 확인하기가 불편합니다.

```kotlin
val data: UserData = getUserData()
```
이와 같이, 타입을 직접 지정해주면 코드를 읽기 매우 쉽습니다.

가독성 향상 이외에 안전을 위해 타입을 지정해주는 것이 좋습니다.
[3. 최대한 플랫폼타입을 사용하지 마라](https://gripcorp.github.io/devblog/posts/kotlin/effective-kotlin/2022/05/10/03-do-not-use-platform-type.html)와 [4. inferred 타입으로 리턴하지 말라](https://gripcorp.github.io/devblog/posts/kotlin/effective-kotlin/2022/05/10/03-do-not-use-platform-type.html)에 관련내용이 있습니다.

## 요약 정리
- 개발자가 변수타입을 확실히 판단 할 수 있도록 코딩하자.
- 안전을 위해서라도 타입을 지정하여 사용해주는 것이 좋다.
