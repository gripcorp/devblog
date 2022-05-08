---
title: "이펙티브 코틀린 - 1. 가변성을 제한합시다"
categories: [posts, kotlin, effective-kotlin]
tag: [kotlin, effective-kotlin]
---
### 상태는 양날의 검 ⚔︎

<center>
    <img src="https://mblogthumb-phinf.pstatic.net/MjAyMDA5MDFfMjc5/MDAxNTk4OTM2NzQ4Njg4.brCGhmPaIzjeUD3hFCx2rS0jUHsipb3vbAt_Ui4X3zog.vnp1-JcJ_85NSbG3VdQ6D0akVNuX-0Q2NWl0zjNwWfgg.JPEG.jwyr5507/%E3%84%B43.jpg?type=w800" width="400px" />
</center>
<p />
<center>
    <i>"상태는 변덕꾸러기여서 같이 일하기 피곤해요"</i>
</center>

#### 상태 관리는 어려워요
상태를 갖는 값은 언제든 변할 수 있기 때문에 코드를 이해하고 추적하기 힘듭니다.<br>
특히 멀티스레드 환경에서 충돌이 발생할 수 있기 때문에 동기화를 신경을 써줘야 합니다.<br>
다양한 상태에 따른 테스트를 수행 및 관리해야 합니다.<br>
종종 상태 변경이 일어날 때, 콜러에게 알려야 하는 경우도 생깁니다.<br>

```kotlin
suspend fun main() {
    var num = 0
    coroutineScope {
        repeat(100) {
            launch {
	        delay(10)
	        num += 1 // 멀티스레드 환경에서 상태 변경
	    }
        }
    }
    println(num) // 동기화 처리를 하지 않아서 실행할 때 마다 값이 달라질 수 있음
}
```

### 가변성 제한해봅시다 🔒

<center>
  <img src="https://s1.blackdesertm.com/KR/FORUM/BDM/Upload/Board/3e98e83041220190811191207629.gif" width="400px" />
</center>
<p />
<center>
    <i>"코틀린은 가변성을 어느정도 제한할 수 있어요"</i>
</center>

#### 읽기 전용 프로퍼티 val
프로퍼티를 선언할 때 val을 이용하면 읽기 전용으로 만들 수 있습니다.<br>
이렇게 선언된 프로퍼티는 상수처럼 동작합니다.<br>
읽기 전용으로 선언된 프로퍼티를 변경하면 컴파일 에러가 발생합니다.<br>

```kotlin
val a = 10
a = 20 // 컴파일 에러
```

#### 완전히 변경 불가능한가요?
읽기 전용 프로퍼티가 mutable 객체를 담고 있다면, 내부적으로 변경 가능합니다.<br>
즉, 프로퍼티 재할당은 불가능하지만, 객체의 상태는 변경이 가능하게 됩니다.<br>

```kotlin
val mutableList = mutableListOf(1, 2, 3)
mutableList = emptyList<Int>() // 컴파일 에러
mutableList.add(4) // 객체 상태 변경 가능
```
<br>

프로퍼티는 캡슐화 되어 있어 게터와 세터를 가질 수 있습니다.<br>
var은 게터와 세터를 모두 제공하지만, val은 게터만 제공합니다.<br>
따라서, 읽기 전용 프로퍼티에 게터를 재정의 할 때 변수를 참조하면 값이 변할 수 있습니다.<br>

```kotlin
var name: String = "Jongmin"
var surName: String = "Kim"

val funllName: String
    get() = "$name $surName" // 게터를 통해 전달하기 때문에 값이 바뀔 수 있음

fun main() {
    println(fullName) // Jongmin Kim
    name = "Minjong"
    println(fullName) // Minjong Kim
}
```

