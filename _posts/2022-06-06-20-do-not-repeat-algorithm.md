---
author_name: 수현
github_nickname: soohyeon13
title: 이펙티브 코틀린 - 20. 일반적인 알고리즘을 반복해서 구현하지 말라
excerpt: 반복해서 구현하지 말라
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

## 이미 있는 것을 활용해라

```kotlin
val percent = when {
    numberFromUser > 100 -> 100
    numberFromUser < 0 -> 0
    else -> numberFromUser
}
```

`stdlib` 의 `coerceIn` 확장 함수로 이미 존재 한다.

```kotlin
val percent = numberFromUser.coerceIn(0, 100)
```
- 이미 있는 것을 활용할 때의 장점
1. 코드 작성 속도가 빨라진다.
2. 함수의 이름 등만 보고도 무엇을 하는지 확실하게 알 수 있다.
3. 직접 구현할 때 발생할 수 있는 실수를 줄일 수 있다.
4. 제작자들이 한 번만 최적화하면, 이러한 함수를 활용하는 모든 곳이 최적화의 혜택을 받을 수 있다.

## 표준 라이브러리 살표보기

-`stdlib`?
 - 가장 대표적인 라이브러리는 바로 표준 라이브러리인 `stdlib` 이다.
 - 확장 함수를 활용해서 만들어진 유틸리티 라이브러리이다.

- stdlib의 함수들을 하나하나 보는것은 힘들겠지만 그럴만한 가치가 있는 일이다.

```kotlin
override fun saveCallResult(item: SourceResponse) {
    var sourceList = ArrayList<SourceResponse>()
    item.sources.forEach {
        var sourceEntity = SourceEntity()
        sourceEntity.id = it.id
        sourceEntity.category = it.category
        sourceEntity.country = it.country
        sourceEntity.description = it.description
        sourceList.add(sourceEntity)
    }
    db.insertSources(sourceList)
}
```

`forEach` 보다는 `map` 함수를 사용하는 것이 좋다. 
SourceEntity를 설정하는 부분에서 이런 형태보다는 팩토리 메서드를 활용하거나, 기본 생성자를 활용하는 것이 좋다.
`apply`를 사용해서 만들 수도 있다.

```kotlin
override fun saveCallResult(item: SourceResponse) {
    val sourceEntries = item.sources.map(::sourceToEntry)
    db.insertSources(sourceEntries)
}

private fun sourceToEntry(source: Source) = SourceEntity()
    .apply {
        id = it.id
        category = it.category
        country = it.country
        description = it.description
    }
```

## 나만의 유틸리티 구현하기

상황에 따라 표준 라이브러리에 없는 알고리즘이 필요할 수도 있다.

```kotlin
fun Iterable<Int>.product() = 
    fold(1) { acc, i -> acc * i }
```

- 동일한 결과를 얻는 함수를 여러 번 만드는 것은 잘못된 일이다.
`모든 함수는 테스트되어야 하고 기억되어야 하며 유지보수되어야 하기 때문이다.`

- 많이 사용되는 알고리즘을 추출하는 방법으로는 톱레벨 함수, 프로퍼티 위임, 클래스 등이 있다. 이러한 방법들과 비교했을때 확장 함수의 장점
1. 함수는 상태를 유지하지 않으므로, 행위를 나타내기 좋다. 특히 부가작용(side-effect)이 없는 경우에는 더 좋다.
2. 톱레벨 함수와 비겨해서, 확장함수는 구체적인 타입이 있는 객체에만 사용을 제한할 수 있다.
3. 수정할 객체를 아규먼트로 전달받아 사용하는 것보다는 확장 리시버로 사용하는 것이 가독성 측면에서 좋다.
4. 확장 함수는 객체에 정의한 함수보다 객체를 사용할 때, 자동 완성 기능 등으로 제안이 이루어지므로 쉽게 찾을 수 있다.
  4-1. `TextUtils.isEmpty("Text")` 보다는 `"Text".isEmpty()`가 더 사용하기 쉽다.

## 요약 정리
1. 일반적인 알고리즘을 반복해서 만들지 말아야 한다.
2. 대부분 stdlib에 이미 있을 가능성이 높다.
3. stdlib 에 없는 일반적인 알고리즘은 확장 함수로 정의하는 것이 좋다.
