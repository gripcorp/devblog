---
author_name: 김종민
github_nickname: jongmin-kim-grip
title: 이펙티브 코틀린 - 1. 가변성을 제한합시다
excerpt: 상태는 변덕꾸러기여서 같이 일하기 피곤해요
categories: [posts, kotlin, effective-kotlin]
tag: [kotlin, effective-kotlin]
---
## 상태는 양날의 검

### 상태란?
- 사물, 현상이 놓여 있는 모양이나 형편.
- 자연 현상의 관측에 의하여 가능 한 완전히 기술된 계의 존재 상황.

### 상태 관리의 어려움
- 상태를 갖는 값은 언제든 변할 수 있기 때문에 코드를 이해하기 힘들게 만듭니다.
- 멀티스레드 환경에서 충돌이 발생할 수 있기 때문에 동기화를 신경을 써줘야 합니다.
- 다양한 상태에 따른 테스트를 수행 및 관리해야 합니다.
- 종종 상태 변경이 일어날 때, 콜러에게 알려야 하는 경우도 생깁니다.

```kotlin
suspend fun noThreadSafe() {
    var number = 0

    coroutineScope {
        repeat(100) {
            launch {
	            delay(10)
	            number += 1 // No synchronized
	        }
        }
    }

    println(number) // The number can change from run to run
}
```

## 가변성 제한

### 읽기 전용 프로퍼티, val
- 프로퍼티를 선언할 때 `val`을 이용하면 읽기 전용으로 만들 수 있습니다.
- 이렇게 선언된 프로퍼티는 상수처럼 동작합니다.
- 만약 프로퍼티를 변경하면 컴파일 에러가 발생합니다.

```kotlin
val a = 10
a = 20 // Compile error
```

### val은 완전히 변경 불가능?
- 읽기 전용 프로퍼티가 뮤터블 객체를 담고 있다면, 내부적으로 변경 가능합니다.
- 즉, 프로퍼티 재할당은 불가능하지만, 객체의 상태는 변경이 가능하게 됩니다.

```kotlin
val mutableList = mutableListOf(1, 2, 3)
mutableList = emptyList<Int>() // Compile error
mutableList.add(4) // Re-assignable
```

- 프로퍼티는 캡슐화 되어있어 게터와 세터 메소드를 가질 수 있습니다.
- `var`은 게터와 세터를 모두 제공하지만, `val`은 게터만 제공합니다.
- 따라서, 읽기 전용 프로퍼티에 게터를 재정의 할 때, 변수를 참조하면 값이 변할 수 있습니다.

```kotlin
var name: String = "Jongmin"
var surName: String = "Kim"

val funllName: String
    get() = "$name $surName"

fun main() {
    println(fullName) // Jongmin Kim

    name = "Minjong"
    println(fullName) // Minjong Kim
}
```

### final 프로퍼티
- 값을 완전히 변경할 필요가 없다면 `final` 프로퍼티를 이용하는 것이 좋습니다.
- `val`로 선언된 값을 게터를 통해 전달받으면 콜러 쪽에서 변하지 않는 값이라고 착각할 수 있습니다.
- 따라서, 가능하면 `final` 프로퍼티를 지향하고, 게터 재정의는 지양합시다.
- 또한, `final` 프로퍼티는 스마트 캐스트를 이용할 수 있기 때문에 불필요한 코드를 줄일 수 있습니다.
- 게터를 꼭 사용해야하는 경우라면, 반환되는 값이 항상 일정한 경우에만 사용합시다.
- 이후 변경으로 버그가 발생할 수 있기 때문에 단위 테스트를 추가해둡시다.

```kotlin
val name: String = "Jongmin"
val surName: String? = "Kim"

val fullNameByGetter: String?
        get() = surName?.let { "$name $it" }
val fullNameByFinal: String? = surName?.let { "$name $it" }

fun main() {
    if (fullNameByGetter != null) {
        println(fullNameByGetter.length) // Not available smart cast, Compile error
    }

    if (fullNameByFinal != null) {
        println(fullNameByFinal.length) // Available smart cast, 11
    }
}
```

### 가변 컬렉션과 읽기 전용 컬렉션

- 코틀린은 읽고 쓸 수 있는 컬렉션과 읽기만 가능한 컬렉션을 구분합니다.
- `mutable`로 시작하지 않는 컬렉션은 읽기에 대한 메소드만 정의합니다.
- `mutable`로 시작하는 컬렉션은 이뮤타블 컬렉션을 상속하여 추가로 쓰기에 대한 메소드를 정의합니다.
- 따라서, 내부적으로 뮤터블한 컬렉션이지만 외부적으로 이뮤터블한 컬렉션으로 보일 수 있습니다.
- 만약, 개발자가 이뮤터블한 컬렉션을 뮤터블한 컬렉션으로 다운캐스팅할 경우 문제가 생깁니다.
- 컬렉션 다운캐스팅은 추상화를 무시하는 행위이기 때문에 지양합시다.
- 이뮤터블 컬렉션의 변경이 필요하다면, `toMutableList` 메소드를 통해 복제된 컬렉션을 이용합시다.

```kotlin
val immutableList = listOf(1, 2, 3)

if (immutableList is MutableList) {
    immutableList.add(4) // UnsupportedOperationException
}

val mutableList = immutableList.toMutableList() // Copy list
mutableList.add(4)
```

