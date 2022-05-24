---
author_name: 김종민
github_nickname: jongmin-kim-grip
title: 이펙티브 코틀린 - 3. 최대한 플랫폼 타입을 사용하지 맙시다 
excerpt: 코틀린을 자바와 함께 사용할 때 발생할 수 있는 문제는? 
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 널 안정성

### 코틀린은 널 안정일까?
- 코틀린은 널 안정성을 보장하기 때문에 자바와 다르게 널 포인터 예외를 만나기 힘듭니다.
- 하지만, 널 안정성을 보장하지 않는 언어와 함께 사용할 경우 문제가 생길 수 있습니다.

```java
public class JavaCallee {
    public String getUnknownText() {
        return "?";
    }

    @NotNull
    public String getNotNullText() {
        return "Not Null";
    }

    @Nullable
    public String getNullableText() {
        return null;
    }
}
```

```kotlin
fun main() {
    val javaCallee = JavaCallee()
    val unknownText = javaCallee.unknownText() // String!
    val notNullText = javaCallee.notNullText() // String
    val nullableText = javaCallee?.nullableText() // String?
}
```

### 콜렉션 일 경우 더 어렵습니다
- 자바에서는 모든 것이 기본적으로 널러블 입니다.
- 특히 제네릭 타입의 콜렉션을 사용할 경우 문제가 됩니다.
- 콜렉션 안에 들어있는 객체들이 널인지 아닌지 확인해야하기 때문입니다.

```java
public class UserRepo {
    @NotNull
    private List<User> users;

    @NotNull
    public List<User> getUsers() {
        return users;
    }
}
```

```kotlin
fun main() {
    val userRepo = UserRepo()
    val users = userRepo.users // List<User!>
    val notNullUsers = users.filterNotNull() // List<User>
}
```

## 플랫폼 타입

### 플랫폼 타입이란?
- 코틀린은 널 안정성을 지원하지 않는 언어와 함께 사용될 수 있습니다.
- 따라서 다른 언어에서 넘어온 타입을 특수하게 다뤄야 했습니다.
- 이런 타입을 코틀린에서는 플랫폼 타입이라 부릅니다.
- 플랫폼 타입은 타입의 이름 뒤에 `!` 기호를 붙여 표기합니다.

### 플랫폼 타입의 특징
- 플랫폼 타입을 사용할 경우 항상 주의를 기울여야 합니다.
- 가능한 자바 코드에서 `@NotNull` 혹은`@Nullable` 어노테이션을 붙여 플랫폼 타입을 없애주세요.
- 또한, 가능하다면 코틀린 코드로 리팩토링하는 것이 가장 안전합니다.

### 플랫폼 타입의 에러 위치
- 스테이티드 타입은 자바에서 값을 가져오는 위치에서 널 포인터 예외가 발생합니다.
- 플랫폼 타입은 값이 사용되는 곳에서 널 포인터 예외가 발생합니다.
- 플랫폼 타입의 경우 콜리의 코드부가 아닌 콜러의 코드부에서 예외가 발생하기 때문에 추적하기 어려워집니다.
- 따라서 플랫폼 타입을 다른 곳에 전파되지 않도록 유지하는 것이 중요합니다.

```java
public class JavaCallee {
    public String getValue() {
        return null;
    }
}
```

```kotlin
fun callValueAsStatedType() {
    val value: String = JavaCallee().value
    println(value.length)

fun callValueAsPlatformType() {
    val value = JavaCallee().value
    println(value.length)
}
```

## 요약정리
1. 다른 언어에서 온 널러블 여부를 알 수 없는 타입을 플랫폼 타입이라 부릅니다.
1. 플랫폼 타입을 사용하는 코드는 해당 부분 뿐만 아니라, 이를 활용하는 곳까지 영향을 줄 수 있습니다.
1. 플랫폼 타입이 사용되고 있는 코드는 빠르게 제거하는 것이 좋습니다.
1. `@NotNull` 혹은 `@Nullable` 어노테이션을 붙여주거나, 코틀린 코드로 리팩토링합시다.
