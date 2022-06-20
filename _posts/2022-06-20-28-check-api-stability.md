---
author_name: 김종민
github_nickname: jongmin-kim-grip
title: 이펙티브 코틀린 - 28. API 안정성을 확인합시다
excerpt: 모두가 도전적인 것을 말하지만 사실은 안정적이고 싶어요
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---
## 안정적 API를 원하는 이유
- API가 자주 변경되면, 콜러의 여러 코드를 수동으로 변경해야 합니다.
- 많은 콜러가 존재하면 변경하는 작업과 테스트를 위해 많은 노력이 필요합니다.
- 그렇다고 무조건 오래된 API를 계속해서 제공할 수 없습니다.
- 심각한 보안적인 문제가 있을 수도 있고, 굳이 바꿀 필요가 없다면 콜러는 API 변경을 미루기 때문입니다.
- 특히 개발자들은 변경된 API 학습을 위한 시간이 필요합니다.
- 따라서, 보통 우리는 보다 안정적이고 변화가 없는 API를 선호합니다.

## API를 안정적으로 배포하기 위한 방법
- API가 불안정하다면 콜러에게 이를 명확하게 알려주는 것이 좋습니다.
- 일반적으로 시멘틱 버저닝을 통해 안정성을 나타냅니다.
- 시멘틱 버저닝은 세 부분(=major.minor.patch)으로 구분합니다.
- 각 부분은 0 이상의 정수로 구성되며 API의 변경이 생길 경우 상황에 따라 구분된 버전을 올립니다.

```text
기존 버전: 1.0.0
이전 버전과 호환이 불가능하여 마이그레이션이 필요한 경우: 2.0.0
이전 버전과 호환이 가능한 기능이 추가되었을 경우: 1.1.0
간단한 버그를 수정했고 영향이 미비한 경우: 1.0.1
```

- 이외에 코드 형상관리에서도 아직 불안정적인 경우 다른 브랜치에 코드를 두는 것이 좋습니다.
- 이후에 어느정도 안정화되면 보다 안정한 브랜치에 머지시켜 최종 배포를 진행합니다.

```text
완전 초기 상태의 불안정한 구현: alpha branch
어느정도 기능이 완성형에 가깝고 약간의 버그가 있는 구현: beta branch
기능이 완전이 안정적인 구현: release branch
```

- 코드 레벨에서는 불안정한 API에 `Experimental` 어노테이션을 붙여주는 것이 좋습니다.
- 이를 사용하는 콜러에서 컴파일 단계에 경고 또는 오류가 출력될 수 있습니다.

```kotlin
@Experimental(level = Experimental.Level.WARNING)
suspend fun getUser(): User
```

- 반대로 기존에 제공하던 API를 제거해야 하는 경우, `Deprecated` 어노테이션을 붙여주는 것이 좋습니다.
- 이를 사용하는 콜러에서 컴파일 단계에 경고를 출력할 수 있습니다.
- 이미 안정적인 다른 대체제가 있는 경우 `ReplaceWith`를 제공하면 더 좋습니다.
- 이후 충분한 시간이 흘러 콜러가 더 이상 해당 API를 참조하지 않을 적당한 때에 제거합니다.

```kotlin
@Deprecated("Stop using this method", ReplaceWith("getUserV2()"))
fun getUser(): User
```

## 요약정리
1. 콜러는 안정적인 API를 원합니다.
2. 그러나 처음부터 안정적인 API를 제공하기는 힘듭니다.
3. API의 변경이 필요한 경우, 단계적으로 콜러가 이해하기 쉽고 따라올 수 있도록 변경해야 합니다.
