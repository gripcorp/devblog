---
author_name: 애나
github_nickname: jyeon-park
title: 이펙티브 코틀린 - 12.연산자 오버로드를 할 때는 의미에 맞게 사용하라
excerpt: 큰 힘에는 큰 책임이 따른다
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## 팩토리얼 구하는 함수

```kotlin
fun Int.factorial(): Int = (1..this).product()

fun Iterable<Int>.product(): Int =
fold(1) { acc, i -> acc * i }
```
이 함수는 Int 확장 함수로 정의되어 있으므로, 편하게 사용 할 수 있음

```kotlin
print(10 * 6.factorial()) // 7200
```

수학 시간에 팩토리얼을 ! 기호를 사용해 표기

```kotlin
10 * 6!
```

코틀리은 이런 연산자를 지원하지 않지만 연산자 오버로딩을 활용하여 만들 수 있음

```kotlin
operator fun Int.not() = factorial()

print(10 * !6) // 7200
```

가능하지만 해서는 안됨!
함수의 이름이 `not` 이므로 논리 연산에 사용해야지 팩토리얼에 사용하면 안됨

```kotlin
print(10 * 6.not()) // 7200
```

<img width="295" alt="image" src="https://user-images.githubusercontent.com/100750946/169883205-c50c7d15-18a4-4f95-8149-ea77ad8bf3a1.png">

- 코틀린에서 각 연산자의 의미는 항상 같게 유지됨
- 스칼라라 같은 언어는 무제한 연산자 오버로딩을 지원해서 만약 + 연산자가 일반적인 의미로 사용되지 않고 있다면 개별적으로 이해해야하기 때문에 코드를 이해하기 어려움

코틀린에서는 각각의 연산자에 구체적인 의미가 있음

```kotlin
x + y == z
```
위 코드는 우리가 알고 있는 의미랑 같다

```kotlin
x.plus(y).equal(z)
```

구체적인 이름을 가진 함수이며, 모든 연산자가 이러한 이름이 나타내는 역할을 할거라고 기대한다
따라서, 관례에 어긋나기 때문에 팩토리얼을 계산하기 위해서 !연산자를 사용하면 안됨

## 분명하지 않은 경우
관례에 충족하는지 아닌지 확실하지 않은 경우 `infix`를 활용한 확장 함수를 사용하는 것이 좋다

### 함수를 세 배하는 것은 (* 연산자) 무슨 의미 일까?

```kotlin
operator fun Int.times(operation: () -> Unit): ()->Unit =
{ repeat(this) { operation() } }

val tripledHello = 3 * { print("Hello") }

tripledHello() // Prints: HelloHelloHello
```
함수를 세 번 반복하는 새로운 함수를 만들어 낸다고 생각 할 수 있음

```kotlin
operator fun Int.times(operation: ()->Unit) {
  repeat(this) { operation() }
}

3 * { print("Hello") } // Prints: HelloHelloHello
```
함수를 세번 호출 한다는 것으로 생각 할 수 있음

```kotlin
infix fun Int.timesRepeated(operation: ()->Unit) = { 
  repeat(this) { operation() }

val tripledHello = 3 timesRepeated { print("Hello") } 
tripledHello() // Prints: HelloHelloHello
```
`infix`를 활용한 확장 함수로 구현하면 이항 연산자 형태처럼 사용 할 있음

## 요약 정리

- 연산자 오버로딩은 그 이름의 의미에 맞고 사용
- 연산자 의미가 명확하지 않다면 이름이 있는 일반 함수로 사용
- 꼭 연산자 같은 형태로 사용하고 싶다면,`infix` 확장 함수 활용
