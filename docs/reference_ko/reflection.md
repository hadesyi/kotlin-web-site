---
type: doc
layout: reference
category: "Syntax"
title: "리플렉션"
---

# 리플렉션

리플렉션은 런타임에 프로그램의 구조를 인트로스펙션(introspection)할 수 있는 언어와 라이브러리 기능 집합이다.
코틀린 언어에서 함수와 프로퍼티는 일급이고 그것을 인트로스펙션(예, 런타임에 함수나 프로퍼티 타입의 이름을 알아내는 것)하는 것은
단순히 함수형이나 리액티브 방식을 사용하는 것과 밀접하게 관련되어 있다.

> 자바 플랫폼에서, 리플렉션 기능을 사용하는데 필요한 런타임 컴포넌트를 별도 JAR 파일(`kotlin-reflect.jar`)로 배포하고 있다.
이는 리플릭센 기능을 사용하지 않는 어플리케이션이 필요로 하는 런타임 라이브러리의 크기를 줄요한다. 리플렉션을 사용하려면
이 jar 파일을 프로젝트 클래스패스에 넣어야 한다.
{:.note}

## 클래스 레퍼런스

가장 기본적인 리플렉션 기능은 코틀린 클래스에 대한 런타임 레퍼런스를 얻는 것이다. 정적으로 아는 코틀린 클래스의 레퍼런스를 구하러면
_클래스 리터럴_ 구문을 사용하면 된다:

``` kotlin
val c = MyClass::class
```

이 레퍼러스의 값은 [KClass](/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html) 타입이다.

코틀린 클래스 레퍼런스는 자바 클래스 레퍼런스와 다르다. 자바 클래스 레퍼런스를 구하려면
`KClass` 인스턴스의 `.java` 프로퍼티를 사용해야 한다.

## 함수 레퍼런스

다음과 같은 이름을 가진 함수 선언이 있다고 하자:

``` kotlin
fun isOdd(x: Int) = x % 2 != 0
```

쉽게 함수를 직접 호출할 수 있다(`isOdd(5)`). 또한, 함수를 다른 함수에 값으로 전달할 수도 있다.
이를 하려면 `::` 연산자를 사용한다:

``` kotlin
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // [1, 3] 출력
```

여기서 `::isOdd`는 함수 타입 `(Int) -> Boolean`의 값이다.

오버로딩한 함수에 대해서는 `::` 연산자를 사용할 수 없다. 향후에 오버로딩한 함수 중에서 선택할 수 있도록 파라미터 타입을
지정하는 구문을 제공할 계획이다.

클래스의 멤버나 확장 함수를 사용해야 한다면 클래스 이름을 명시한다. 예를 들어,
`String::toCharArray`는 `String`을 위한 확장 함수인 `String.() -> CharArray`를 제공한다.

### 예제: 함수 조합

다음 함수를 보자:

``` kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}
```

이는 전달한 두 함수를 조합해서 리턴한다(`compose(f, g) = f(g(*))`).
이제 이 함수에 호출할 수 있는 레퍼런스를 적용할 수 있다:


``` kotlin
fun length(s: String) = s.size

val oddLength = compose(::isOdd, ::length)
val strings = listOf("a", "ab", "abc")

println(strings.filter(oddLength)) // "[a, abc]" 출력
```

## 프로퍼티 레퍼런스

코틀린에서 일급 객체인 프로퍼티에 접근할 때에도 `::` 연산자를 사용한다:

``` kotlin
var x = 1

fun main(args: Array<String>) {
    println(::x.get()) // "1" 출력
    ::x.set(2)
    println(x)         // "2" 출력
}
```

`::x` 식은 `KProperty<Int>` 타입의 프로퍼티 객체를 구한다. 이 타입을 이용하면
`get()`을 사용해서 값을 읽거나 `name` 프로퍼티를 이용해서 프로퍼티 이름을 구할 수 있다.
더 자세한 정보는 [`KProperty` 클래스 문서](/api/latest/jvm/stdlib/kotlin.reflect/-k-property/index.html)를 참고한다.

`var y = 1`과 같은 수정 가능 프로퍼티의 경우 `::y`는 [`KMutableProperty<Int>`](/api/latest/jvm/stdlib/kotlin.reflect/-k-mutable-property/index.html) 타입 값을 리턴한다.
이 타입은 `set()` 메서드를 갖고 있다.

파라미터가 없는 함수가 필요한 곳에 프로퍼티 레퍼런스를 사용할 수 있다:

``` kotlin
val strs = listOf("a", "bc", "def")
println(strs.map(String::length)) // [1, 2, 3] 출력
```

클래스의 멤버인 프로퍼티에 접근할 때에는 클래스를 한정한다:

``` kotlin
class A(val p: Int)

fun main(args: Array<String>) {
    val prop = A::p
    println(prop.get(A(1))) // prints "1"
}
```

확장 프로퍼티의 경우:


``` kotlin
val String.lastChar: Char
  get() = this[size - 1]

fun main(args: Array<String>) {
  println(String::lastChar.get("abc")) // prints "c"
}
```

### 자바 리플렉션과의 상호 운용성

자바 플랫폼에서, 표준 라이브러리는 리플렉션 클래스를 위해 자바 리플렉션 객체와의 매핑을 제공하는 확장을 포함하고 있다(`kotlin.reflect.jvm` 패키지 참고).
예를 들어 backing 필드나 코틀린 프로퍼티의 getter를 위한 자바 메서드를 찾고 싶다면 다음과 같은 코드를 사용할 수 있다:


``` kotlin
import kotlin.reflect.jvm.*

class A(val p: Int)

fun main(args: Array<String>) {
    println(A::p.javaGetter) // "public final int A.getP()" 출력
    println(A::p.javaField)  // "private final int A.p" 출력
}
```

자바 클래스에 해당하는 코틀린 클래스를 구하려면 확장 프로퍼티로 `.kotlin`을 사용한다:

``` kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```

## 생성자 레퍼런스

메서드나 프로퍼티처럼 생성자 레퍼런스를 구할 수 있다. 생성자와 같은 파라미터를 갖고 관련 타입 객체를 리턴하는
함수 타입이 필요한 곳에 생성자 레퍼런스를 사용할 수 있다.
다음 함수는 `::` 연산자와 클래스 이름을 사용해서 생성자 레퍼런스를 구한다. 이 함수는 파라미터를 갖지 않고 리턴 타입이 `Foo`인 함수를 파라미터로 사용한다:

``` kotlin
class Foo

fun function(factory : () -> Foo) {
    val x : Foo = factory()
}
```

`::Foo`를 사용하면, 즉 Foo 클래스의 인자 없는 생성자 레퍼런스로, 다음처럼 간단히 생성자를 호출할 수 있다:

``` kotlin
function(::Foo)
```
