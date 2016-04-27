---
type: doc
layout: reference
category: "Syntax"
title: "범위"
---

# 범위

범위(Range) 식은 연산자 형식인 ".."를 갖는 `rangeTo` 함수로 구성된다. 이 식은 *in*{: .keyword }이나 *!in*{: .keyword }과 함께 쓰인다.
모든 Comparable 타입에 대해 범위를 정의할 수 있으며 기본 정수 타입은 최적화한 구현을 제공한다. 다음은 범위 사용 예이다.

``` kotlin
if (i in 1..10) { // 1 <= i && i <= 10와 동일
  println(i)
}
```

정수 타입 범위(`IntRange`, `LongRange`, `CharRange`)는 특수 기능-이터레이션 가능한-을 제공한다.
컴파일러가 이를 자바의 인덱스 기반 *for*{: .keyword }-루프와 동일하게 바꿔서 추가 오버헤드가 없다.

``` kotlin
for (i in 1..4) print(i) // "1234" 출력

for (i in 4..1) print(i) // 아무것도 출력하지 않음
```

숫자를 역으로 이터레이션하고 싶다면? 간단하다. 표준 라이브러리에 있는 `downTo()` 함수를 사용하면 된다.

``` kotlin
for (i in 4 downTo 1) print(i) // "4321" 출력
```

1씩 증가/감소가 아닌 지정한 단계만큼 숫자를 이터레이션하는 것은? 물론 가능하다. `step()` 함수를 쓰면 된다.

``` kotlin
for (i in 1..4 step 2) print(i) // "13" 출력

for (i in 4 downTo 1 step 2) print(i) // "42" 출력
```


## 동작 방식

범위는 라이브러리의 공통 인터페이스인 `ClosedRange<T>`를 구현한다.

`ClosedRange<T>`는 Comparable 타입에 대한 수학적 의미의 닫힌 구간을 뜻한다.
이는 범위에 포함되는 `start`와 `endInclusive`의 두 끝지점을 갖는다.
주요 오퍼레이션인 `contains'는 보통 *in*{: .keyword }/*!in*{: .keyword } 연산자 형식으로 사용한다.

정수 타입 프로그레션(`IntProgression`, `LongProgression`, `CharProgression`)은 숫자 진행을 뜻한다.
프로그레션은 `first` 요소, `last` 요소, 0이 아닌 `increment`로 정의한다.
첫 번째 요소는 `first`이고, 다음 요소는 이전 요소에 `increment`를 더한 값이다.
`last` 요소는 프로그레이션이 비어(emptty) 있지 않으면 항상 도달한다.

프로그레션은 `Iterable<N>`의 하위 타입으로 `N`에는 `Int`, `Long`, `Char`가 올 수 있으며,
*for*{: .keyword }-루프나 `map`, `filter`와 같은 함수에서 사용할 수 있다.
프로그레션에 대한 이터레이션은 자바/자바스크립에서 다음의 인덱스 기반 *for*{: .keyword }-루프와 동일하다:

``` java
for (int i = first; i != last; i += increment) {
  // ...
}
```

정수 타입에서, `..` 연산자는 `ClosedRange<T>`와 `*Progression`을 모두 구현한 객체를 생성한다.
예를 들어, `IntRange`는 `ClosedRange<Int>`를 구현하고 `IntProgression`를 확장해서, `IntProgression`에 정의된 모든 오퍼레이션을 `IntRange`에서 사용 가능하다.
`downTo()`와 `setp()` 함수 결과는 항상 `*Progression`이다.

프로그레션은 컴페니언 객체에 정의한 `fromClosedRange` 함수로 생성한다:

``` kotlin
  IntProgression.fromClosedRange(start, end, increment)
```

양수 `increment` 기준으로 `end` 값보다 크지 않은 최댓값을 또는 음수 `increment`에 대해 `end` 값보다 작지 않은 최솟값을 찾기 위해 `(last - first) % increment == 0`와 같은 식을 사용해서 프로그레션의 `last` 요소를 계산한다.


## 유틸리티 함수

### `rangeTo()`

정수 타입 `rangeTo()` 연산자는 단순히 `*Range` 클래스의 생성자를 호출한다:

``` kotlin
class Int {
  //...
  operator fun rangeTo(other: Long): LongRange = LongRange(this, other)
  //...
  operator fun rangeTo(other: Int): IntRange = IntRange(this, other)
  //...
}
```

실수형 숫자(`Double`, `Float`)는 `rangeTo` 연산자를 정의하지 않고, 지네릭 `Comparable` 타입을 위해 표준 라이브러리가 제공하는 연산자를 대신 사용한다:

``` kotlin
  public operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

이 함수가 리턴하는 범위는 이터레이션할 수 없다.

### `downTo()`

 `downTo()`는 정수 타입 쌍을 위한 확장 함수이다. 다음은 두 가지 예이다:

``` kotlin
fun Long.downTo(other: Int): LongProgression {
  return LongProgression.fromClosedRange(this, other, -1.0)
}

fun Byte.downTo(other: Int): IntProgression {
  return IntProgression.fromClosedRange(this, other, -1)
}
```

### `reversed()`

`reversed()`는 각 `*Progression` 클래스를 위한 확장 함수이다.
모든 확장 함수는 역순 프로그레션을 리턴한다.

``` kotlin
fun IntProgression.reversed(): IntProgression {
  return IntProgression.fromClosedRange(last, first, -increment)
}
```

### `step()`

`step()`은 `*Progression` 클래스를 위한 확장 함수이다.
모든 확장 함수는 수정한 `step` 값(함수 파라미터)을 가진 프로그레션을 리턴한다.
step 값은 항상 양수여야 한다. 따라서 이 함수는 이터레이션의 방향을 절대 바꾸지 않는다.

``` kotlin
fun IntProgression.step(step: Int): IntProgression {
  if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
  return IntProgression.fromClosedRange(first, last, if (increment > 0) step else -step)
}

fun CharProgression.step(step: Int): CharProgression {
  if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
  return CharProgression.fromClosedRange(first, last, step)
}
```

`(last - first) % increment == 0` 규칙을 유지하기 위해 리턴한 프로그레션의 `last` 값은 원래 프로그레션의 `last`와 다를 수 있다. 다음은 예이다:

``` kotlin
  (1..12 step 2).last == 11  // [1, 3, 5, 7, 9, 11] 값을 가진 프로그레션
  (1..12 step 3).last == 10  // [1, 4, 7, 10] 값을 가진 프로그레션
  (1..12 step 4).last == 9   // [1, 5, 9] 값을 가진 프로그레션
```
