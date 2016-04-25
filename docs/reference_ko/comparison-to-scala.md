---
type: doc
layout: reference
category: FAQ
title: "스칼라와 비교"
---

# 스칼라와 비교

코틀린 팀의 주요 목적은 실용적이고 생산적인 프로그래밍 언어를 만드는 것이다. 프로그래밍 언어 연구 용으로 앞서가는 최신 언어를 만드는 건 목적이 아니다.
이를 고려해서, 만약 스칼라에 만족한다면, 아마도 코틀린이 필요하지 않을 것이다.

## 코틀린에 없고 스칼라에 있는 것

* 암묵적 변환, 파라미터, etc
    * 스클라는, 때때로 디버거 없이 코드에서 무슨 일이 벌어지고 있는지 말하기 어렵다. 왜냐면 동작하기 위해 암묵적으로 너무 많은 것이 일어나기 때문이다.
    * 코틀린에서 타입에 함수를 추가하고 싶으면 [확장 함수](extensions.html)를 사용한다.
* 오버라이딩할 수 있는 타입 멤버
* 경로-의존 타입(Path-dependent types)
* 매크로
* Existential types
    * [Type projections](generics.html#type-projections)은 매우 특수한 경우이다
* 트레잇 초기화를 위한 복잡한 로직
    * [클래스와 상속](classes.html) 참고
* 커스텀 오퍼레이션 심볼
    * [연산자 오버로딩](operator-overloading.html) 참고
* XML 기본 지원
    * [Type-safe Groovy-style builders](type-safe-builders.html) 참고
* Structural types
* 값 타입(Value types)
    * [Project Valhalla](http://openjdk.java.net/projects/valhalla/)가 JDK에 일부로 릴리즈되면 지원할 계획이다.
* Yield 연산자
* 액터
    * JVM에 액터를 지원하기 위한 외부 프레임워크인 [Quasar](http://www.paralleluniverse.co/quasar/)를 지원한다.
* 병렬 콜렉션
    * 코틀린은 유사한 기능을 제공하는 자바 8 스트림을 지원한다

## 스칼라에 없고 코틀린에 있는 것

* [오버헤드 없는 null-안정성](null-safety.html)
    * 스칼라는 신택틱(syntactic)하고 런타임 래퍼인 Option을 갖는다.
* [스마트 변환](typecasts.html)
* [코틀린의 인라인 함수는 non-로컬 점프를 쉽게할 수 있다](inline-functions.html#inline-functions)
* [일급 위임](delegation.html). 또한 외부 플러그인인 Autoproxy로 구현된다.
* [멤버 참조](reflection.html#function-references) (자바 8에서도 지원한다).
