---
author_name: 장상진
github_nickname: AlbertJang
title: 이펙티브 코틀린 - 7. 예외를 활용해 코드에 제한을 걸어라
excerpt: 제한을 걸어 예상치 못한 상황을 막자
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
### 요구사항이 확실한 상황에서는 예외를 활용해서 제한을 걸어주는게 좋습니다.
- 문서를 읽지 않은 개발자도 문제를 확인할 수 있습니다. (문서화가 필요하지 않다.)
- 예상치 못한 동작을 막을 수 있습니다.
- 코드가 자체적으로 검사되기 때문에 단위 테스트를 줄일 수 있고 안정적으로 동작하게 됩니다.
- 스마트 캐스트 기능을 활용할 수 있습니다.


## 제한 방법

### require
- Argument와 관련된 요구사항을 명시할 때 사용합니다.
- 요구사항을 만족시키지 못할 경우 IllegalArgumentException을 발생시킵니다.

```kotlin
fun factorial(n: Int): Long {
    require(n>=0) {
        "Cannot calculate factorial of $n " +
                "because it is smaller than 0" }
    return if (n<=1) 1 else factorial(n-1) * n
}

fun sendEmail(user: User, message: String) {
    requireNotNull(user.email)
    require(isValidEmail(user.email))
    //...
}
```

### check
- State와 관련된 요구사항을 명시할 때 사용합니다.
- 요구사항을 만족시키지 못할 경우 IllegalStateException을 발생시킵니다.

```kotlin
fun speak(text: String) {
    check(isInitialized)
    //...
}

fun getUserInfo(): UserInfo {
    checkNotNull(token)
    //...
}
```

### assert
- 값이 true인지 판별합니다.
- 테스트 모드에서 테스트를 할 때 사용합니다.
- 크리티컬한 코드라면 check를 사용하는 것을 권장합니다.

```kotlin
class StackTest {

    @Test
    fun 'Stack pops correct number of elements'() {
        val stack = Stack(10) { it }
        val ret = stack.pop(10)
        assertEquals(10, ret.size)
    }

    //...
}
```

### nullability와 스마트 캐스팅
`check`와 `require`를 활용해 타입 비교를 했다면 스마트 캐스트가 작동합니다.
```kotlin
// smart cast
fun changeDress(person: Person) {
    require(person.outfit is Dress)
    val dress: Dress = person.outfit
}
```
`checkNotNull`과 `requireNotNull`을 사용한다면 unpack 용도로 사용할 수 있습니다.
```kotlin
// unpack
fun sendEmail(person: Person) {
    val email = requireNotNull(person.email)
    validateEmail(email)
    //...
}
```

Elvis 연산자를 사용해서 nullable을 처리할 수 있습니다.
```kotlin
fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: run {
        log("Email not sent, no email address")
        return // or throw
    }
    //...
}
```

## 요약정리
1. `require` 블록은 `argument` 관련된 사항을 정의할 때 사용합니다.
1. `check` 블록은 `state`와 관련된 사항을 정의할 때 사용합니다.
1. `assert` 블록은 테스트 모드에서 테스트 할 때 사용합니다.
1. Elvis 연산자와 `return`/`throw`를 활용합시다.
