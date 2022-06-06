---
author_name: 수현
github_nickname: soohyeon13
title: 이펙티브 코틀린 - 16.프로퍼티는 동작이 아니라 상태를 나타내야 한다.
excerpt:
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## 프러퍼티는 상태를 나타낸다.

코틀린의 프로퍼티는 자바의 필드와 비슷하지만 다르다.

```kotlin
//코틀린의 프로퍼티
var name: String? = null

//자바의 필드
String name = null
```

둘 다 데이터를 저장한다는 점은 같지만, 코틀린의 프로퍼티의 경우 사용자 정의 세터와 게터를 가질 수 있다.
이 처럼 `var`로 만든 프로퍼티를 파생 프로퍼티라고 한다.

`field`는 프로퍼티의 데이터를 저장해 두는 백킹 필드에 대한 레퍼런스이다.
`val`의 경우 field가 만들어 지지 않는다.

```kotlin
var name: String? = null
    get() = field?.toUpperCase()
    set(value) {
        if(!value.isNullOrBlank()) {
            field = value
        }
    }

val fullName: String
    get() = "$name $surname"
```

프로퍼티는 필드가 필요 없다. 
오히려 프로퍼티는 개념적으로 접근자를 나타낸다.
그래서 코틸린의 인터페이스에도 프로퍼티를 정의할 수 있다.

```kt
interface Person {
    val name: String
}
```

이렇게 한다면 게터를 가질 거라는 것을 나타낸다.

```kotlin
open class Supercomputer {
    open val theAnswer: Long = 42
}

class AndroidComputer: SuperComputer() {
    override val theAnwer: Long = 1_800_275_2273
}
```

프로퍼티를 위임할 수도 있다.
프로퍼티와 관련된 다양한 조작을 할 수 있고 이로인해 프로퍼티의 동작을 추출해서 재사용할 수 있다.(아이템 21)

```kotlin
val db: Database by laze { connectToDb() }
```

프로퍼티는 본질적으로 함수이므로, 확장 프로퍼티를 만들 수도 있다.

```kotlin
val Context.preferences: SharedPreferences
    get() = PreferenceManager
        .getDefaultSharedPreferences(this)
        
val Context.inflater: LayoutInflater
    get() = getSystemService(
        Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        
val Context.notificationManager: NotificationManager
    get() = getSystemService(Context.NOTIFICATION_SERVICE)
        as NotificationManager
```

프로퍼티를 함수 대신 사용할 수도 있지만 그렇다고 완전히 대체해서 사용하는 것은 좋지 않다.

```kotlin
// 이렇게 하지 마세요!
val Tree<Int>.sum: Int {
    get() = when (this) {
        is Leaf -> value 
        is Node -> left.sum + right.sum
    }
}

//이렇게 사용
fun Tree<Int>.sum(): Int = when (this) {
    is Leaf -> value
    is Node -> left.sum() + right.sum()
}
```

원칙적으로 프로퍼티는 상태를 나타내거나 설정하기 위한 목적으로만 사용하는 것이 좋고, 다른 로직 등을 포함하지 않아야 한다.

그렇다면 어떤 것을 프로퍼티로 해야하는가? 
"이 프로퍼티를 함수로 정의할 경우, 접두사로 get 또는 set을 붙일 것인가?" 
만약 아니라면, 이를 프로퍼티로 만드는 것은 좋지 않다.

- 프로퍼티 대신 함수를 사용하는 것이 좋은 경우
 - 연산 비용이 높거나, 복잡도가 O(1)보다 큰 경우
 - 비즈니스 로직(애플리케이션의 동작)을 포함하는 경우
 - 결정적이지 않은 경우
 - 변환의 경우
 - 게터에서 프로퍼티의 상태 변경이 일어나야 하는 경우 

상태를 추출/설정할 때는 프로퍼티를 사용해야 한다.

```kotlin
class UserIncorrect {
    private var name: String = ""
    
    fun getName() = name
    
    fun setName(name: String) {
        this.name = name
    }
}

class UserCorrect {
    var name: String = ""
}
```



## 요약 정리

- 상태를 추출/설정할 때는 프로퍼티를 사용해야 한다. 
- 상태 변경이 필요하거나 로직이 필요한 경우 함수를 사용해야 한다.
