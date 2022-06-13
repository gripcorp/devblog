---
author_name: 김준석
github_nickname: idKimjunseok
title: 이펙티브 코틀린 - 22. 일반적인 알고리즘을 구현할 때 제네릭을 사용해라.
excerpt: 제네릭을 사용하여 메서드를 안정하게 사용하자.
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 제너릭 함수
```kotlin
inline fun <T> Iterable<T>.filter(
	predicate: (T) -? Boolean
): List<T> {
	val destination = ArrayList<T>()
	for (element in this) {
		if(predicate(element)) {
			destination.add(element)
		}
	}
	retuen destination
}
	
```
타입 아규먼트(타입 파라미터)를 사용하는 함수를 제네릭 함수라고 부름.
대표적인 예로 filter함수가 있고 타임 파라미터로 T를 가짐.
컴파일러에 타입과 관련된 정보를 제공하여 조금이라도 더 정확하게 추축할수 있게 해줌.
개발자는 프로그래밍이 편해지지만. 런타임때는 어떤 이득도 없음.
제네릭은 기본적으로 구제적인 타입의 컬렉션을 만들수 있게 클래스와 인터페이스에 도입된 기능.

## 제너릭 제한

```kotlin
fun <T: Comparable<T>> Iterable<T>.sorted(): List<T> {
	/*...*/
}

fun <T, C : MutableCollection<in T>>
Iterable<T>.toCollection(destination: C): C {
	/*...*/
}

class ListAdapter<T: ItemAdapter>(/*...*/) { /*...*/ }
```
타입 파라미터의 중요한 기능중 하나로 콜론 뒤에 슈퍼타입을 설정하여 서브타입만 사용하게 타입을 제한 함.
타입 제한으로 인하여 내부에서 해당 타입이 제공하는 메서드를 사용할수 있음.

```kotlin
inline fun <T,R : Any> Iterable<T>.mapNotNull(
	transform: (T) -> R?
): List<R> {
	retuen mapNotNullTo(ArrayList<R>(), transform)
}
```
많이 사용하는 제한으로는 Any가 있으며 이는 nullable이 아닌 타입을 나타냄.

```kotlin
fun <T: Animal> pet(animal: T) where T: GoodTempered {
	/*...*/
}
// 또는
fun <T> pet(animal: T) where T: Animal, T:GoodTempered {
	/*...*/
}
```
드물지만 둘 이상의 제한을 걸 수도 있음.

## 요약정리
1. 타입 파라미터를 사용해 type-safe 제네릭 알고리즘과 객체를 구현함.
2. 타입 파라미터는 구체 자료형의 서브타입을 제한할수 있음.
3. 특정 자료형이 제공하는 메서드를 안전하게 사용 가능.

