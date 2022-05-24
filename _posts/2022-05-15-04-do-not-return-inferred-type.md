---
author_name: 김종민
github_nickname: jongmin-kim-grip
title: 이펙티브 코틀린 - 4. 추론형 타입으로 리턴하지 맙시다
excerpt: 메소드는 정확한 타입으로 리턴하는게 좋아요
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 타입 추론

### 타입 추론의 위험성
- 추론형 타입은 오른쪽에 있는 피연산자에 맞게 설정됩니다.
- 슈퍼 클래스 또는 인터페이스로 설정되지 않습니다.
- 원하는 타입보다 제한된 타입이 설정되었다면, 타입을 명시적으로 지정하면 해결할 수 있습니다.

```kotlin
open class Animal

class Zebra: Animal()

fun main() {
    var zebra = Zebra()
    zebra = Animal() // Compile error

    var animal: Animal = Zebra()
    animal = Animal()
}
```

- 하지만 외부 라이브러리 같은 경우, 직접 조작할 수 없는 경우 문제가 생깁니다.
- 또한 이런 경우 추론형 타입을 노출하면, 위험한 일이 발생할 수 있습니다.
- 예를들어 아래와 같이 `produce` 메소드가 추론형 타입으로 리턴했기 때문에 `Fiat126P` 밖에 생산하지 못하게됩니다.
- 게다가 외부 라이브러리의 경우에 사용하는 쪽에서 내부 구현을 모를 수 있고, 고치기 쉽지 않게됩니다.

```kotlin
open class Car

class Fiat126P: Car()


interface CarFactory {
    fun produce() = DEFAULT_CAR

    companion object {
        private val DEFAULT_CAR = Fiat126P()
    }
}
```

## 요약정리
1. 추론형 타입은 예측 불가능한 결과를 낼 수 있다는 것을 기억해주세요.
1. 타입을 확실하게 지정해야 하는 경우 굉장히 중요한 정보이므로, 숨기지 맙시다.
1. 안전을 위해 외부 라이브러리를 만들 경우 반드시 리턴 타입을 지정해주세요.
1. 지정된 리턴 타입을 필요가 없어보인다고 함부로 제거하지 말아주세요.
