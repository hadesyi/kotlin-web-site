---
type: doc
layout: reference
category: "Syntax"
title: "Null 안전성"
---

# Null 안전성

## Nullable 타입과 non-null 타입

코틀린 타입 시스템은 [The Billion Dollar Mistake](http://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions)라고 불리는 null 참조 위험을 제거하는데 목표가 있다.

자바를 포함한 많은 프로그래밍 언어에서 가장 흔한 위험 중 하나가 null 레퍼런스의 멤버에 접근해서 발생하는 null 참조 익셉션이다.
자바에서는 `NullPointerException` 또는 줄여서 NPE가 이에 해당한다.

코틀린 타입 시스템은 코드에서 `NullPointerException`을 제거하려고 노력했다. 오직 다음 경우만 NPE가 발생한다.

* 직접 `throw NullPointerException()`을 실행
* 아래 설명할 `!!` 연산자 사용
* 외부 자바 코드에서 발생
* 초기화에 관한 데이터 불일치 존재(생성자에서 초기화하지 않은 *this*를 어딘가에서 사용)

코틀린 타입 시스템은 *null*{: .keyword }을 가질 수 있는 레퍼런스(nullable 레퍼런스)와 가질 수 없는 레퍼런스(non-null 레퍼런스)를 구분한다.
예를 들어 일반 `String` 타입 변수는 *null*{: .keyword }을 가질 수 없다.

``` kotlin
var a: String = "abc"
a = null // 컴파일 에러
```

null을 가지려면 `String?`로 쓴 nullable String을 변수로 선언해야 한다.

``` kotlin
var b: String? = "abc"
b = null // ok
```

`a`의 프로퍼티에 접근하거나 메서드를 호출할 때 NPE가 발생하지 않음을 보장할 수 있으며 다음 코드가 안전하다고 할 수 있다.

``` kotlin
val l = a.length
```

하지만 `b`의 프로퍼티에 접근하면 안전하지 않으므로 컴파일러는 에러를 발생한다.

``` kotlin
val l = b.length // 에러: 변수 'b'가 null일 수 있다
```

그래도 b의 프로퍼티에 접근하고 싶다면 몇 가지 할 수 있는 방법이 있다.

## 조건에서 *null*{: .keyword }을 검사하기

첫 번째 방법은 `b`가 *null*{: .keyword }인지 검사하고 두 상황을 각각 처리하는 것이다.

``` kotlin
val l = if (b != null) b.length else -1
```

컴파일러는 검사 결과를 추적하며 *if*{: .keyword } 안에서 `length`를 호출할 수 있도록 한다.
더 복잡한 조건도 지원한다.

``` kotlin
if (b != null && b.length > 0)
  print("String of length ${b.length}")
else
  print("Empty string")
```

이 코드는 오직 `b`가 불변(예, 검사와 사용 사이에 바뀌지 않는 로컬 변수 또는 backing 필드를 갖거나 오버라이딩할 수 없는 *val*{: .keyword } 멤버)인 경우에만 동작한다.
왜냐면 불변이 아니면 검사 이후에 `b`를 *null*{: .keyword }로 바꾸는 일이 벌어질 수 있기 때문이다.

## 안전한 호출

두 번째 방법은 안전 호출 연산자인 `?.`를 사용하는 것이다.

``` kotlin
b?.length
```

`b`가 null이 아니면 `b.length`를 리턴하고 아니면 *null*{: .keyword }을 리턴한다. 이 식의 타입은 `Int?`이다.

안전 호출은 연속할 때 유용하다. 예를 들어, Employee 타입 bob을 Department에 할당하거나 그렇지 않을 수 있고 또 다른 Employee를 Department의 head로 가질 수 있을 때,
Bob의 department head가 존재하면 그 이름을 구하는 코드를 다음과 같이 작성할 수 있다.


``` kotlin
bob?.department?.head?.name
```

프로퍼티 중 하나라도 null이면 이 체인은 *null*{: .keyword }을 리턴한다.

## 엘비스 연산자

nullable 레퍼런스 `r`이 있을 때, "`r`이 not null이면 그것을 사용하고 아니면 non-null 값 `x`를 사용"하는 코드는 다음과 같이 작성한다.

``` kotlin
val l: Int = if (b != null) b.length else -1
```

완전한 *if*{: .keyword }-식 대신 엘비스 연산자인 `?:`를 사용해서 이를 표현할 수 있다.

``` kotlin
val l = b?.length ?: -1
```

`?:`의 왼쪽 식이 null이 아니면 엘비스 연산자는 그것을 리턴하고, 그렇지 않으면 오른쪽 식을 리턴한다.
우측 식은 왼쪽 식이 null인 경우에만 평가한다.

코틀린에서 *throw*{: .keyword }와 *return*{: .keyword }은 식이기 때문에 엘비스 연산자의 우측에 사용할 수 있다.
이는 함수 인자를 검사할 때 매우 유용하게 쓸 수 있다.

``` kotlin
fun foo(node: Node): String? {
  val parent = node.getParent() ?: return null
  val name = node.getName() ?: throw IllegalArgumentException("name expected")
  // ...
}
```

## `!!` 연산자

세 번째 방법은 NPE-추종자를 위한 것이다. `b!!`라고 작성하면 `b`가 non-null이면 값을 리턴하고(이 예에서는 `String`)
`b`가 null이면 NPE를 발생한다.

``` kotlin
val l = b!!.length()
```

따라서 NPE를 원한다면 NPE를 사용할 수 있다. 하지만 NPE를 명시적으로 써야만 한다면 생각하지 못한 곳에서 NPE가 발생하면 안 된다.

## 안전한 변환

일반 변환은 객체가 대상 타입이 아니면 `ClassCastException`을 발생한다.
다른 옵션은 변환에 실패할 때 *null*{: .keyword }을 리턴하는 안전 변환을 사용하는 것이다.

``` kotlin
val aInt: Int? = a as? Int
```
