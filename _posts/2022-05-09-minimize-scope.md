---
author_name: 김종민
github_nickname: jongmin-kim-grip
title: 이펙티브 코틀린 - 2. 변수의 스코프를 최소화합시다
excerpt: 좁힐 수 있을 때 최대한 좁혀놓으면 편해요 
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 스코프란
- 요소를 볼 수 있는 컴퓨터 프로그램 영역입니다.
- 코틀린의 소코프는 중괄호로 만들어집니다.
- 내부 스코프에서 외부 스코프에 있는 요소에만 접근할 수 있습니다.

```kotlin
val a = 1
val scope = {
    val b = 3
    println(b)
    println(a) // Compile error
}
```

## 스코프 최소화
- 상태를 정의할 때는 변수와 프로퍼티의 스코프를 최소화하는 것이 좋습니다.
- 프로퍼티보다 지역 변수를 사용하는 것이 좋습니다.
- 또한, 최대한 좁은 스코프를 갖게 변수를 사용합니다.
- 스코프를 좁게 유지하면 프로그램을 추적하고 관리하기 쉽기 때문입니다.
- 다른 개발자에 의해서 변수가 잘못 사용될 수 없도록 차단할 수도 있습니다.

```kotlin
// Bad case
var user: User
for (i in users.indices) {
    user = users[i]
    println("i=$i, user=$user")
}

// Good case
users.forEachIndexed { i, user ->
    println("i=$i, user=$user")
}
```

## 구조분해 선언
- 여러 프로퍼티를 한꺼번에 설정해야 하는 경우, 구조분해 선언을 활용하면 좋습니다.

```kotlin
// Bad case
fun updateWeather(degrees: Int) {
    val description: String
    val color: Int

    if (degrees < 6) {
        description = "cold"
        color = Color.BLUE
    } else if (degrees < 23) {
        description = "mild"
        color = Color.YELLOW
    } else {
        description = "hot"
        color = Color.RED
    }

    println(description=$description, color=$color)
}

// Good case
fun updateWeather(degrees: Int) {
    val (description, color) = when {
        degrees < 5 -> "cold" to Color.BLUE
        degrees < 23 -> "mild" to Color.YELLOW
        else -> "hot" to Color.RED
    }

    println(description=$description, color=$color)
}
```

## 정리해봅시다
1. 변수의 스코프는 좁게 만들어서 관리하게 좋게 만듭시다.
1. `var`보다는 `val`을 사용하는 것이 좋습니다.