### data 클래스 한정자
- 뮤터블 객체는 예측하기 어려우며 위험하다는 단점이 있습니다.
- 반면, 이뮤터블 객체는 변경할 수 없다는 단점이 있습니다.
- 상황에 따라서 이뮤터블 객체의 특정 프로퍼티를 변경해야 할 경우가 생깁니다.
- 이 경우 복사 메소드를 만들어야 하는데, 프로퍼티의 수가 많아지면 관리하기 힘들어집니다.

```kotlin
class User(
    val name: String,
    val surName: String
) {
    fun createNameChangedUser(name: String): User = User(name, surName)

    fun createSurNameChangedUser(surName: String): User = User(name, surName)
}

val user = User(name = "Jongmin", surName = "Kim")
val nameChangedUser = user.createNameChangedUser(name = "Minjong")
val surNameChangedUser = user.createSurNameChangedUser(surName = "Lee")
```

- 이럴 때, `data` 클래스 한정자를 이용하면 `copy` 메소드를 자동으로 만들어줍니다.
- `copy` 메소드는 `named parameter`를 이용하여 특정 프로퍼티만 변경된 복제된 객체를 반환 해줍니다.
- 프로퍼티가 변경됨에 따라서 복사 메소드를 직접 관리할 필요가 없기 때문에 생산성이 증가합니다.

```kotlin
data class User(
    val name: String,
    val surName: String
)

val user = User(name = "Jongmin", surName = "Kim")
val nameChangedUser = user.copy(name = "Minjong")
val surNameChangedUser = user.copy(surName = "Lee")
```

## 변경 가능 지점

### 여러 변경 가능 지점들
- 첫번째 방법은 뮤터블 리스트를 `val`로 선언하는 것입니다.
- 이 경우 리스트 구현 내부에 변경점이 있습니다.
- 멀티스레드 환경에서 리스트 내부에 변경점에 대한 처리를 보장 하는지 알 수 없어 위험합니다.

```kotlin
val mutableListVal = mutableListOf(1, 2, 3)
mutableListVal.add(4)
mutableListVal += 5 // Call plusAssgin
```

- 두번째 방법은 이뮤터블 리스트를 `var`로 선언하는 것입니다.
- 이 경우 프로퍼티 자체가 변경점입니다.
- 멀티스레드 환경에서 프로퍼티 값 변경점에 적절한 동기화를 처리해줘야 합니다.

```kotlin
var immutableListVar = listOf(1, 2, 3)
mutableListVal += 4 // Call plus
```

- 세번째 방법은 뮤터블 리스트를 `var`로 선언하는 것입니다.
- 리스트 내부에도 변경점이 있고, 프로퍼티 값에도 변경점이 있습니다.
- 양쪽 변경점을 개발자가 모두 관리해야하기 때문에 매우 위험합니다.

```kotlin
var mutableListVar = mutableListOf(1, 2, 3)
mutableListVar.add(4)
mutableListVar += 5 // Call plusAssign
mutableListVar = mutableListOf()
```

### 변경 가능 지점 숨기기
- 상태를 나타내는 뮤터블 객체를 외부에 노출하는 것은 굉장히 위험합니다.
- 콜리의 노출된 뮤터블 객체가 콜러에서 변경된다면 공유되는 상태가 다른 콜러에게도 영향을 주게됩니다.
- 따라서, 적절한 방법으로 원본 객체에 영향이 없도록 전달해야합니다.

```kotlin
class UserRepo {
    private val users: MutableList<User> = mutableListOf(User(name = "Jongmin"))

    fun getUsers(): MutableList<User> = users
}

class Caller {
    fun call() {
        val userRepo = UserRepo()

        print(userRepo.getUsers()) // [User(name=Jongmin)]

        users.clear()
        println(userRepo.getUsers()) // []
    }
}
```

- 첫번째 방법은 방어적 복제를 이용하는 것입니다.
- 반환할 때, 원본 객체를 복사하여 콜러에게 전달합니다.
- 이 경우 콜러가 해당뮤터블 객체를 변경하더라도, 원본 객체에는 영향이 없습니다.

```kotlin
class UserRepo {
    private val users: MutableList<User> = mutableListOf(User(name = "Jongmin"))

    fun getUsers(): MutableList<User> = mutableListOf(users) // Defensive copy
}
```

- 두번째 방법은 읽기 전용 슈퍼타입으로 업캐스트하여 제공하는 것입니다.
- 뮤터블 컬렉션은 이뮤터블 컬렉션을 상속하고 있기 때문에 가능합니다.
- 이 경우 콜러에게 변경이 필요한 경우 스스로 복제해서 사용하도록 강제합니다.

```kotlin
class UserRepo {
    private val users: MutableList<User> = mutableListOf(User(name = "Jongmin"))

    fun getUsers(): List<User> = users // To immutable list
}
```

## 요약정리
1. `var`보다는 `val`을 이용해주세요.
1. 뮤터블 객체보다는 이뮤터블 객체를 이용해주세요.
1. 객체의 변경이 필요한 경우 이뮤터블 객체를 `data` 클래스로 만들고 `copy` 메소드를 이용해주세요.
1. 변경점을 최소화 시키고 적절한 처리 및 테스트 코드를 작성해주세요.
1. 이뮤터블 객체는 외부로 부터 최대한 숨겨주세요.
1. 가끔은 효율성 때문에 뮤터블 객체를 이용하는 경우도 있지만, 최적화를 다시 한번 고려해주세요.
