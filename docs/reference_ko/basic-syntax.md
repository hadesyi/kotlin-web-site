---
type: doc
layout: reference
category: "Basics"
title: "기본 문법Basic Syntax"
---

# 기본 문법

## 패키지 정의

소스 파일 최상단에 패키지를 지정해야 한다:

``` kotlin
package my.demo

import java.util.*

// ...
```

디렉토리와 패키지가 일치할 필요는 없다. 소스 파일은 아무 디렉토리에나 위치할 수 있다.

[패키지](packages.html) 참고.

## 함수 정의

두 개의 `Int` 파라미터와 `Int` 리턴 타입을 갖는 함수:

``` kotlin
fun sum(a: Int, b: Int): Int {
  return a + b
}
```

표현식 몸체를 갖는 리턴 타입 추론 함수:

``` kotlin
fun sum(a: Int, b: Int) = a + b
```

의미있는 값을 리턴하지 않는 함수:

``` kotlin
fun printSum(a: Int, b: Int): Unit {
  print(a + b)
}
```

`Unit` 리턴 타입은 생략할 수 있음:

``` kotlin
fun printSum(a: Int, b: Int) {
  print(a + b)
}
```

[함수](functions.html) 참고.

## 로컬 변수 정의

한 번 할당Assign-once(읽기 전용) 로컬 변수:

``` kotlin
val a: Int = 1
val b = 1   // `Int` 타입 추론
val c: Int  // 값을 할당하지 않을 경우 타입 필요
c = 1       // 확정(definite) 할당
```

변경 가능 변수:

``` kotlin
var x = 5 // `Int` 타입 추론
x += 1
```

[프로퍼티와 필드](properties.html) 참고.


## Comments

자바와 자바스크립트처럼, 코틀린도 라인(end-of-line) 주석과 블록 주석을 지원한다.

``` kotlin
// 이것은 라인(end-of-line) 주석

/* 이 코드는 여러 줄에 걸치는
   블록 주석입니다. */
```

자바와 달리, 코틀린은 블록 주석을 중첩할 수 있다.

문서화를 위한 주석 문법에 대한 내용은 [코틀린 코드 문서화](kotlin-doc.html) 참고.

## Using string templates

``` kotlin
fun main(args: Array<String>) {
  if (args.size == 0) return

  print("First argument: ${args[0]}")
}
```

[문자열 템플릿](basic-types.html#string-templates) 참고.

## Using conditional expressions

``` kotlin
fun max(a: Int, b: Int): Int {
  if (a > b)
    return a
  else
    return b
}
```

표현식에 *if*{: .keyword } 사용하기:

``` kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

[*if*{: .keyword }-식](control-flow.html#if-expression) 참고.

## nullable 값 사용과 *null*{: .keyword } 검사

레퍼런스가 *null*{: .keyword } 값을 가질 수 있으면 반드시 nullable하다고 표시해야 한다.

`str`이 정수를 포함하지 않을 때 *null*{: .keyword }을 리턴하는 코드 예:

``` kotlin
fun parseInt(str: String): Int? {
  // ...
}
```

 nullable 값을 리턴하는 함수 사용하기:

``` kotlin
fun main(args: Array<String>) {
  if (args.size < 2) {
    print("Two integers expected")
    return
  }

  val x = parseInt(args[0])
  val y = parseInt(args[1])

  // 두 값이 null일 수 있으므로 `x * y`는 에러를 발생할 수 있다.
  if (x != null && y != null) {
    // null 검사 후에 x와 y를 non-nullable로 자동으로 캐스팅
    print(x * y)
  }
}
```

또는

``` kotlin
  // ...
  if (x == null) {
    print("Wrong number format in '${args[0]}'")
    return
  }
  if (y == null) {
    print("Wrong number format in '${args[1]}'")
    return
  }

  // null 검사 후에 x와 y를 non-nullable로 자동 변환
  print(x * y)
```

[null 안전성 Null-safety](null-safety.html) 참고.

## 타입 체크와 자동 변환 사용하기

*is*{: .keyword } 연산자는 표현식이 지정한 타입의 인스턴스인지 검사한다.
불변(immutable) 로컬 변수나 프로퍼티에 대해 특정 타입인지 검사하면, 명시적으로 타입 변환할 필요가 없다:

``` kotlin
fun getStringLength(obj: Any): Int? {
  if (obj is String) {
    // 이 코드 브랜치(branch)에서 `obj`는 자동으로 `String`으로 변환
    return obj.length
  }

  // 타입 검사 브랜치 밖에서 `obj`는 여전히 `Any` 타입
  return null
}
```

또는

``` kotlin
fun getStringLength(obj: Any): Int? {
  if (obj !is String)
    return null

  // `obj`는 이 브랜치에서 자동으로 `String`으로 변환
  return obj.length
}
```

또는 심지어

``` kotlin
fun getStringLength(obj: Any): Int? {
  // `&&`의 우측에서 자동으로 `String`으로 변환
  if (obj is String && obj.length > 0)
    return obj.length

  return null
}
```

[클래스](classes.html)와 [타입 변환](typecasts.html) 참고.

## `for` 루프 사용하기

``` kotlin
fun main(args: Array<String>) {
  for (arg in args)
    print(arg)
}
```

또는

``` kotlin
for (i in args.indices)
  print(args[i])
```

[for 루프](control-flow.html#for-loops) 참고.

## `while` 루프 사용하기

``` kotlin
fun main(args: Array<String>) {
  var i = 0
  while (i < args.size)
    print(args[i++])
}
```

[while 루프](control-flow.html#while-loops) 참고.

## `when` 표현식 사용하기

``` kotlin
fun cases(obj: Any) {
  when (obj) {
    1          -> print("One")
    "Hello"    -> print("Greeting")
    is Long    -> print("Long")
    !is String -> print("Not a string")
    else       -> print("Unknown")
  }
}
```

[when 표현식](control-flow.html#when-expression) 참고.

## 범위(range) 사용하기

*in*{: .keyword } 연산자를 사용해서 숫자가 특정 범위 안에 있는지 검사:

``` kotlin
if (x in 1..y-1)
  print("OK")
```

숫자가 범위 밖인지 검사:

``` kotlin
if (x !in 0..array.lastIndex)
  print("Out")
```

범위에 속한 숫자를 이터페이션:

``` kotlin
for (x in 1..5)
  print(x)
```

[범위](ranges.html) 참고.

## 콜렉션 사용하기

콜렉션 이터레이션:

``` kotlin
for (name in names)
  println(name)
```

*in*{: .keyword } 연산자를 사용해서 콜렉션이 객체를 포함하고 있는지 검사하기:

``` kotlin
if (text in names) // names.contains(text) is called
  print("Yes")
```

콜렉션을 필터링하고 변환(맵)할 때 람다식 사용하기:

``` kotlin
names
    .filter { it.startsWith("A") }
    .sortedBy { it }
    .map { it.toUpperCase() }
    .forEach { print(it) }
```

[고차함수와 람다](lambdas.html) 참고.
