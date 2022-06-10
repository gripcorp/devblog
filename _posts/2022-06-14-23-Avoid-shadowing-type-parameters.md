---
author_name: 애나
github_nickname: jyeon-park
title: 이펙티브 코틀린 - 23.타입 파라미터의 섀도잉을 피하라
excerpt: 타입 파라미터가 새도잉되는 경우에는 코드를 주의해서 살펴봐라
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## 섀도잉(shadowing)

```kotlin
class Forest(val name: String) { 
  fun addTree(name: String) {
      // ...
  }
}
```
- 프로퍼티와 파라미터가 같은 이름을 가짐
- 지역 파라미터가 외부 스코프에 있는 프로퍼티를 가리는 것을 **섀도잉(shadowing)**이라고 함

**클래스 타입 파라미터**와 **함수 타입 파라미터** 사이에서도 발생

```kotlin
interface Tree 
class Birch: Tree 
class Spruce: Tree

class Forest<T: Tree> {
  fun <T: Tree> addTree(tree: T) {
      // ...
  }
}
```
- Forest와 addTree의 타입 파라미터가 독립적으로 동작함

```kotlin
val forest = Forest<Birch>()
forest.addTree(Birch())
forest.addTree(Spruce())
```
- 독립적으로 동작하는 것을 빠르게 알아내기 힘듬

```kotlin
class Forest<T: Tree> { 
  fun addTree(tree: T) {
    // ...
  }
}

// Usage
val forest = Forest<Birch>() 
forest.addTree(Birch()) 
forest.addTree(Spruce()) // ERROR, type mismatch
```
- addTree가 클래스 타입 파라미터인 T를 사용하게 하는 것이 좋음

```kotlin
class Forest<T: Tree> {
  fun <ST: Tree> addTree(tree: ST) {
      // ...
  }
```
- 독립적인 타입 파라미터를 의도했다면, 이름을 아예 다르게 다는 것이 좋음

```kotlin
class Forest<T: Tree> {
  fun <ST: T> addTree(tree: ST) {
      // ...
  }
}

// Usage
val forest = Forest<Birch>() 
forest.addTree(Birch()) 
forest.addTree(Spruce()) // ERROR, type mismatch
```
- 타입 파라미터를 사용해서 **다른 타입 파라미터에 제한**을 줌

## 요약 정리

- 타입 파라미터 섀도잉을 피하라
- 타입 파라미터 섀도잉이 발생한 코드는 이해하기 어려울 수 있다
