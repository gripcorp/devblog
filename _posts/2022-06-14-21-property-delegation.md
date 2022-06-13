---
author_name: 김준석
github_nickname: idKimjunseok
title: 이펙티브 코틀린 - 21. 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라.
excerpt: 코틀린은 프로퍼티 위임으로 프로퍼티 패턴을 간단히 구현할수 있다.
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 지연 프로퍼티
```kotlin
val value by lazy {createValue() }
```
lazy 프로퍼티는 처음 사용하는 요청이 들어올때 초기화되는 프로퍼티.
일반적인 언어에서는 필요할때마다 구현해야함.
코틀린에서는 lazy함수로 쉽게 구현

```kotlin
val items: List<Item> by
	Delegates.observable(listOf()) {_, _, _ ->
		notifyDataSetChanged()
	}

var key: String? by {
	Delegates.observable(null) {_, old, new ->
	Log.e("key changed from $old to $new")
```
프로퍼티 위임 사용하면 observable 패턴을 쉽게 만들수 있음.

```kotlin
//안드로이드에서의 뷰와 리소스 바인딩
private val button: Button by bindView(R.id.button)
private val textSize by bindDimension(R.dimen.font_size)
private val doctor: Doctor by argExtra(DOCTOR_ARG)

//Koin에서의 종속성 주입
private val presenter: MainPresenter by inject()
private val repository: NetworkRepository by inject()
private val vm: MainViewModel by viewModel()

//데이터 바인딩
private val port by bindConfiguration("port")
private val token: String by preferences.bind(TOKEN_KEY)
```
자바등에서는 어노테이션을 많이 활용해야 하지만.
코틀린 프로퍼티 위임을 사용하면 뷰, 리소스 바인딩, 의존성 주입, 데이터 바인딩을 
간단하고 type-safe하게 구현할수 있다.

```kotlin
var token: String? = null
	get() {
		print("token returnbed value $field")
		return field
	}
	set(value) {
		print("token changed from value $field to $field")
		field = value
	}
var attempts: Int = 0
	get() {
		print("attempts returnbed value $field")
		return field
	}
	set(value) {
		print("attempts changed from value $field to $field")
		field = value
	}
```
내부적으로는 다르지만 반복되는 패턴. 즉 추출하기 좋은 부분
프로퍼티 위임은 다른 객체의 메서드를 활용해서 프로퍼티 접근자(게터 세터)를 만드는 방식.

```kotlin
var token: String? by LogginfProperty(null)
var attempts: Int by LogginfProperty(0)

private class LogginfProperty<T>(var value: T) {
	operator fun getValue(
		thisRef: Anty?,
		prop: KProperty<*>
		): T {
			print("${prop.name} returned value $value")
			return value
		}
		
		operator fun setValue(
		thisRef: Anty?,
		prop: KProperty<*>,
		newValue: T
		) {
			val name = prop.name
			print("$name changed from $value to $newValue")
			value = newValue
		}
}
```
이전 코드를 프로퍼티 위임을 사용해 변경한 예.
게터는 getValue, 세터는 setValue 함수를 사용해서 만들어야 함.
게터를 만든 뒤 by를 사용해 getValue, setValue를 클래스와 연결해주면 됨.

```kotlin
@JvmField
private val 'token$delegate' = 
	LoggingProperty<String?>(null)
var token: String?
	get() = 'token$delegate'.getValue(this, ::token)
	set(valuue) {
			'token$delegate'.setValue(this, ::token, value)
	}
```

by를 사용한 token 프로퍼티가 컴파일 되면 다음과 비슷한 형태로 컴파일이 됨.
getValue, setValue는 값처리 만 아니라 컨텍스트와 프로퍼티 레퍼런스의 경계도 함께 사용하는 형태로 바뀜. 
프로퍼티 레퍼런스는 이름, 어노테이션과 관련된 정보.
컨텍스트는 함수가 어떤 위치에서 사용되는지와 관련된 정보.

```kotlin
class SwipeRefreshBindderDelegate(val id: Int) {
	private var cache: Swiperefreshlayout? = null
	
	operator fun getValue(
		activity: Activity,
		prop: KProperty<*>
		): Swiperefreshlayout {
			return cache ?: activity
			.findViewById<Swiperefreshlayout>(id)
			.also { cache = it }
		}
		
		operator fun getValue(
		fragment: Fragment,
		prop: KProperty<*>
		): Swiperefreshlayout {
			return cache ?: fragment
			.findViewById<Swiperefreshlayout>(id)
			.also { cache = it }
		}
	
```
getValue와 setvalue가 여러게 있어도 컨텍스트를 활용함으로서 상황에 맞는 적절한 메소드가 선택이 됨. 

```kotlin
val map: Map<String, Any> = mapOf(
	"name" to "black",
	"kotlinProgrammer" to true
)
val name by map
print(name) //black

inline operator fun <V, V2 : V> Map<in String, V>
.getValue(thisRef: Any?, property:Kproperty<*>): V1 =
getOrImplicitDefault(pproperty.name) as V1
```

멤버 함수뿐 아니라 Map<String, *> 과 같은 확장 함수로도 만들수 있음.
코틀린 stdlib에 확장 함수가 정의되어 있어 사용 가능.

## 요약정리
1. 코틀린은 코드 재사용과 관련해서 프로퍼티 위임이란 기능을 제공.
2.  프로퍼티 델리게이트는 프로퍼티와 컨텍스트에 관련된 정보와 조작을 할수 있음.
3. 일반적인 패턴을 추출하거나 더 좋은 API를 만들때 활용 가능.

