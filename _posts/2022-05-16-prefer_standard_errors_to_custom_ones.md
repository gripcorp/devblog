---
author_name: 장상진
github_nickname: AlbertJang
title: 이펙티브 코틀린 - 6. 사용자 정의 오류보다는 표준 오류를 사용하라
excerpt: 표준 라이브러리 오류를 사용하자
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 사용자 정의 오류
표준 라이브러리에 적절한 오류가 없다면 사용자 정의 오류를 사용합니다.

```kotlin
inline fun <reified T> String.readObject(): T {
    //...
    if (incorrectSign) {
        throw JsonParsingException()
    }
    //...
    return result
}
```

하지만 가능하다면 표준 오류를 사용하자!

## 표준 라이브러리 오류를 사용해야 하는 이유
사용자 정의 오류보다 표준 라이브러리의 오류는 많은 개발자들이 알기 때문에 다른 사람들이 보더라도 더 쉽게 이해할 수 있습니다.

### 표준 라이브러리 오류 예시
- IllegalArgumentException과 IllegalStateException 
 : 잘못된 argument 혹은 잘못된 state가 발생했다.
- IndexOutOfBoundsException
 : 인덱스 파라미터 값이 범위를 벗어났다.
- ConcurrentModificationException
 : 동시 수정을 금지 했는데, 발생했다.
- UnsupportedOperationException
 : 사용자가 사용하려했던 메서드가 현재 객체에서는 사용할 수 없다.
- NoSuchElementException
 : 사용자가 사용하려고 했던 요소가 존재하지 않는다.

## 요약정리
사용자 정의 오류보다 표준 라이브러리 오류를 사용합시다.
