---
author_name: 애나
github_nickname: jyeon-park
title: 이펙티브 코틀린 - 24.제네릭 타입과 variance 한정자를 활용하라
excerpt: variance 한정자(out or in) 
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## variance 한정자(out or in)

### invariant(불공변성)

```kotlin
class Cup<T>
```
- **타입 파라미터 T**는 variance 한정자(out or in)가 없으므로 기본적으로 **invariant**(불공변성)이다.
- invariant는 제네릭 타입으로 만들어지는 타입들이 서로 관련성이 없다는 의미
- ex, Cup<**Int**>와 Cup<**Number**>, Cup<**Any**>와 Cup<**Nothing**>은 어떤 관련성도 없음

```kotlin
fun main() {
  val anys: Cup<Any> = Cup<Int>() // Error: Type mismatch 
  val nothings: Cup<Nothing> = Cup<Int>() // Error
}
```

관련성을 원한다면, out or in 이라는 variance 한정자를 붙임

### covariant(공변성)
  
```kotlin
class Cup<out T> 
open class Dog 
class Puppy: Dog()

fun main(args: Array<String>) {
  val b: Cup<Dog> = Cup<Puppy>() // OK 
  val a: Cup<Puppy> = Cup<Dog>() // Error
  
  val anys: Cup<Any> = Cup<Int>() // OK
  val nothings: Cup<Nothing> = Cup<Int>() // Error 
}
```
- out은 타입 파라미터를 **covariant(공변성)**으로 만듦
- A가 B의 서브타입이고 Cup이 covariant(out)라면, Cup<**A**>가 Cup<**B**>의 **서브 타입이**라는 의미
  
### contravariant(반변성)   

```kotlin
class Cup<in T> 
open class Dog 
class Puppy(): Dog()

fun main(args: Array<String>) {
  val b: Cup<Dog> = Cup<Puppy>() // Error val a: Cup<Puppy> = Cup<Dog>() // OK
  val anys: Cup<Any> = Cup<Int>() // Error
  val nothings: Cup<Nothing> = Cup<Int>() // OK 
}
```

- in 한정자는 반대 의미 
- 타입 파라미터를 contravariant(반변성)을 만듦
- A가 B의 서브타입이고 Cup이 contravariant(in)라면, Cup<**A**>가 Cup<**B**>의 **슈퍼 타입이**라는 의미

### variance 한정자

<img width="700" alt="image" src="https://user-images.githubusercontent.com/100750946/173183606-93354bbe-1092-4784-8c98-31b431e6912c.png">


## 함수 타입

```kotlin
fun printProcessedNumber(transition: (Int)->Any) { 
  print(transition(42))
}
```
- Int를 받고, Any를 리턴하는 함수를 파라미터로 받는함수
- (Int)->Any 타입의 함수는 (Int)->Number, (Number)->Any, (Number)->Number, (Number)->Int 등으로 작동함

```kotlin
val intToDouble: (Int) -> Number = { it.toDouble() }
val numberAsText: (Number) -> Any = { it.toShort() }
val identity: (Number) -> Number = { it }
val numberToInt: (Number) -> Int = { it.toInt() }
val numberHash: (Any) -> Number = { it.hashCode() }

printProcessedNumber(intToDouble)
printProcessedNumber(numberAsText)
printProcessedNumber(identity)
printProcessedNumber(numberToInt)
printProcessedNumber(numberHash)
```
<img width="600" alt="image" src="https://user-images.githubusercontent.com/100750946/173183783-d5b47deb-d39d-4fa1-b74d-dceb4d6f8f79.png">

- **파라미터 타입**이 더 **높은 타입**으로 이동
- **리턴 타입**은 계층 구조의 더 **낮은 타입**으로 이동

### 코틀린 타입 계층

<img width="600" alt="image" src="https://user-images.githubusercontent.com/100750946/173183887-d18d34e4-ee57-42b8-ab91-215a7a7a1e2e.png">

- 코틀린 함수 타입의 모든 **파라미터 타입**은 contravariant(in)
- 모든 **리턴 타입**은 covariant(out)  