#### final 프로퍼티를 이용해요
값을 완전히 변경할 필요가 없다면 final 프로퍼티를 이용하는 것이 좋습니다.<br>
val로 선언된 값을 게터를 통해 전달받으면 콜러 쪽에서 변하지 않는 값이라고 착각할 수 있습니다.<br>
따라서, 가능하면 final 프로퍼티를 지향하고, 게터 재정의는 지양합시다.<br>
또한, final 프로퍼티로 선언하면 스마트 캐스트를 이용할 수 있기 때문에 불필요한 코드를 줄일 수 있습니다.<br>
게터를 꼭 사용해야하는 경우라면, 반환되는 값이 항상 일정한 경우에만 사용합시다.<br>
이 경우, 이후 변경으로 버그가 발생할 수 있기 때문에 단위 테스트를 추가해둡시다.<br>

```kotlin
val name: String = "Jongmin"
val surName: String? = "Kim"

val fullNameByGetter: String?
        get() = surName?.let { "$name $it" }
val fullNameByFinal: String? = surName?.let { "$name $it" }

fun main() {
    if (fullNameByGetter != null) {
        println(fullNameByGetter.length) // 스마트 캐스트 불가능, 컴파일 에러
    }

    if (fullNameByFinal != null) {
        println(fullNameByFinal.length) // 스마트 캐스트 가능, 11
    }
}
```

#### 가변 컬렉션과 읽기 전용 컬렉션

<center>
    <img src="https://kotlinlang.org/docs/images/collections-diagram.png" width="400px"/>
</center>
<p />
<center>
    <i>"읽을 줄만 알면 문제가 간단해집니다"</i>
</center>
<p />
<br>

코틀린은 읽고 쓸 수 있는 컬렉션과 읽기만 가능한 컬렉션을 구분합니다.<br>
mutable로 시작하지 않는 컬렉션은 읽기에 대한 메소드만 정의합니다.<br>
mutable로 시작하는 컬렉션은 immutable 컬렉션을 상속합니다.<br>
mutable 컬렉션은 쓰기에 대한 메소드만 추가로 정의합니다.<br>
따라서, 내부적으로 mutable한 컬렉션이지만 외부적으로 immutable한 컬렉션으로 보일 수 있습니다.<br>
만약, 개발자가 immutable한 컬렉션을 mutable한 컬렉션으로 다운캐스팅할 경우 문제가 생깁니다.<br>
컬렉션 다운캐스팅은 계약을 위반하고, 추상화를 무시하는 행위이기 때문에 지양합시다.<br>
immutable 컬렉션의 변경이 필요하다면, toMutableList 메소드를 통해 복제된 컬렉션을 이용합시다.<br>

```kotlin
val immutableList = listOf(1, 2, 3)

if (immutableList is MutableList) {
    immutableList.add(4) // UnsupportedOperationException
}

val mutableList = immutableList.toMutableList() // immutable 리스트 복제
mutableList.add(4)
```

#### data 클래스 한정자
mutable 객체는 예측하기 어려우며 위험하다는 단점이 있습니다.<br>
반면, immutable 객체는 변경할 수 없다는 단점이 있습니다.<br>
상황에 따라서 immutable 객체의 특정 프로퍼티를 변경한 새로운 객체를 만들어야 하는 경우가 있습니다.<br>
이 경우 copy 메소드를 만들어야 하는데, 프로퍼티의 수가 많아지면 관리하기 힘듭니다.<br>

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
<br>

이럴 때, data 클래스 한정자를 이용하면 copy 메소드를 자동으로 만들어줍니다.<br>
copy 메소드는 named parameter를 이용하여 특정 프로퍼티만 변경된 복제된 객체를 반환 해줍니다.<br>
프로퍼티가 변경됨에 따라서 copy 메소드를 직접 관리할 필요가 없기 때문에 생산성이 증가합니다.<br>

```kotlin
data class User(
    val name: String,
    val surName: String
)

val user = User(name = "Jongmin", surName = "Kim")
val nameChangedUser = user.copy(name = "Minjong")
val surNameChangedUser = user.copy(surName = "Lee")
```

