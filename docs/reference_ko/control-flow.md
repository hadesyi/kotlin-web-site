---
type: doc
layout: reference
category: "Syntax"
title: "흐름 제어"
---

# 흐름 제어

## If 식

코틀린에서 *if*{: .keyword }는 식이며 값을 리턴한다.
삼항 연산자(condition ? then : else)는 없는데, 이유는 *if*{: .keyword }로 충분히 할 수 있기 때문이다.

``` kotlin
// 구식 용법
var max = a
if (a < b)
  max = b

// else와 함께 사용
var max: Int
if (a > b)
  max = a
else
  max = b

// 식으로 사용
val max = if (a > b) a else b
```

*if*{: .keyword } 브랜치는 블록이 될 수 있고, 마지막 식이 블록의 값이 된다:

``` kotlin
val max = if (a > b) {
    print("Choose a")
    a
  }
  else {
    print("Choose b")
    b
  }
```

*if*{: .keyword }를 문장이 아닌 식으로 사용할 경우, `else` 브랜치를 가지려면 식이 필요하다.

[*if*{: .keyword } 문법](grammar.html#if) 참고.

## When 식

*when*{: .keyword }은 C와 유사한 언어의 switch 연산자를 대체한다. 가장 단순한 형태는 다음과 같다.

``` kotlin
when (x) {
  1 -> print("x == 1")
  2 -> print("x == 2")
  else -> { // 블록 가능
    print("x is neither 1 nor 2")
  }
}
```

*when*{: .keyword }은 충족하는 브랜치 조건이 나올때까지 모든 브랜치를 순차적으로 찾는다.
*when*{: .keyword }은 식이나 문장으로 사용할 수 있다. 만약 식으로 사용하면, 충족한 브랜치의 값이 전체 식의 값이 된다.
문장으로 사용하면, 개별 브랜치의 값은 무시한다. (*if*{: .keyword }처럼 각 브랜치는 블록일 수 있으며, 블록의 마지막 식이 값이 된다.)
*else*{: .keyword } 브랜치는 모든 브랜치 조건이 맞지 않을 때 사용한다.
*when*{: .keyword }을 식으로 사용할 경우, 각 브랜치 조건이 모든 경우의 수를 다뤘다고 컴파일러에서 판명할 수 없으면,
*else*{: .keyword }를 필수로 사용해야 한다.

여러 경우를 동일하게 처리해야 할 경우, 브랜치 조건을 콤마로 묶을 수 있다:

``` kotlin
when (x) {
  0, 1 -> print("x == 0 or x == 1")
  else -> print("otherwise")
}
```

브랜치 조건으로 (상수뿐만 아니라) 임의 식을 사용할 수 있다.

``` kotlin
when (x) {
  parseInt(s) -> print("s encodes x")
  else -> print("s does not encode x")
}
```

[range](ranges.html)나 콜렉션에 대해 *in*{: .keyword }이나 *!in*{: .keyword }을 사용해서 값을 검사할 수 있다:

``` kotlin
when (x) {
  in 1..10 -> print("x is in the range")
  in validNumbers -> print("x is valid")
  !in 10..20 -> print("x is outside the range")
  else -> print("none of the above")
}
```

*is*{: .keyword }나 *!is*{: .keyword }로 특정 타입의 값인지 검사할 수 있다. [스마트 변환](typecasts.html#smart-casts) 덕에
추가 검사 없이 타입의 프로퍼티와 메서드에 접근할 수 있다.

```kotlin
val hasPrefix = when(x) {
  is String -> x.startsWith("prefix")
  else -> false
}
```

*if*{: .keyword }-*else*{: .keyword } *if*{: .keyword } 체인 대신 *when*{: .keyword }을 사용할 수 있다.
인자를 제공하지 않으면, 브랜치 조건은 단순 부울 식이며, 각 조건이 true일 때 브랜치를 실행한다:

``` kotlin
when {
  x.isOdd() -> print("x is odd")
  x.isEven() -> print("x is even")
  else -> print("x is funny")
}
```

[*when*{: .keyword } 문법](grammar.html#when) 참고.


## For 루트

*for*{: .keyword } 루트는 이터레이터를 제공하는 모든 것에 대해 반복을 수행한다. 구문은 다음과 같다:

``` kotlin
for (item in collection)
  print(item)
```

몸체는 블록일 수 있다.

``` kotlin
for (item: Int in ints) {
  // ...
}
```

앞서 언급한 것처럼, *for*{: .keyword }는 다음에 맞는 이터레이터(iterator)를 제공하는 모든 것을 반복한다.

* `iterator()` 멤버 함수나 확장 함수를 갖고, 이 함수의 리턴 타입이
  * `next()` 멤버 함수나 확장 함수를 갖고,
  * `Boolean`을 리턴하는 `hasNext()` 함수나 확장 함수를 갖는다.

이 세 함수는 `operator`로 표시해야 한다.

배열에 대한 `for` 루프는 이터레이터 객체를 생성하지 않는 인덱스 기반 루프로 컴파일된다.

인덱스를 이용해서 배열이나 리스트를 반복하고 싶다면, 다음 방법을 사용하면 된다:

``` kotlin
for (i in array.indices)
  print(array[i])
```

이 "범위를 통한 반복"는 추가 객체 생성 없는 최적화된 코드로 컴파일된다.

대신, `withIndex` 라이브러리 함수로 처리할 수도 있다:

``` kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

[*for*{: .keyword } 문법](grammar.html#for) 참고.

## While 루프

*while*{: .keyword }과 *do*{: .keyword }..*while*{: .keyword }은 예상하는대로 동작한다.

``` kotlin
while (x > 0) {
  x--
}

do {
  val y = retrieveData()
} while (y != null) // 여기서 y에 접근 가능!
```

[*while*{: .keyword } 문법](grammar.html#while) 참고.

## 루프에서 break와 continue

코틀린은 루프에서 전통적인 *break*{: .keyword }와 *continue*{: .keyword }를 지원한다. [리턴과 점프](returns.html)를 참고한다.
