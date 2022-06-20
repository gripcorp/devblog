---
author_name: 승열
github_nickname: seung-yeol
title: 이펙티브 코틀린 - 26.함수 내부의 추상화 레벨을 통일하라
excerpt: 함수 내부의 작업을 나눌 때, 비슷한 급으로 나누자
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## 추상화 계층

대표적인 추상화 계층의 예시로서, 컴퓨터를 예를 들 수 있습니다.

하드웨어를 통해 데이터를 직접 생성되고 제어를 할 수 있는 어셈블리어로 표현이 되었으며, 개발자들이 제어를 더욱 편안히 할 수 있도록 컴파일언어, 프로그래밍 언어가 만들어졌으며, 
이 언어로 애플리케이션을 만들게 되었습니다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/18206333/174617367-922a67a2-ad51-4783-bbc1-25f0234dc895.jpg">

아주 복잡한 동작들을 비슷한 레벨의 계층으로 나누어서, 상위의 계층은 하위의 계층의 내부동작을 정확히 알지 못해도 사용할 수 있어, 개발자가 해당 계층에서 할 일에만 집중할 수 있도록 합니다.

애플리케이션을 개발하는데, 하위의 계층인 프로그래밍 언어, 어셈블리, 하드웨어의 내부동작이 어떻게 되는지 몰라도, 제공되는 인터페이스를 통해 사용을 할 수 있습니다.


## 추상화 레벨
컴퓨터 과학에서는 높은 레벨로 올라갈수록 물리장치로부터 점점 멀어지고, 생각해야하는 구체적인 내용들이 줄어듭니다. 

즉 높은 레벨일수록 단순해지지만, 하위 레벨의 인터페이스로 부터 제공되지 않는 부분은 제어할 수 없게 됩니다.

컴퓨터 과학처럼 코드도 추상화를 계층처럼 만들어서 사용할 수 있으며, 이를 위한 가장 기본적인 도구가 함수입니다.

```kotlin
class CoffeeMachine {
  fun makeCoffee()}
    //수도꼭지의 물을 틀고
    //커피포트에 물을 채우고
    //커피포트에 전원을 켜서 끓이고
    //커피콩을 갈고 ...
  }
}
```

이렇게 코드를 작성하게되면 함수 한줄이 몇백줄이 될 수 있습니다.

이런함수는 읽고 이해하는 것이 불가능하고, 수정요청이 들어왔을 때, 대응을 할 수 없습니다.

```kotlin
class CoffeeMachine {
  fun makeCoffee()}
    boilWater()
    brewCoffee()
    pourCoffee()
    pourMilk()
  }
  
  private fun boilWater(){...}
  private fun brewCoffee(){...}
  private fun pourCoffee(){...}
  private fun pourMilk(){...}
}
```

이와 같이 계층을 나누면, 내부 함수명을 읽고 어떻게 동작하는지 유추가 가능하게 되고, 수정요청이 들어온다면 대응하기가 쉬워집니다.

```kotlin

  fun makeEspressoCoffee()}
    boilWater()
    brewCoffee()
    pourCoffee()
  }

```

이러한 코드는 전과 비교하여 가독성이 크게 향상이 되었으며, 재사용과 테스트가 쉽습니다.

## 프로그램 아키텍처의 추상 레벨
추상화 계층이라는 개념은 함수보다 높은 레벨에서도 적용할 수 있습니다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/18206333/174631371-4e833181-4966-4a66-b96a-9b8903a40300.jpg">
  
## 요약 정리!

#### 1. 추상화 계층을 만드는 것은 프로그래밍에서 일반적으로 사용되는 개념.

#### 2. 내부 동작을 나누게되면, 재사용과 테스트가 쉬워집니다.

#### 3. 내부 동작을 나눌 떼, 비슷한 레벨로 나누어야 읽기가 쉽고, 동작의 흐름이 잘 보입니다.

#### 4. 각각의 레이어가 너무 커지는 것은 좋지 않으며, 각 함수가 최소한의 책임을 갖는 것이 좋습니다.
  