### 변경 가능 지점 🎯

<center>
    <img src="https://img.animalplanet.co.kr/news/2020/09/17/700/d44eb06uyrrcs7pn3pu6.jpg" width="300px" />
</center>
<p />
<center>
    <i>"가전제품은 거거익선, 변경 가능 지점은 소소익선"</i>
</center>

#### 여러 변경 가능 지점들
첫번째 방법은 mutable 리스트를 val로 선언하는 것입니다.<br>
이 경우 리스트 구현 내부에 변경점이 있습니다.<br>
멀티스레드 환경에서 리스트 내부에 변경점에 대한 처리를 보장 하는지 알 수 없어 위험합니다.<br>

```kotlin
val mutableListVal = mutableListOf(1, 2, 3)
mutableListVal.add(4)
mutableListVal += 5 // plusAssgin 호출
```
<br>

두번째 방법은 immutable 리스트를 var로 선언하는 것입니다.<br>
이 경우 프로퍼티 자체가 변경점입니다.<br>
멀티스레드 환경에서 프로퍼티 값 변경점에 적절한 동기화를 처리해줘야 합니다.<br>

```kotlin
var immutableListVar = listOf(1, 2, 3)
mutableListVal += 5 // plus 호출
```
<br>

세번째 방법은 mutable 리스트를 var로 선언하는 것입니다.<br>
리스트 내부에도 변경점이 있고, 프로퍼티 값도 변경점이 있습니다.<br>
양쪽 변경점을 개발자가 모두 관리해야하기 때문에 매우 위험합니다.<br>

```kotlin
var mutableListVar = mutableListOf(1, 2, 3)
mutableListVar.add(4)
mutableListVar += 5 // plusAssign 호출
mutableListVar = mutableListOf()
```

#### 변경 가능 지점을 숨기세요
상태를 나타내는 mutable 객체를 외부에 노출하는 것은 굉장히 위험합니다.<br>
콜리의 노출된 mutable 객체가 콜러에서 변경된다면 공유되는 상태가 다른 콜러에게도 영향을 주게됩니다.<br>
따라서, 적절한 방법으로 원본 객체에 영향이 없도록 전달해야합니다.<br>

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
<br>

첫번째 방법은 방어적 복제를 이용하는 것입니다.<br>
반환할 때, 원본 객체를 복사하여 콜러에게 전달합니다.<br>
이 경우 콜러가 해당 mutable 객체를 변경하더라도, 원본 객체에는 영향이 없습니다.<br>

```kotlin
class UserRepo {
    private val users: MutableList<User> = mutableListOf(User(name = "Jongmin"))

    fun getUsers(): MutableList<User> = mutableListOf(users) // 방어적 복제
}
```
<br>

두번째 방법은 읽기 전용 슈퍼타입으로 업캐스트하여 제공하는 것입니다.<br>
mutable 컬렉션은 immutable 컬렉션을 상속하고 있기 때문에 가능합니다.<br>
이 경우 콜러에게 변경이 필요한 경우 스스로 복제해서 사용하도록 강제합니다<br>

```kotlin
class UserRepo {
    private val users: MutableList<User> = mutableListOf(User(name = "Jongmin"))

    fun getUsers(): List<User> = users // immutable 리스트 반환
}
```

### 요약해봅시다 ⭐︎
var보다는 val을 이용해주세요.<br>
mutable 객체보다는 immutable 객체를 이용해주세요.<br>
객체의 변경이 필요한 경우 immutable 객체를 data 클래스로 만들고 copy 메소드를 이용해주세요.<br>
변경점을 최소화 시키고 적절한 처리 및 테스트 코드를 작성해주세요.<br>
mutable 객체는 외부로 부터 최대한 숨겨주세요.<br>
가끔은 효율성 때문에 mutable 객체를 이용하는 경우도 있지만, 최적화를 다시 한번 고려해주세요.<br>
