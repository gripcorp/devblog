---
author_name: 김종민
github_nickname: jongmin-kim-grip
title: 이펙티브 코틀린 - 27. 추상화를 통해 코드를 보호합시다
excerpt: 내가 몰라도 되는 코드는 모르는 게 약입니다.
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 추상화의  좋은점
- 추상화로 클래스와 함수의 구현 코드를 숨기면, 콜러가 세부 사항을 알 수 없도록 만들 수 있습니다.
- 예를들어  정렬  로직을  함수로  구현하면, 함수의 구현부가  변경되어도  콜러의 코드에는 변경이 없습니다.

## 상수 추상화
- 반복적인 리터럴은 다른 개발자가 코드를 이해하기 힘들게합니다.
- 또한, 값 변경이 필요할 경우 사용되는 곳을 모두 찾아내어 변경해야 합니다.
- 이 과정에서 누락으로 인한 버그를 쉽게 발생시킬 수 있습니다.

```kotlin
val miniScreenSize = screenSize * 0.25f
```

- 변경이 필요없고 반복적으로 사용되는 리터럴을 상수로 선언하는 것을 상수 추상화라 부릅니다.
- 상수는 한번 초기화되면 변경할 수 없기 때문에 런타임에 변경으로 인한 버그를 사전에 차단합니다.
- 또한, 이름을 붙여주기 때문에 다른 개발자가 코드를 이해하기 쉽게 도와줍니다.
- 그 밖에, 코드의 양이 줄어들고, 상수 값만 바꾸면 참조하는 모든 곳에 적용되기 때문에 누락으로 인한 버그를 피할 수 있습니다.

```kotlin
// In callers,
val miniScreenSize = screenSize * MINI_SCREEN_SCALE

companion object {
	private const val MINI_SCREEN_SCALE: Float = 0.25f
}
```

<img src="https://mblogthumb-phinf.pstatic.net/20160325_179/ohlees_1458866686371xkU6u_JPEG/20160325_094357.png?type=w2" width="400" />

## 함수 추상화
- 자주 사용되는 로직을 함수로 선언하는 것을 함수 추상화라 부릅니다.
- 예를들어 토스트를 출력하는 경우에 메세지 부분만 달라진다면 아래와 같은 확장 함수를 만들 수 있습니다.

```kotlin
fun Context.toast(message: String, duration: Int = Toast.LENGTH_LONG) = 
    Toast.makeText(this /* context */, message, duration).show()

// In callers,
context.toast("Hello World!")
```

- 토스트를 출력하기 위한 전체 코드를 기억할 필요 없이, 짧은 코드로 동일한 로직을 수행할 수 있습니다.
- 또한, 로직의 변경이 필요할 경우에 구현부만 수정하면 모든 콜러에 적용할 수 있습니다.

