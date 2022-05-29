
---
author_name: 승열
github_nickname: seung-yeol
title: 이펙티브 코틀린 - 13.Unit?을 리턴하지 말라
excerpt: 
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## Unit 
java의 void를 대체하여 사용하는 유형으로 아무런 값도 리턴하지 않는다는 뜻으로 사용하고 있다.

굳이 null에 의미를 부여하여 사용을 한다고 하였을 때,
```kotlin
fun keyIsCorrect(key: String): Boolean = //...

if(!keyIsCorrect(key)) return
```
이와 같은 Boolean을 리턴하는 메서드를 Unit?으로 대체하여 사용한다면

```kotlin
fun verifyKey(key: String): Unit? = //...

verifyKey(key) ?: return 
```
이와 같이 표현이 될 수 있습니다.

```kotlin
if(!keyIsCorrect(key)) return

verifyKey(key) ?: return 
```
두가지를 비교했을 때, 위의 if문은 한번에 이해가 되는 반면,
verifyKey()의 경우 리턴값이 유효한 값을 갖고있는 객체를 리턴하는 것으로 오해할 수 있어 보입니다.

Unit으로 Boolean을 표현하는 것은 오해의 소지가 있으며, 예측하기 어려운 오류를 생성 할 수 있습니다.
명확히 표현할 수 있는 Boolean이 있는데 굳이 null을 통해 표현을 할 필요가 없습니다.

## 요약 정리
- Unit?을 리턴하지 말자
