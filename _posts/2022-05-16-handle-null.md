---
author_name: 김종민
github_nickname: jongmin-kim-grip
title: 이펙티브 코틀린 - 8. 적절하게 null을 처리합시다
excerpt: null은 언제 사용하고 또 어떻게 사용하는게 좋을까요?
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## `null`?
- 값이 아직 부족해서 설정되지 않는 상태를 의미합니다.
- 혹은, 더 이상 값이 필요없기 때문에 제거되었다는 것을 의미합니다.

### `null`이란?

### 사용하는 경우들
- 프로퍼티 참조가 더 이상 필요없는 경우에 `null`을 할당합니다.
- 메소드의 리턴 값으로 사용되는 `null`은 여러 의미로 사용될 수 있습니다.
- 예를들어 `String.toIntOrNull()`은 적절하게 변환이 불가능한 경우 `null`을 리턴합니다.
- 또한, `Interable<T>.firstOrNull(() -> Boolean)`은 주어진 조건에 맞는 요소가 없을 경우 `null`을 리턴합니다.

## `null`의 처리
- 세이프 콜과 엘비스 연산자 등을 이용해 안전하게 처리합니다.
- 프로퍼티 혹은 함수를 리팩토링해 `nullable` 타입을 제거합니다.
- 혹은 `null`인 경우 오류를 던집니다.

```kotlin
val printer: Printer? = getPrinter()

printer.print() // Compile error
printer?.print() // Safe call
if (printer != null) printer.print() // Smart casting
printer!!.print() // Non-null assertion
```

### 세이프 콜 및 스마트 캐스팅으로 처리
- 개발자 관점에서 가장 안전한 방법입니다.
- 엘비스 연산자와 함께 사용할 수 있습니다.
- 코틀린의 다양한 확장 함수들은 스마트 캐스팅을 통한 `null` 처리를 지원합니다.

```kotlin
val printer: Printer? = getPrinter()

val printerName = printer?.name ?: "no-name"
val printerName = printer?.name ?: return
val printerName = pirnter?.name ?: throw Exception("no-name")

if (!printerName.isNullOrBlank()) {
    println(printerName.length)
}
```

### 오류를 던져 처리
- `null`일 경우 오류를 강제로 발생시켜 처리하는 방법입니다.
- `throw`, `!!`, `requireNotNull`, `checkNotNull` 등을 이용할 수 있습니다.

```kotlin
fun process(user: User) {
    requireNotNull(user.name)

    val context = checkNotNull(context)
    val networkService = getNetworkService(context) 
        ?: throw Exception("network service")

    networkService.getData { data ->
        show(data!!)
    }
}
```

### `not-null assertion`의 문제
- `null`을 가장 쉽게 처리할 수 있는 방법입니다.
- 하지만 남용하면 널 포인터 예외를 만나기 쉬워집니다.
- 예외가 발생할 경우 어떠한 설명도 없는 제네릭 예외가 발생합니다.
- 특히 현재는 널이 아니지만, 미래에도 확신할 수 없기 때문에 사용하지 않는 것이 좋습니다.
- 널인지 아닌지 관련된 정보가 보통 숨겨져 있기 때문에 발견하기 쉽지 않을 수 있습니다.


## `null` 피하기
- `null`은 어떻게든 적절하게 처리해야하기 때문에 추가 비용이 발생합니다.
- 따라서 꼭 필요한 경우가 아니라면 피하는 것이 좋습니다.

### 함수 제공하기
- 개발자가 필요한 상황에 맞는 처리를 할 수 있는 메소드를 제공하는 방법입니다.
- 예를들어, 리스트의 `get`, `getOrNull`, `getOrDefault` 등이 있습니다.

```kotlin
list.get(1)
list.getOrNull(1)
list.getOrDefault { item }
```

### 초기 값 만들기
- `null`을 대체할 수 있는 초기 값을 프로퍼티나 메소드의 리턴 값으로 이용하는 방법입니다.
- 예를들어 enum에서 `INVALID`를 추가하거나 `sealed` 클래스의 `Failure` 등이 있습니다.

```kotlin
enum class MembershipStatus {
    INVALID, VALID 
}

sealed class SignUpResult {
    object Success : SignUpResult()

    class Failure(val throwable: Throwable) : SignUpResult()
}
```

### `lateinit` 프로퍼티
- `lateinit` 한정자는 프로퍼티가 이후에 초기화 될 것을 명시합니다.
- 이를 이용하면 클래스를 초기화 할 때는 해당 프로퍼티가 `null`이지만 이후 할당을 통해 `not-null`을 보장할 수 있습니다.
- 하지만, 초기화 전에 값을 사용하면 널 포인트 예외가 발생합니다.
- 보통 클래스 생성 이후, 특정 라이프 사이클에 따라 초기화되는 프로퍼티에 사용합니다.

```kotlin
class UserTest {
    private lateinit var userDao

    @Before
    fun setUp() {
        userDao = UserDao()
    }

    @Test
    fun `Test getUsers`() {
        val users = userDao.getUsers()
    }
}
```

### `not-null` 델리게이트
- 기본 자료형인 경우 `lateinit`을 이용할 수 없습니다.
- 이때, `lateinit` 보다는 약간 느리지만, 델리게이트를 이용할 수 있습니다.
- `Delegates.notNull`이나 프로퍼티 위임을 사용합니다.

```kotlin
class UserActivity : Activity() {
    private var userId: Int by Delegates.notNull()
    private val isFromNotification: Boolean by arg(EXTRA_IS_FROM_NOTIFICATION)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        userId = intent.extras.getInt(EXTRA_USER_ID)
    }
}
```

## 요약정리
1. `null`은 특별한 의미를 전달하는데만 사용합시다.
1. `null`은 안전하게 처리하고, `not-null assertion`은 사용을 지양합시다.
1. 불필요한 `null` 대신 `not-null`로 만들어서 사용하는 방법을 적용합시다.
