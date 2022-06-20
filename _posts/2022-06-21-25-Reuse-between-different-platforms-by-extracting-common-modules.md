---
author_name: 승열
github_nickname: seung-yeol
title: 이펙티브 코틀린 - 25.공통 모듈을 추출해서 여러 플랫폼에서 재사용하라
excerpt: 플랫폼 간 공통되는 코드는 모듈화하여 관리하라
categories: [posts, kotlin, effective-kotlin]
tags: [kotlin, effective-kotlin]
header_image: effective-kotlin-header.jpeg
---

기업은 한 플랫폼만을 대상으로 애플리케이션을 만드는 경우는 없습니다.

최소 두개의 애플리케이션은 서로 소통하고, 재사용할 수 있는 부분이 많을 것이므로 소스코드를 공유할 수 있다면, 큰 이득이 발생할 것입니다.


## 예시

### 풀스택 개발
웹사이트 개발로서 사용되는 대표적인 언어로는 자바스크립트(프론트), 자바(백엔드)가 있으며, 코틀린/js 라이브러리와 kotlin 서버를 사용해 서로 공통되는 코드를 추출하여 재사용 할 수 있습니다.
- 스프링, ktor 등
 
### 모바일 개발
android와 ios의 경우 거의 대부분의 동일한 동작을 하며 비슷한 로직을 사용합니다. 코틀린의 멀티플랫폼 기능을 활용하여 공통 모듈을 만들어 활용할 수 있습니다.
- 코틀린을 사용한 안드로이드 개발, 코틀린/네이티브를 통해 Object-c/스위프트로 ios 개발


<img width="700" alt="image" src="https://user-images.githubusercontent.com/18206333/174615087-d3e276bb-11bd-4d8b-b42d-00bde906a2d9.jpg">

  
## 요약 정리

#### 1. 공통 모듈을 추출해서 여러 플랫폼에서 재사용하라
  