## 함수를 추상화 할 경우 주의점
- 다만 위와 같은 코드는 몇 가지 개선가능한 부분들이 있습니다.
- 첫번째, 함수의 이름은 동작을 표현하는 것(=동사)이 더 좋습니다.
- 예를들어 스낵바를 출력하는 확장 함수가 추가된다면, 두 함수가 출력을 위한 것인지, 다른 동작을 위한 것인지 확실하지 않게 됩니다.
- 따라서, 위와 같은 경우 보여주는 의미를 확실하게 전달하려면 `showToast`라는 이름이 좋겠습니다.
- 둘째, `duration` 파라미터는 현재 정수 자료형을 요구하고 있습니다.
- 이 경우, 콜러는 어떤 값에 따라 출력이 변경되는지 알기 힘들며 의도하지 않은 값이 들어올 수 있습니다.
- 따라서, `enum` 자료형으로 범위를 제한하면, 콜러의 실수를 방지할 수 있습니다.
- FYI: [왜 Android API는 굳이 정수로 제공하는 것일까?](https://youtu.be/Hzs6OBcvNQE)

## 클래스 추상화
- 함수는 매우 단순한 추상화를 제공하며, 함수가 종료되면 상태를 유지할 수 없습니다.
- 또한, 시그니처를 변경하면 프로그램 전체에 큰 영향을 줄 수 있습니다.
- 클래스를 이용하면 상태를 가질 수 있고, 관련있는 함수들을 한 곳에 모을 수 있습니다.

```kotlin
class ToastShower(private val context: Context) {
	fun show(message: String, duration: Duration = Duration.LONG) = 
		Toast.makeText(context, message, duration.intValue).show()

	enum class Duration(val intValue: Int) {
		SHORT(intValue = Toast.LENGTH_LONG), 
		LONG(intValue = Toast.LENGTH_SHORT)
	}
}

// In callers,
ToastShower(context).show("Hellow World!")
```

- 클래스 추상화를 이용하면 콜러의 유닛 테스트 코드에서 해당 클래스의 목 객체를 만들 수 있습니다.
- 사용하는 클래스 함수의 구현부와 상관 없이, 넘어가는 파라미터를 검증하거나 리턴 값을 목킹 할 수 있습니다.

```kotlin
class Caller(private val toastShower: ToastShower) {
	fun call() = toastShower.show(message = "Hello World!")
}

@RunWith(RobolectricTestRunner::class)
class CallerTest {
	@get:Rule
	val mockitoRule: MockitoRule = MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS)

	@Mock
	private lateinit var toastShower: ToastShower

	private lateinit var caller: Caller

	@Before
	fun setUp() {
		caller = Caller(toastShower)
	}

	@Test
	fun `Test call`() {
		// Given
		doNothing().whenever(toastShower).show(anyString(), any()
		
		// When
		caller.callShowHelloWorldToast()

		// Then
		verify(toastShower).show(eq(value = "Hello World!", eq(ToastShower.Duration.LONG))
	}
}
```

- 클래스를 이용한 추상화는 함수를 이용한 추상화보다 많은 기능을 제공합니다.
- 하지만 클래스이기 때문에 구현부가 불필요하게 콜러에게 노출될 수 있습니다.

## 인터페이스 추상화
- 라이브러리를 만드는 개발자는 클래스의 구현부를 콜러에게 노출시키지 않고 시그니처만 제공하길 원합니다.
- 결합도(=coupling)를 최소화하여 콜러에서 구현부가 변경되더라도 시그니처가 변경되지 않으면 영향이 없도록 하기 위함입니다.
- 예를들어 `Kotlin`은 제공하는 라이브러리는 대부분 인터페이스 추상화로 제공합니다.
- 이로서, JVM, Native, JavaScript 등 다양한 플랫폼에 관계 없이 콜러에 동일한 시그니처를 제공할 수 있습니다.
- 앞서 만든 토스트 예제 코드에 스낵바가 추가되면 아래와 같이 작성할 수 있습니다.
```kotlin
interface MessageShower {
    fun show(message: String, duration: Duration = Duration.LONG)

    enum class Duration {
        SHORT, LONG
    }
}

internal class ToastShower(private val context: Context) : MessageShower {
    override fun show(message: String, duration: MessageShower.Duration) =
        Toast.makeText(context, message, duration.toIntValue()).show()

    private fun MessageShower.Duration.toIntValue(): Int = when (this) {
        MessageShower.Duration.LONG -> Toast.LENGTH_LONG
        MessageShower.Duration.SHORT -> Toast.LENGTH_SHORT
    }
}

internal class SnackbarShower(private val view: View) : MessageShower {
    override fun show(message: String, duration: MessageShower.Duration) =
        Snackbar.make(view, message, duration.toIntValue()).show()

    private fun MessageShower.Duration.toIntValue(): Int = when (this) {
        MessageShower.Duration.LONG -> Snackbar.LENGTH_LONG
        MessageShower.Duration.SHORT -> Snackbar.LENGTH_SHORT
    }
}

class Caller(private val messageShower: MessageShower) {
    fun call() = messageShower.show(message = "Hello World!")
}

@RunWith(RobolectricTestRunner::class)
class CallerTest {
    @get:Rule
    val mockitoRule: MockitoRule = MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS)

    @Mock
    private lateinit var messageShower: MessageShower

    private lateinit var caller: Caller

    @Before
    fun setUp() {
        caller = Caller(messageShower)
    }

    @Test
    fun `Test call`() {
        // Given
        doNothing().whenever(messageShower).show(anyString(), any())

        // When
        caller.call()

        // Then
        verify(messageShower).show(eq(value = "Hello World!", eq(MessageShower.Duration.LONG))
    }
}
```

## 보편적인 객체 추상화
- 고유 아이디를 생성하여 사용해야하는 경우, 보통 정수나 문자열 자료형을 생각해볼 수 있습니다.
- 
```kotlin
var nextId = 0

// In callers,
val newId = nextId++
```

- 위와 같은 코드는 무조건 0부터 시작하게 됩니다.
- 콜러에서 값을 마음대로 변경할 수도 있습니다.
- 또한, 멀티쓰레드 환경에서 안전하지 않습니다.
- 이후에 다른 자료형으로 변경해야 할 경우 유연하게 대처할 수 없습니다.
- 위와 같은 문제는 사용자 정의 자료형을 만들어 콜러에게 추상화된 객체를 전달하는 방법으로 해결합니다.

```kotlin
private var nextId: Int = 0

data class Id(private val value: Int)

fun getNextId(): Id = Id(value++)
```

## 추상화의 문제점
- 추상화는 많은 장점을 제공하지만, 반대로 구현부의 코드의 양이 늘어나고 이해하기 어려워집니다.
- 특히, 오버 엔지니어링으로 인한 다중 상속이 사용되면 이런 문제는 점점 심해집니다.
- 반대로 콜러에게도 심한 정도의 추상화는 어떤 기능을 수행하는지 제대로 알기 힘들 수 있습니다.
- 따라서, 개발자는 적절한 정도의 추상화를 제공해야 합니다.
- 구현부를 숨김으로써 가져갈 수 있는 장점과 복잡도로 인한 유지보수 난이도를 저울질해야 합니다.
- 또한, 다른 개발자에게 길잡이가 되어줄 수 있는 유닛 테스트를 보다 꼼꼼하게 작성해야 합니다.

<img src="https://mblogthumb-phinf.pstatic.net/20140318_57/dohnynose_1395087760818IaqaS_JPEG/T05010_10.jpg?type=w2" width="300" />

## 균형의 수호자
- 많은 개발자가 참여할 수록 한번 구현부 코드가 작성되면 변경하기 쉽지 않습니다.
- 이 경우, 유지보수를 위해 모듈화와 추상화에 조금 더 힘을 쏟아야합니다.
- 또한 프로젝트에서 신뢰성이 중요한 경우, 유닛 테스트를 작성해야한다면 적절한 추상화는 도움이 됩니다.
- 함께하는 팀원들이 객체지향과 모듈화에 대한 지식이 있다면 적절한 크기의 모듈로 분리하고 숨겨야할 구현부를 숨기는게 좋습니다.
- 프로젝트가 진행됨에 따라 변화가 많이 생길 부분이라면 적당한 추상화를 진행하는 것이 좋습니다.

<img src="https://img.danawa.com/images/descFiles/4/394/3393003_1496382280720.jpeg" width="300" />

## 요약정리
1. 추상화는 단순히 중복 코드를 제거하는 것은 아닙니다.
2. 자주 변경되는 코드의 경우 추상화를 이용하면 유연하게 대처할 수 있습니다.
3. 추상화를 적용하고 유지보수하는 일을 쉽지 않습니다.
4. 적절한 추상화를 적용하고 저울질하는 것은 개발자의 책임입니다.