<img width="400" alt="image" src="https://user-images.githubusercontent.com/100750946/173183947-23b2eb62-6d06-451f-a6fd-c43d20eca697.png">

- 함수 타입을 사용할 때는 자동으로 variant 한정자가 사용됨
- covariant(out)를 가진 List는 variant 한정자가 붙지 않은 MutableList와 다르며 List가 더 많이 사용됨

## variance 한정자의 안전성

### java 배열

자바의 배열은 **covariant(out)** 이며, 배열을 기반으로 제네릭 연산자는 정렬 함수 등을 만들기 위해서라고 함. 이러한 속성 때문에 큰 문제가 발생됨

```java
// Java
Integer[] numbers = {1, 4, 2, 1};
Object[] objects = numbers;
objects[2] = "B"; // Runtime error: ArrayStoreException
```

- 컴파일 중에는 문제 없지만 런타임 오류가 발생
- number를 Object[]로 캐스팅해도 실질적인 타입이 바뀌는 것이 아니며, 여전히 Integer이라서 string 타입의 값을 할당하면 오류가 발생
- 이는 자바의 명백한 결함임

### kotlin 배열

- 코틀린은 이러한 결함을 해결하기 위해서 Array(IntArray, CharArray 등)를 **invariant(in)** 으로 구현됨
- 따라서 Array<**Int**>가 Array<**Any**> 등으로 바꿀수 없음

파라미터 타입을 예측할 수 있다면, 어떤 서브타입이라도 전달할 수 있음. (암무적으로 **업캐스팅**)

```kotlin
open class Dog
class Puppy: Dog()
class Hound: Dog()

fun takeDog(dog: Dog) {} 

takeDog(Dog())
takeDog(Puppy())
takeDog(Hound())
```

- covariant (out) 타입 파라미터가 in 한정자 위치(예를들어 타입 파라미터)에 있다면, covariant와 업캐스팅을 연결해서 우리가 원하는 타입을 아무것이나 전달 가능
- 따라서, value는 매우 구체적인 타입이라서 value에 Dog 타입으로 지정하면 String 타입을 넣을 수 없음

```kotlin
class Box<out T> {
  private var value: T? = null

  // Illegal in Kotlin
  fun set(value: T) { // Error: Type parameter T is declared as 'out' but occurs in 'in' position in type T
    this.value = value
  }
  
  fun get(): T = value ?: error("Value not set") 
}

val puppyBox = Box<Puppy>()
val dogBox: Box<Dog> = puppyBox
dogBox.set(Hound()) // But I have a place for a Puppy

val dogHouse = Box<Dog>()
val box: Box<Any> = dogHouse
box.set("Some string") // But I have a place for a Dog
box.set(42) // But I have a place for a Dog
```

- 캐스팅 후에 실질적인 객체가 그대로 유지되고, 타이핑 시스템에서만 다르게 처리되므로 안전하지 않음
- Int를 설정하려고하는데, 해당 위치는 Dog만을 위한 자리임
- 이것이 가능하면 오류가 발생할 것임

```kotlin
class Box<out T> {
  var value: T? = null // Error: Type parameter T is declared as 'out' but occurs in 'invariant' position in type T?

  fun set(value: T) { // Error: Type parameter T is declared as 'out' but occurs in 'in' position in type T
    this.value = value
  }

  fun get(): T = value ?: error("Value not set")
}
```

- **코틀린은 public in 한정자 위치에 covariant(out) 타입 파라미터가 오는 것을 금지**하여 이러한 상황을 막음


```kotlin
class Box<out T> {
  private var value: T? = null
  
  private set(value: T) { 
    this.value = value
  }
  
  fun get(): T = value ?: error("Value not set") 
} 
```

- **private로 제한**하면, 오류가 발생하지 않음
- 객체 내부에서는 **업캐스트 객체**에 covariant(out)를 사용할 수 없음
- 생성 되거나 노출되는 타입에만 covariant(out)를 사용
- producer or immutable 데이터 홀더에서 많이 사용

