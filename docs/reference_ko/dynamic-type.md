---
type: doc
layout: reference
category: "Syntax"
title: "동적 타입"
---

# 동적 타입

> 동적 타입은 JVM 대상 코드에서 지원하지 않는다
{:.note}

코틀린은 정적 타입 언어이지만, 자바스크립트 에코시스템과 같이 untyped거나 타입이 유연한 환경에서도 돌아야 한다.
이런 환경을 쉽게 처리하기 위해 언어에서 `동적` 타입을 사용할 수 있다.

``` kotlin
val dyn: dynamic = ...
```

`dynamic` 타입은 기본적으로 코틀린의 타입 검사를 하지 않는다.
  - 이 타입의 값은 모든 변수에 할당하거나 파라미터로 어디든 전달할 수 있다.
  - `dynamic` 타입 변수에 모든 값을 할당할 수 있고, 함수의 `dynamic` 파라미터에 모든 값을 전달할 수 있다.
  - 이 값에는 `null`-검사를 할 수 없다.

`dynamic`의 가장 특별한 기능은 `dynamic` 변수에 대해 파라미터를 갖는 함수나 **모든** 프로퍼티를 호출할 수 있다는 것이다.

``` kotlin
dyn.whatever(1, "foo", dyn) // 어디에도 'whatever'가 정의되어 있지 않음
dyn.whatever(*arrayOf(1, 2, 3))
```

자바스크립트 플랫폼에서, 이 코드는 있는 그대로 컴파일된다. 즉 코틀린의 `dyn.whatever(1)` 코드가 생성한 자바스크립트 코드에서도 `dyn.whatever(1)`가 된다.

동적 호출은 항상 결과로 `dynamic`을 리턴하므로 자유롭게 호출을 연결할 수 있다.

``` kotlin
dyn.foo().bar.baz()
```

동적 호출에 람다를 전달하면 기본적으로 모든 파라미터는 `dynamic` 타입을 갖는다.

``` kotlin
dyn.foo {
  x -> x.bar() // x가 dynamic
}
```

기술적인 내용은 [스펙 문서](https://github.com/JetBrains/kotlin/blob/master/spec-docs/dynamic-types.md)를 참고한다.
