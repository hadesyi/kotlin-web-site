---
type: doc
layout: reference
category: "Syntax"
title: "애노테이션"
---

# 애노테이션

## 애노테이션 선언
애노테이션은 코드에 메타데이터를 추가하는 방법이다. 애노테이션을 선언하려면 *annotation*{: .keyword } 제한자를 클래스 앞에 넣으면 된다.

``` kotlin
annotation class Fancy
```

애노테이션 클래스에 메타 애노테이션을 붙여서 애노테이션의 속성을 지정할 수 있다.

  * [`@Target`](/api/latest/jvm/stdlib/kotlin.annotation/-target/index.html)은 애노테이션을 할 수 있는 요소 종류를 지정한다(클래스, 함수, 프로퍼티, 식 등);
  * [`@Retention`](/api/latest/jvm/stdlib/kotlin.annotation/-retention/index.html)은 애노테이션을 컴파일한 클래스에 보관할지 여부와 런타임에 리플렉션을 통해 접근할 수 있는지 여부를 지정한다(기본은 둘 다 true);
  * [`@Repeatable`](/api/latest/jvm/stdlib/kotlin.annotation/-repeatable/index.html)은 같은 애노테이션을 한 요소에 여러 번 적용할 수 있는지 여부를 지정한다;
  * [`@MustBeDocumented`](/api/latest/jvm/stdlib/kotlin.annotation/-must-be-documented/index.html)은 애노테이션이 공개 API에 속하는지 여부와 생성한 API 문서의 클래스나 메서드 시그너처에 포함해야 하는지 여부를 지정한다.

``` kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
public annotation class Fancy
```

### 용법

``` kotlin
@Fancy class Foo {
  @Fancy fun baz(@Fancy foo: Int): Int {
    return (@Fancy 1)
  }
}
```

클래스의 주요 생성자에 애노테이션을 하고 싶으면 *constructor*{: .keyword} 키워드를 생성자 선언에 추가하고 그 키워드 앞에 애노테이션을 넣으면 된다.


``` kotlin
class Foo @Inject constructor(dependency: MyDependency) {
  // ...
}
```

프로퍼티 accessor에도 애노테이션 할 수 있다.

``` kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```

### 생성자

애노테이션은 파라미터가 있는 생성자를 가질 수 있다.

``` kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```

허용하는 파라미터 타입은 다음과 같다.

 * 자바의 기본 타입에 해당하는 타입(Int, Long 등)
 * 문자열
 * 클래스(`Foo::class`)
 * 열거형
 * 다른 애노테이션
 * 위에 나열한 타입의 배열

애노테이션을 다른 애노테이션의 파라미터로 사용하면 @ 문자를 이름 앞에 붙이지 않는다.

``` kotlin
public annotation class ReplaceWith(val expression: String)

public annotation class Deprecated(
        val message: String,
        val replaceWith: ReplaceWith = ReplaceWith(""))

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```

### 람다

람다에서도 애노테이션을 사용할 수 있다. 람다의 몸체를 생성할 때 `invoke()` 메서드에 애노테이션을 적용한다.
동시성 제어를 위해 애노테이션을 사용하는 [Quasar](http://www.paralleluniverse.co/quasar/)와 같은 프레임워크에서 이를 유용하게 쓰고 있다.

``` kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```

## 사용-위치(Use-site) 대상 애노테이션

주요 생성자 파라미터나 프로퍼티는 여러 자바 요소를 생성할 수 있다.
따라서 이 코틀린 요소에 애노테이션을 달면 생성한 자바 바이트코드의 여러 위치에 애노테이션이 붙을 수 있다.
정확하게 애노테이션을 어느 자바 요소에 생성할지 지정하려면 다음 구문을 사용한다.

``` kotlin
class Example(@field:Ann val foo,    // 자바 필드
              @get:Ann val bar,      // 자바 getter
              @param:Ann val quux)   // 자바 생성자 파라미터
```

같은 구문을 전체 파일을 애노테이션 할 때에 사용할 수 있다. 이를 하기 위해 패키지 디렉티브 전에 또는 기본 패키지면 모든 임포트 전에
파일 최상단에 대상이 `file`인 애노테이션을 넣는다.

``` kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```

여러 애노테이션을 같은 대상에 적용하고 싶다면 대상 뒤에 대괄호 안에 모든 애노테이션을 넣어서 대상을 반복하는 것을 피할 수 있다.

``` kotlin
class Example {
     @set:[Inject VisibleForTesting]
     public var collaborator: Collaborator
}
```

지원하는 전체 사용 위치(use-site) 대상은 다음과 같다.

  * `file`
  * `property` (이 대상을 갖는 애노테이션은 자바에는 보이지 않는다)
  * `field`
  * `get` (프로퍼티 getter)
  * `set` (프로퍼티 setter)
  * `receiver` (확장 함수나 프로퍼티의 리시버 파라미터)
  * `param` (생성자 파라미터)
  * `setparam` (프로퍼티 setter 파라미터)
  * `delegate` (위임 프로퍼티를 위한 위임 인스턴스를 보관한 필드)

확장 함수의 리시버 파라미터를 애노테이션 하려면 다음 구문을 사용한다.

``` kotlin
fun @receiver:Fancy String.myExtension() { }
```

사용 위치(use-site) 대상을 지정하지 않으면 사용할 애노테이션의 `@Target` 애노테이션에 따라 대상을 선택한다.
만약 여러 대상이 적용 가능하면 다음 목록에서 먼저 적용할 수 있는 대상을 선택한다.

  * `param`
  * `property`
  * `field`


## 자바 애노테이션

자바 애노테이션은 코틀린과 100% 호환된다.

``` kotlin
import org.junit.Test
import org.junit.Assert.*

class Tests {
  @Test fun simple() {
    assertEquals(42, getTheAnswer())
  }
}
```

자바로 작성한 애노테이션은 파라미터 순서가 없기 때문에 인자를 전달할 때 일반 함수 호출 구문을 사용할 수 없다.
대신 이름 인자 구문을 사용해야 한다.

``` java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```

``` kotlin
// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```

자바와 마찬가지로 `value` 파라미터는 특수 케이스다. 이름 없이 이 파라미터 값을 지정할 수 있다.

``` java
// Java
public @interface AnnWithValue {
    String value();
}
```

``` kotlin
// Kotlin
@AnnWithValue("abc") class C
```

자바에서 `value` 인지가 배열 타입이면 코틀린의 `vararg` 파라미터가 된다.

``` java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```

``` kotlin
// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") class C
```

애노테이션 인자로 클래스를 지정하고 싶다면 코틀린 클래스
([KClass](/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html))를 사용한다.
코틀린 컴파일러는 KClass를 자동으로 자바 클래스로 변환하므로 자바 코드는 애노테이션과 인자를 볼 수 있다.

``` kotlin

import kotlin.reflect.KClass

annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any?>)

@Ann(String::class, Int::class) class MyClass
```

애노테이션 인스턴스의 값은 코틀린 코드에서 프로퍼티로 노출된다.

``` java
// Java
public @interface Ann {
    int value();
}
```

``` kotlin
// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```