```kotlin
fun append(list: MutableList<Any>) {
  list.add(42) 
}
val strs = mutableListOf<String>("A", "B", "C")
append(strs) // Illegal in Kotlin : Type mismatch: inferred type is MutableList<String> but MutableList<Any> was expected
val str: String = strs[3]
print(str)
```

- MutableList<T>는 in 한정자 위치에서 사용되며, 안전하지 않으므로 invarinat(불공변성)임

  
### Response

variance 한정자 덕분에 모두 참이 됨
  
- Response<**T**>라면 T의 모든 **서브타입**이 허용됨 
  (ex, Response<**Any**>가 예상된다면, Response<**Int**>와 Responce<**String**>이 허용됨)
- Response<**T1,T2**>라면 T1과 T2의 모든 **서브타입**이 허용됨
- Failure<**T**>라면, T의 모든 **서브타입** Failure가 허용됨
  (ex, Failure<**number**>라면, Failure<**Int**>와 Failure<**Double**>이 모두 허용됨, Failure<**Any**>라면, Failure<**Int**>와 Failure<**String**>이 모두 허용됨)

```kotlin
sealed class Response<out R, out E>
class Success<out R>(val value: R): Response<R, Nothing>()
class Failure<out E>(val error: E): Response<Nothing, E>()
```

- covariant와 Nothing타입으로 인해서 
- Failure는 오류타입을 지정하지 않아도 됨
- Success는 잠재적인 값을 지정하지 않아도 됨

```kotlin
open class Car
interface Boat
class Amphibious: Car(), Boat

fun getAmphibious(): Amphibious = Amphibious()
val car: Car = getAmphibious()
val boat: Boat = getAmphibious()  
```

- covariant(out)와 public in 위치와 같은 문제는 contravariant(in) 타입 파라미터와 public out 위치 (함수 리턴 타입 또는 프로퍼티 타입)에서도 발생
- out 위치는 암무적인 업캐스팅을 허용함
- contravariant(in)에 맞는 동작이 아님
 
  
```kotlin
class Box<in T>(
  // Illegal in Kotlin
  val value: T
)

val garage: Box<Car> = Box(Car())
val amphibiousSpot: Box<Amphibious> = garage
val boat: Boat = garage.value // But I only have a Car

val noSpot: Box<Nothing> = Box<Car>(Car())
val boat: Nothing = noSpot.value
// I cannot produce Nothing!
```
  
- 어떤 상자에 어떤 타입이 들어 있는지 확실하게 알수 없음

 
```kotlin
class Box<in T> {
  var value: T? = null // Error 
  
  fun set(value: T) {
    this.value = value
  }
  
 fun get(): T = value // Error
     ?: error("Value not set")
}
```    

- 코틀린은 **contravariant(in) 타입 파라미터를 public out 한정자 위치에 사용하는 것을 금지**함.  
  
```kotlin
class Box<in T> {
  private var value: T? = null 
  
  fun set(value: T) {
    this.value = value
  }
  
 private fun get(): T = value
     ?: error("Value not set")
}
```     

- private 일 때는 문제가 없음  
  
이런 형태로 타입 파라미터에 contravariant(in)를 사용하는 경우는 kotlin.coroutines.Continuation이 있음

```kotlin
public interface Continuation<in T> {
  public val context: CoroutineContext 
  public fun resumeWith(result: Result<T>)
}
```      

## varinace 한정자의 위치

```kotlin
// 선언 쪽의 varinace 한정자
class Box<out T>(val value: T)
val boxStr: Box<String> = Box("Str")
val boxAny: Box<Any> = boxStr
```    
  
1. 선언부분
- 일반적인 위치
- 클래스와 인터페이스 선언에 한전자가 적용됨
- 클래스와 인터페이스가 사용되는 모든 곳에 영향을 줌 

```kotlin
class Box<T>(val value: T)
val boxStr: Box<String> = Box("Str")

// Use-side variance modifier
val boxAny: Box<out Any> = boxStr
```   

