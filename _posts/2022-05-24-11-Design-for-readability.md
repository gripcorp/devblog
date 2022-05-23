---
author_name: 애나
github_nickname: jyeon-park
title: 이펙티브 코틀린 - 11.가독성을 목표로 설계하라
excerpt: 개발자는 코드를 작성하는 것보다 읽는데 더 많은 시간을 소모한다.
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

코틀린은 간결성을 목표로 설계된 프로그래밍 언어가 아니라, 가독성(readability)을 좋게 하는데 목표를 두고 설계된 프로그래밍 언어

## 가독성(readability)

가독성이란 코드를 읽고 얼마나 빠르게 이해할 수 있는지를 의미

`개발자가 코드를 작성하는 데는 1분 걸리지만, 이를 읽는 데는 10분이 걸린다`
- 로버트 마틴(Robert C. Martin)의 클린 코드(Clean Code)

코드를 작성하다가 오류가 발생할 때, 오류를 찾기 위해 코드를 작성할 때보다 오랜 시간 코드를 읽는다.
따라서, 프로그래밍은 쓰기보다 읽기가 더 중요하기 때문에 항상 가독성을 생각하면서 코드를 작성해야 한다.

## 인식 부하 감소

```kotlin
// Implementation A
if (person != null && person.isAdult) { 
  view.showPerson(person)
} else {
  view.showError()
}

// Implementation B
person?.takeIf { it.isAdult }
   ?.let(view::showPerson)
   ?: view.showError()
```
- 코틀린 초보자에게는 일반적인 관용구(if/else, &&, 메서드 호출)를 사용하는 구현 A가 더 읽기 쉬움
- 구현 B는 코틀린에서 일반적으로 사용되는 관용구(안전 호출 ?., takeIf, let, Elvis 연산자, 제한된 함수 레퍼런스 view::showPerson)를 사용하고 있어 숙력된 개발자는 쉽게 읽을 수 있음
- 숙련된 개발자만을 위한 코드는 좋은 코드가 아니므로 A가 훨씬 가독성이 좋은 코드
- 사용빈도가 적은 관용구는 코드를 복잡하게 만들며 그런 관용구들을 한 문장 내부에 조합해서 사용하면 복잡성은 훨씬 더 증가함

### 1. 구현 A는 수정하기 쉬움
```kotlin
if (person != null && person.isAdult) { 
  view.showPerson(person) 
  view.hideProgressWithSuccess()
} else { 
  view.showError() 
  view.hideProgress()
}

person?.takeIf { it.isAdult } 
  ?.let {
    view.showPerson(it)
    view.hideProgressWithSuccess()
   } ?: run {
       view.showError()
       view.hideProgress()
   }
```

### 2. 구현 A는 디버깅도 더 간단함 
- 일반적으로 디버깅 도구조자 이러한 기본 구조를 더 잘 분석해 줌
- '굉장히 창의적인' 구조는 유연하지 않고, 지원도 제대로 받지 못함
- person이 null이 아닐 경우 성인인지 아동인지에 따라서 다른 처리를 하게 하는 조건문을 추가할때 A는 리팩터링 기능을 활용해서 간단히 할 수 있지만 B는 다시 작성해야 할 수 도 있다

### 3. 구현 A와 구현 B는 실행 결과가 다름
let은 람다식 결과를 리턴함
showPerson이 null을 리턴하면, 두번째 구현 때는 showError도 호출함.

### 정리
우리의 뇌는 패턴을 인식하고, 패턴을 기반으로 프로그램의 작동 방식을 이해하기 때문에 기본적으로 인지 부하를 줄이는 방향으로 코드를 작성해야 함
자주 사용되는 패턴을 활용하면, 이와 같은 과정을 더 짧게 만들 수 있음
짧은 코드는 빠르게 읽을 수 있지만, 익숙한 코드는 더 빠르게 이해 할 수 있다.


## 극단적이 되지 않기
let으로 인해 예상치 못한 결과가 나올 수 있다고 `let은 절대로 쓰면 안 된다`라고 생각하지 말기
let은 좋은 코드를 만들기 위해 다양하게 활용되는 관용구임
nullable 가변 프로퍼티가 있고, null이 아닐 때만 어떤 작업을 수행해야한다면 안전하게 let을 사용한다
(가변 프로퍼티는 쓰레드와 관련된 문제를 발생할 수 있으므로, 스마트 캐스팅이 불가능)

```kotlin
class Person(val name: String) 
var person: Person? = null

fun printName() { 
  person?.let {
    print(it.name)
  }
}
```

- 연산을 아규먼트 처리 후로 이동시킬 때
- 데코레이터를 사용해서 객체를 랩할 때

```kotlin
students
  .filter { it.result >= 50 }
  .joinToString(separator = "\n") {
    "${it.name} ${it.surname}, ${it.result}"
  }
  .let(::print)

var obj = FileInputStream("/file.gz") 
  .let(::BufferedInputStream) 
  .let(::ZipInputStream) 
  .let(::ObjectInputStream) 
  .readObject() as SomeObject
```

이 코드들은 디버그하기도 어렵고, 경험이 적은 코틀린 개발자는 이해하기 어렵지만 비용을 지불할 만한 가치가 있으므로 사용해도 된다.
정당한 이유 없이 복잡성을 추가되는 비용 지불 할 가치가 없는 코드가 문제가 된다.

## 요약정리
1. 항상 가독성을 생각하면서 코드를 작성하라.
2. 정당한 이유가 있다면 복잡성을 가져도 된다.
