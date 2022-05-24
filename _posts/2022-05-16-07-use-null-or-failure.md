---
author_name: 김종민
github_nickname: jongmin-kim-grip
title: 이펙티브 코틀린 - 7. 문제상황 결과는 null 혹은 Failure를 사용합시다
excerpt: 문제가 발생했을 때 리턴 값을 어떻게 해야할까요?
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 문제상황 처리

### 올바른 결과를 줄 수 없는 경우들
- 인터넷 연결 문제로 서버로부터 데이터를 받지 못한 경우
- 조건에 맞는 첫 번째 요소를 못찾은 경우
- 텍스트 형식이 맞지 않아 파싱을 제대로 하지 못한 경우

### 문제상황 처리 방법?
- `null` 또는 실패를 나타내는 `sealed` 클래스를 리턴합니다.
- 혹은, 예외를 `throw`합니다.

## 예외

### 예외와 에러
- 예외란 입력 값 혹은 참조 값이 잘못되어 정상적인 프로그램 처리가 불가능한 경우를 말합니다.
- 개발자는 보통 예외에 대한 올바른 처리를 프로그램에 구현 해줍니다.
- 에러란 시스템에 비정상적인 상황이 발생하여 프로그램 처리가 불가능한 경우를 말합니다.
- 보통 에러는 프로그램 코드에서 처리를 구현하지 않습니다. 

### 예외의 종류
- 예외는 크게 `checked` 예외와 `unchecked` 예외로 나뉩니다.
- 자바에서 `Error`와 `RuntimeException`을 상속받은 경우 `unchecked` 예외에 속하며 그 외에는 `checked` 예외입니다.
- `checked` 예외의 경우 반드시 사용하는 코드에서 try-catch 블록으로 처리해야합니다.
- `unchecked` 예외의 경우 처리할지 말지는 개발자의 재량에 따릅니다.

```java
public void main() {
    throw new Exception(); // Compile error
    throw new RuntimeException();
}
```

### 예외로 정보 전달을 하면 안되는 이유
- 예외는 잘못된 특별한 상황을 나타내야 합니다.
- 코틀린의 모든 예외는 `unchecked` 예외입니다.
- 따라서, 많은 개발자가 예외가 전파되는 과정을 제대로 추적하지 못합니다.
- 또한 예외를 처리하기 위한 `try-catch` 블록 내부에 코드를 배치하면, 컴파일러 최적화가 제한됩니다.

## null과 Failure

### 언제 사용해야할까요?
- 충분히 예측할 수 있는 범위의 문제상황에는 `null` 또는 `Failure`를 사용하는게 좋습니다.
- 예측하기 어렵고 예외적인 경우에 대한 오류만 예외를 던지는게 좋습니다.
- 예외 상황에 대해서 추가 정보가 필요하지 않다면 `null`을 정보를 전달해야 한다면 `Failure`를 이용합니다.
- 만약 `null`을 처리해야 한다면, 안전 호출 혹은 엘비스 연산자를 이용할 수 있습니다.
- `Result` 클래스를 리턴한다면, `when` 표현식을 이용할 수 있습니다.

```kotlin
// Return null
users.firstOrNull()?.let(::println) ?: println("empty")

// Return result
sealed class GetUserResult {
    class Success(val user: User): Result()

    class Failure(val throwable: Throwable): Result()
}

when (result) {
    is Success -> println(result)
    is Failure -> println("fail")
}
```

### 왜 좋을까요?
- 이런 처리 방식은 `try-catch` 블록보다 효율적이고 명확하며 사용하기 쉽습니다.
- 예외는 개발자의 실수로 놓칠 수도 있어 프로그램을 중지시킬 수 있습니다.
- `null` 값과 `sealed result` 클래스는 명시적으로 처리하도록 강요하며 프로그램을 중지하지 않습니다.

## 요약정리
1. 예외는 잘못된 특별한 상황에만 사용합시다.
1. 문제상황에 대한 정보 전달이 필요 없을때는 `null`을 리턴합시다.
1. 문제상황에 대한 정보 전달이 필요할 경우에는 `result sealed` 클래스를 리턴합시다.