2. 클래스와 인터페이스를 **활용하는 위치**
- 이 위치에 variance 한정자를 사용하면 특정한 변수에만 variance 한정자가 적용 됨
- 모든 인스턴스에 variance 한정자를 적용하지 않고 특정 인스턴스에만 적용 해야할 때 사용
  (ex, MutableList 는 in 한정자를 포함하면, 요소를 리턴 할 수 없으므로 in 한정자를 붙이지 않음)
- 단일 파라미터 타입에 in 한정자를 붙여서 contravariant를 가지게하는 것이 가능
  (여러 가지 타입을 받아 들일 수 있게 됨)
    
```kotlin
interface Dog
interface Cutie
data class Puppy(val name: String): Dog, Cutie
data class Hound(val name: String): Dog
data class Cat(val name: String): Cutie

fun fillWithPuppies(list: MutableList<in Puppy>) {
list.add(Puppy("Jim"))
list.add(Puppy("Beam"))
}

val dogs = mutableListOf<Dog>(Hound("Pluto"))
fillWithPuppies(dogs)
println(dogs)
// [Hound(name=Pluto), Puppy(name=Jim), Puppy(name=Beam)]

val animals = mutableListOf<Cutie>(Cat("Felix"))
fillWithPuppies(animals)
println(animals)
// [Cat(name=Felix), Puppy(name=Jim), Puppy(name=Beam)]  
```    

### variance 한정자를 사용하면 위치가 제한 됨
  
- MutableList<**out T**>가 있다면, get으로 요소를 추출 했을 때 T 타입이 나옴
- 하지만 set은 nothing 타입의 아규먼트가 전달 될 것이라 예상됨으로 사용 할 수가 없음
  (모든 타입의 **서브타입**을 가진 리스트(Nothing 리스트)가 존재할 가능성이 있기 때문)
  
- MutableList<**in T**>를 사용 할 경우, get과 set을 모두 사용 할 수 있음
- 하지만 get를 사용하면 전달 자료형은 Any?가 됨
  (모든 타입의 **슈퍼타입**을 가진 리스트(Any 리스트)가 존재 할 가능성이 있기 때문)
  
## 요약 정리
  
### 1. 타입 파라미터의 기본적인 variance의 동작은 invariant이다

- 만약 Cup<**T**>라고 하면, 타입 파라미터 T는 invariant이다.
- A가 B의 서브타입이라고 할때, Cup<**A**>와 Cup<**B**>는 아무런 관계도 없다.  
  
### 2. out 한정자는 타입 파라미터를 covariant 하게 만듦

- 만약 Cup<**T**>라고 하면, 타입 파라미터 T는 covariant 이다.
- A가 B의 서브타입이라고 할때, Cup<**A**>와 Cup<**B**>는 **서브 타입**이 됨
- covariant 타입은 out 위치에서 사용 할 수 있음
  
### 3. in 한정자는 타입 파라미터를 contravariant 하게 만듦  

- 만약 Cup<**T**>라고 하면, 타입 파라미터 T는 contravariant 이다.
- A가 B의 서브타입이라고 할때, Cup<**B**>는 Cup<**A**>의 **슈퍼 타입**이 됨
- contravariant 타입은 in 위치에서 사용 할 수 있음  
  
### 4. 코틀린에서 List와 Set의 타입 파라미터는 covariant(out 한정자)이다.
  
- * List<**Any**>가 예상되는 모든 곳에 전달 할 수 있음
- * Map에서 값의 타입을 나타내는 타입 파라미터는 covariant(out 한정자)이다 
- * Array, MutableList, MutableSet, MutableMap의 타입 파라미터는 invariant(한정자 지정없음)이다
  
### 5. 함수 타입의 파라미터 타입은 contravariant(in 한정자)이다.
### 6. 함수 타입의 리턴 타입은 covariant(out 한정자)이다.
### 7. 리턴만 되는 타입에는 covariant(out 한정자)를 사용한다.(produced or exposed)
### 8. 허용만(only accepted) 되는 타입에는 contravariant(in 한정자)를 사용한다.  
  
