---
type: doc
layout: reference
category: "Syntax"
title: "확장"
---

# 확장

C#이나 Gosu와 비슷하게 코틀린은 클래스를 확장하거나 데코레이터와 같은 디자인 패턴을 사용하지 않고 클래스에 새 기능을 확장하는 기능을 제공한다.
_확장(extension)_이라 불리는 특수한 선언을 이용해서 확장할 수 있다. 코틀린은 _확장 함수_와 _확장 프로퍼티_를 지원한다.

## 확장 함수

확장 함수를 선언하려면 _리시버 타입(receiver type)_(확장할 타입)의 이름을 접두어로 사용해야 한다.
다음은 `MutableList<Int>`에 `swap` 함수를 추가한다:

``` kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
  val tmp = this[index1] // 'this'는 리스트에 해당한다
  this[index1] = this[index2]
  this[index2] = tmp
}
```

확장 함수에서 *this*{: .keyword } 키워드는 리시버 객체(점 기호 앞에 전달되는 객체)에 해당한다.
이제 `MutableList<Int>`에 대해 다음과 같이 함수를 호출할 수 있다:

``` kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // 'swap()'에서 'this'는 'l'의 값을 갖는다.
```

물론 이 함수는 모든 `MutableList<T>`에 의미가 있으므로 지네릭으로 만들 수 있다:

``` kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
  val tmp = this[index1] // 'this'는 리스트에 해당한다
  this[index1] = this[index2]
  this[index2] = tmp
}
```

함수 이름 앞에 리시버 타입 식에서 사용가능한 타입 파라미터를 선언한다.
[지네릭 함수](generics.html)를 참고한다.

## **정적**인 확장 결정

확장은 실제로 확장할 클래스를 수정하지 않는다. 확장을 정의하면 클래스에 새 멤버를 추가하는 것이 아니라
단지 그 클래스의 인스턴스에 대해 점-기호로 호출할 수 있는 새로운 함수를 만드는 것뿐이다.

실행할 확장 함수는 **정적으로** 결정한다! 예를 들어 리시버 타입에 따라 버추얼하게 결정하지 않는다.
확장 함수를 호출하는 코드의 타입으로 호출할 확장 함수를 결정한다. 런타임에 그 식을 평가한 결과 타입으로 결정하지 않는다.
다음 예를 보자:

``` kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```

이 예는 "C"를 출력한다. 왜냐면, 호출할 확장 함수를 선택할 때 `c` 파라미터의 선언 타입인 `C` 클래스만 사용하기 때문이다.

만약 클래스가 멤버 함수를 갖고, 동일 리시버 타입을 갖는 동일 이름의 확장 함수가 있고, 주어진 인자를 적용할 수 있다면, **항상 멤버가 이긴다**.
다음 예를 보자:

``` kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }
```

타입이 `C`인 `c`에 에 대해 `c.foo()`를 호출하면, "extension"이 아닌 "member"를 출력한다..

## Nullable 리서버

nullable 리시버 타입을 정의할 수도 있다. 이 확장은 비록 객체 변수가 null이어도 호출되며, 확장 함수 몸체에서 `this == null`로 검사할 수 있다.
이것이 코틀린에서 null 검사 없이 toString()을 호출할 수 있는 이유이다. 확장 함수 안에서 null 여부를 검사한다.

``` kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // 검사 이후에, 'this'를 non-null 타입으로 자동 변환하므로,
    // 다음의 toString()은 Any 클래스의 멤버 함수를 사용한다.
    return toString()
}
```

## 확장 프로퍼티

함수와 유사하게, 코틀린은 확장 프로퍼티를 지원한다:

``` kotlin
val <T> List<T>.lastIndex: Int
  get() = size - 1
```

