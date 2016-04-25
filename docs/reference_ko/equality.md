---
type: doc
layout: reference
category: "Other"
title: "Equality"
---

# Equality

코틀린은 두 가지 동등성이 있다:
In Kotlin there are two types of equality:

* 참조 동등성Referential equality (두 레퍼런스가 같은 객체를 참고)
* 구조적 동등성 (`equals()`로 검사)

## 참조 동등성

참조 동등성은 `===` 오퍼레이션으로 검사한다(역은 `!==`)
`a === b`는 `a`와 `b`가 같은 객체를 참조하는 경우에만 true이다.

## 구조적 동등성

구조적 동등성은 `==` 오퍼레이션으로 검사한다(역은 `!=`). 규칙에 따라, `a == b`와 같은 식은 다음으로 변환된다.

``` kotlin
a?.equals(b) ?: (b === null)
```

만약 `a`가 `null`이 아니면, 이 코드는 `equals(Any?)` 함수를 호출하고, 그렇지 않으면(`a`가 `null`이면) `b`가 `null`을 참조하는지 검사한다.

`null`을 직접 비교할 때 코드 최적화 포인트는 없다. `a == null`을 자동으로 `a === null`로 변환해준다.
