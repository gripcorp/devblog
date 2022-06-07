---
author_name: 수현
github_nickname: soohyeon13
title: 이펙티브 코틀린 - 19.knowledge를 반복하여 사용하지 말라
excerpt: 극단적인 것은 언제나 좋지 않다.
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

### "프로젝트에서 이미 있던 코드를 복사해서 붙여넣고 있다면, 무언가가 잘못된 것이다."

## knowledge

- knowledge는 넓은 의미로 `의도적인 정보`

1. 로직: 프로그램이 어떠한 식으로 동작하는지와 프로그램이 어떻게 보이는지
2. 공통 알고리즘: 원하는 동작을 하기 위한 알고리즘

- 시간에 따른 변화

비지니스 로직은 시간이 지나면서 계속해서 변하지만, 공통 알고리즘은 한 번 정의된 이후에는 크게 변하지 않는다.

## 모든 것은 변화한다

- 프로그래밍에서 유일하게 유지되는 것은 `변화한다는 속성`이라는 말이 있다. 
- 모든 것은 변화하고, 우리는 이에 대비해야 한다. 변화할 때 가장 큰 적은 knowledge가 반복되어 있는 부분이다.
반복되어 있는 코드를 수정하려면 어떻게 해야할까 가장 간단한 방법은 모두 찾고 모두 수정하는것이다. 
`하지만 실수하기 쉽고 귀찮다.`

knowledge 반복은 프로젝트의 확장성(scaleable)을 막고, 쉽게 깨지게(fragile) 만든다.

## 언제 코드를 반복해도 될까?

두 코드가 같은 knowledge를 나타내는지, 다른 knowledge를 나타내는지는 
`"함께 변경될 가능성이 높은가? 따로 변경될 가능성이 높은가?"`
코드를 추출하는 이유는 변경을 쉽게 만들기 위함이다. 

## 단일 책임 원칙(Single Responsibility Principle, SRP)

단일 책임 원칙이란? 
- 클래스를 변경하는 이유는 단 한 가지여야 한다

클린아키텍쳐를 보면 `두 액터(actor)가 같은 클래스를 변경하는 일은 없어야 한다` 
개발자들이 같은 코드를 변경하는 것은 굉장히 위험한 일이다.


```kotlin
class Student {
    // ...
    
    fun isPassing(): Boolean = 
        calculatePointsFromPassedCourses() > 15
        
    fun qualifiesForScholarship(): Boolean = 
        calculatePointsFromPassedCourses() > 30
        
    private fun calculatePointsFromPassedCourses(): Int{
        //...
    }
}
```

- `qualifiesForScholarship()`은 장학금 관련 부서에서 만든 프로퍼티로, 학생이 장학금을 받을 수 있는 포인트를 갖고 있는지 나타낸다.
- `isPassing()`은 인증관련 부서에서 만든 프로퍼티로, 학생이 인증을 통과했는지를 나타낸다.

`private` 함수에 두 가지 이상의 책임이 있다.

```kotlin
// accreditations 모듈
fun Student.qualifiesForScholarship(): Boolean {
    /*...*/
}

// scholarship 모듈
fun Student.calculatePointsFromPassedCourses(): Boolean {
    /*...*/
}
```

책임에 따라 StudentIsPassingValidator 와 StudentQualifiesForScholarshipValidator 클래스로 분리할 수 있다.
또는 확장 함수를 활용하여 분리해서 사용한다.

- 단일 책임 원칙은 두 가지 사실을 알려준다.
1. 서로 다른 곳에서 사용하는 knowledge는 독립적이로 변경할 가능성이 많다. 따라서 비슷한 처리를 하더라도, 완전히 다른 knowledge로 취급하는 것이 좋다.
2. 다른 knowledge는 분리해 두는 것이 좋다. 그렇지 않으면, 재사용해서는 안되는 부분을 재사용하려는 유혹이 발생할 수 있다.

## 요약 정리

- 모든 것은 변화한다. 따라서 공통 knowledge가 있다면 이를 추출해서 이러한 변화에 대비해야 한다.
- 여러 요소에 비슷한 부분이 있는 경우, 변경이 필요할 때 실수가 발생할 수 있다. 이런 부분은 추출하는 것이 좋다.
- 의도하지 않은 수정을 피하려면 또는 다른 곳에서 조작하는 부분이 있다면 분리해서 사용하는 것이 좋다.
- 극단적인 것은 언제나 좋지 않다. 균형이 중요하다.