확장은 실제로 클래스에 멤버를 추가하지 않으므로, 확장 프로퍼티가 [backing 필드](properties.html#backing-fields)를 가질 방법은 없다.
이것이 **확장 프로퍼티에 대해 initializer를 허용하지 않는** 이유이다. 이는 명시적으로 제공하는 getter/setter로만 가능하다.

예제:

``` kotlin
val Foo.bar = 1 // 에러: 확장 프로퍼티에 대한 initializer 허용하지 않음
```


## 컴페니언 오브젝트 확장

클래스가 [컴페니언 오브젝트](object-declarations.html#companion-objects)를 가지면, 컴페니언 오브젝트에 대해서 함수와 프로퍼티를
확장할 수 있다:

``` kotlin
class MyClass {
  companion object { }  // "Companion"으로 불림
}

fun MyClass.Companion.foo() {
  // ...
}
```

컴페니언 오브젝트의 다른 멤버처럼, 클래스 이름을 사용해야 확장을 호출할 수 있다:

``` kotlin
MyClass.foo()
```


## 확장의 범위

대부분 패키지 하위의 최상위 레벨에 직접 확장을 정의한다.

``` kotlin
package foo.bar

fun Baz.goo() { ... }
```

패키지 밖에서 이런 확장을 사용하려면, 사용측에서 확장을 임포트해야 한다:

``` kotlin
package com.example.usage

import foo.bar.goo // "goo"의 모든 확장을 임포트
                   // 또는
import foo.bar.*   // "foo.bar"로부터 모두 임포트

fun usage(baz: Baz) {
  baz.goo()
)

```

더 많은 정보는 [임포트](packages.html#imports)를 참고한다.

## 멤버로 확장을 선언하기

클래스 안에서, 다른 클래스를 위한 확장을 선언할 수 있다. 그 확장 안에는 여러 _암묵적(implicit) 리서버_-한정자(qualifier) 없이 접근할 수 있는 오브젝트 멤버-가 존재한다.
확장을 선언한 클래스의 인스턴스를 _디스패치(dispatch) 리시버_라 부르고, 확장 메서드의 리시버 타입 인스턴스를 _확장 리시버_라고 부른다.

``` kotlin
class D {
    fun bar() { ... }
}

class C {
    fun baz() { ... }

    fun D.foo() {
        bar()   // D.bar 호출
        baz()   // C.baz 호출
    }

    fun caller(d: D) {
        d.foo()   // 확장 함수 호출
    }
}
```

디스패치 리시버와 확장 리시버의 멤버 이름이 충돌할 경우, 확장 리시버가 우선한다. 디스패치 리시버의 멤버를 참조하려면,
[한정한 `this` 구문](this-expressions.html#qualified)을 사용하면 된다.

``` kotlin
class C {
    fun D.foo() {
        toString()         // D.toString() 호출
        this@C.toString()  // C.toString() 호출
    }
```

멤버로 선언한 확장을 `open`으로 설정하면 하위클래스에서 오버라이딩할 수 있다. 이는 확장 함수 선택이 디스패치 리시버 타입에 따라
결정됨을 의미한다. 하지만, 확장 리시버 타입은 정적이다.

``` kotlin
open class D {
}

class D1 : D() {
}

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }

    fun caller(d: D) {
        d.foo()   // 확장 함수 호출
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}

C().caller(D())   // "D.foo in C" 출력
C1().caller(D())  // "D.foo in C1" 출력 - 디스패치 리시버를 버추얼하게 선택
C().caller(D1())  // "D.foo in C" - 확장 리시버를 정적으로 선택
```


## 동기

자바에서는 `FileUtils`, `StringUtils`처럼 "\*Utils"라는 이름을 갖는 클래스에 익숙하다. 잘 알려진 `java.util.Collections`도 이에 속한다.
이런 유틸 클래스가 싫은 부분은 코드가 다음과 같은 모습을 띄기 때문이다:

``` java
// Java
Collections.swap(list, Collections.binarySearch(list, Collections.max(otherList)), Collections.max(list))
```

클래스 이름이 항상 방해가 된다. 정적 임포트를 사용하면 다음 코드가 된다:

``` java
// Java
swap(list, binarySearch(list, max(otherList)), max(list))
```

조금 나아졌지만, IDE의 강력한 코드 완성 기능의 도움을 거의 받지 못한다. 다음 코드처럼 할 수 있다면 훨씬 나을 것이다.

``` java
// Java
list.swap(list.binarySearch(otherList.max()), list.max())
```

하지만 `List` 클래스에 모든 가능한 메서드를 구현하는 것을 원치 않는다, 그렇지 않나? 이것이 확장이 우리를 돕는 지점이다.
