---
author_name: 장상진
github_nickname: AlbertJang
title: 이펙티브 코틀린 - 17. 이름 있는 Argument를 사용하라
excerpt: 읽을 때도 좋고 코드의 안정성까지 향상시킬 수 있다.
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## 이름 있는 Argument를 사용해야 하는 이유
코드에서 Argument의 의미가 명확하지 않은 경우가 있습니다.

```kotlin
val text = (1..10).joinToString("|")
```
`joinToString`에 대해 미리 알지 못한다면 오해할 수 있습니다.

```kotlin
val text = (1..10).joinToString(separator = "|")
```
`separator`에 정확한 값이 들어갔는지 모릅니다.

```kotlin
val separator = "|"
val text = (1..10).joinToString(separator)
```
함수 호출에서 `separator`에 제대로 들어갔는지 모릅니다.

```kotlin
val separator = "|"
val text = (1..10).joinToString(separator = separator)
```

## 이름 있는 Argument를 사용했을 때 장점
- 이름을 기반으로 값이 무엇을 나타내는지 알 수 있습니다.
- 파라미터의 입력 순서와 상관 없으므로 안전합니다.

또한, 읽는 다른 사람에게도 좋습니다.


## 이름 있는 Argument는 언제 사용해야 할까?

### 디폴트 Argument의 경우
일반적으로 함수 이름은 필수 파라미터들과 관련되어 있기 때문에 디폴트 값을 갖는 optional 파라미터의 설명이 명확하지 않습니다.
그래서 이름을 붙여서 사용하는 것이 좋습니다.

### 같은 타입의 파라미터가 많은 경우
여러 파라미터가 같은 타입으로 되어 있다면 잘못 입력할 경우 문제가 발생할 수 있습니다.

```kotlin
fun sendEmail(to: String, message: String) { /*...*/}

sendEmail(
    to = "contact@kt.academy",
    message = "Hello, ..."
)
```


### 함수 타입 파라미터
예외) 함수 타입 파라미터는 함수 이름이 Argument를 설명해주기도 합니다. 그래서 이름을 붙여주지 않아도 됩니다.
```kotlin
thread {
    // ...
}
```

그 밖에 모든 함수 타입 Argument는 이름 있는 Argument를 사용하는 것이 좋습니다.

```kotlin
fun call(before: () -> Unit = {}, after: () -> Unit = {}) {
    before()
    print("Middle")
    after()
}

call ({ print("CALL")}) // CALLMiddle
call { print("CALL") }  // MiddleCALL
```

```kotlin
// Good!
call (before = { print("CALL") }) // CALLMiddle
call (after = { print("CALL") })  // MiddleCALL
```

Rxjava에서 `Observable`을 구독할 때 함수를 설정하는데 이는 함수 타입 Argument를 자주 볼 수 있는 형태입니다.
- 각각의 아이템을 받을 때 (OnNext)
- 오류가 발생했을 때 (OnError)
- 전체가 완료되었을 때 (OnComplete)

자바에서는 일반적으로 람다 표현식으로 코드를 작성하고 주석으로 설명을 붙입니다.
```java
observable.getUsers()
        .subscribe((List<User> users) -> { // onNext
             //...
        }, (Throwable throwable) -> { // onError
            //...
        }, () -> { // onCompleted
            //...
        });

```

코틀린에서는 이름있는 Argument를 활용해서 의미를 더 명확하게 할 수 있습니다.
```kotlin
observable.getUsers()
    .subscribeBy(
        onNext = { users: List<User> ->
            // ...
        },
        onError = { throwable: Throwable ->
            //...
        },
        onCompleted = {
            //...
        })
```


## 요약 정리
1. 함수에 같은 타입의 파라미터가 여러 개인 경우 이름있는 Argument를 사용하자
2. 함수 타입의 파라미터가 있는 경우 이름있는 Argument를 사용하자
3. 옵션 파라미터가 있는 경우 이름있는 Argument를 사용하자
4. 마지막 파라미터가 특별한 의미를 갖는 경우에만 이름있는 Argument를 생략해도 괜찮다
