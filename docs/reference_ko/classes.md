---
type: doc
layout: reference
category: "Syntax"
title: "클래스와 상속"
related:
    - functions.md
    - nested-classes.md
    - interfaces.md
---

# 클래스와 상속

## 클래스

코틀린은 *class*{: .keyword } 키워드를 사용해서 클래스를 선언한다:

``` kotlin
class Invoice {
}
```

클래스 선언은 클래스 이름, 클래스 헤더(타입 파라미터 지정, 주요 생성자 등)와 중괄호로 둘러 싼 클래스 몸체로 구성된다.
헤더와 몸체는 필수가 아니다. 클래스가 몸체를 갖지 않으면 중괄호를 생략할 수 있다.

``` kotlin
class Empty
```


### 생성자

코틀린의 클래스는 한 개의 **주요 생성자**와 한 개 이상의 **보조secondary 생성자**를 가질 수 있다.
주요 생성자는 클래스 헤더에 속한다. 클래스 헤더는 클래스 이름(그리고 선택에 따라 타입 파라미터) 뒤에 위치한다.

``` kotlin
class Person constructor(firstName: String) {
}
```

주요 생성자가 애노테이션이나 가시성 제한자를 갖지 않으면, *constructor*{: .keyword } 키워드를 생략할 수 있다:

``` kotlin
class Person(firstName: String) {
}
```

주요 생성자는 코드를 포함할 수 없다. 초기화 코드는 *init*{: .keyword } 키워드를 이용한
**초기화 블록**에 위치할 수 있다.

``` kotlin
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```

주요 생성자의 파라미터는 초기화 블록과 클래스 몸체에 선언한 프로퍼티 초기화(initializer)에서 사용할 수 있다.

``` kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```

주요 생성자에서 프로퍼티를 선언하고 초기화할 수 있도록 코틀린은 간결한 구문을 제공한다:


``` kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
  // ...
}
```

일반 프로퍼티와 동일하게, 주요 생성자에 선언한 프로퍼티도 변경 가능(*var*{: .keyword })이거나
읽기 전용(*val*{: .keyword })이 될 수 있다.

생성자가 애노테이션이나 가시성 제한자를 가질 경우, *constructor*{: .keyword } 키워드가 필요하며,
키워드 전에 제한자가 위치하게 된다:

``` kotlin
class Customer public @Inject constructor(name: String) { ... }
```

자세한 내용은 [가시성 제한자](visibility-modifiers.html#constructors)를 참고한다.


#### 보조 생성자(Secondary Constructors)

클래스는 *constructor*{: .keyword }를 이용해서 **보조 생성자**를 선언할 수 있다.

``` kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

클래스가 주요 생성자를 가지면, 각 보조 생성자는 직접 주요 생성자에 위임하거나 다른 보조 생성자를 통해 간접적으로 주요 생성자에
위임해야 한다. *this*{: .keyword } 키워드를 이용해서 같은 클래스의 다른 생성자에 위임할 수 있다:

``` kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

추상이 아닌 클래스가 어떤 생성자도 선언하지 않으면, 인자를 갖지 않은 주요 생성자를 만든다.
클래스가 공개(public) 생성자를 갖는 것을 원치 않으면, 기본 가시성이 아닌 빈(empty) 주요 생성자를 추가해야 한다:

``` kotlin
class DontCreateMe private constructor () {
}
```

> **주의**: JVM에서, 주요 생성자의 모든 파라미터가 기본 값을 가지면, 컴파일러는 기본 값을 사용하는 파라미터 없는 생성자를 추가로 만든다.
> 이는 Jackson이나 JPA처럼 파라미터없는 생성자를 이용해서 클래스 인스턴스를 생성하는 라이브러리에 대해 코틀린을 사용하기 쉽게 만들어준다.
>
> ``` kotlin
> class Customer(val customerName: String = "")
> ```
{:.info}

### 클래스의 인스턴스 생성하기

클래스의 인스턴스를 생성하려면, 일반 함수와 비슷하게 생성자를 호출한다:

``` kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

코틀린은 *new*{: .keyword }가 없음에 유의한다.


### 클래스 멤버

클래스는 다음을 갖는다.

* 생성자와 초기화 블록
* [함수](functions.html)
* [프로퍼티](properties.html)
* [중첩 클래스와 내부 클래스](nested-classes.html)
* [오브젝트 선언](object-declarations.html)


## 상속

코틀린의 모든 클래스는 공통의 상위클래스(superclass)인 `Any`를 갖는다. `Any`는 상위타입(supertype)을 지정하지 않은 클래스의 기본 상위이다:

``` kotlin
class Example // 암무적으로 Any를 상속한다
```

`Any`는 `java.lang.Object`가 아니다; 특히, `equals()`, `hashCode()` 그리고 `toString()`외에 다른 멤버를 갖지 않는다.
보다 자세한 내용은 [자바 호환](java-interop.html#object-methods) 절을 참고한다.

상위타입을 직접 선언하려면, 클래스 헤더에 콜론 뒤에 타입을 위치시킨다:

``` kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

클래스가 주요 생성자를 가지면, 주요 생성자의 파라미터를 이용해서 베이스 타입을 바로 초기화한다.(그리고 초기화해야 한다.)

클래스가 주요 생성자를 갖지 않으면, 각 보조 생성자는 *super*{: .keyword } 키워드를 사용해서 베이스 타입을 초기화하거나
그것을 하는 다른 생성자에 위임해야 한다.
이 경우 보조 생성자는 베이스 타입의 다른 생성자를 호출할 수 있다:

``` kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx) {
    }

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs) {
    }
}
```

클래스의 *open*{: .keyword } 애노테이션은 자바의 *final*{: .keyword }과 정반대이다. *open*{: .keyword }은 다른 클래스가
이 클래스를 상속할 수 있도록 한다. 기본적으로 코틀린에서 모든 클래스는 final이다. 이는
[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html)의
Item 17: *Design and document for inheritance or else prohibit it* 내용을 따른 것이다.

### 멤버 오버라이딩

앞서 언급한 것처럼, 콘틀린에서는 명시적으로 지정해야 한다. 자바와 달리, 코틀린은 오버라이딩 가능한 멤버(*open*이라고 부름)와
오버라이드를 위해 명시적으로 애노테이션을 붙여야 한다:

``` kotlin
open class Base {
  open fun v() {}
  fun nv() {}
}
class Derived() : Base() {
  override fun v() {}
}
```

`Derived.v()`는 *override*{: .keyword } 애노테이션이 필요하다. 붙이지 않으면 컴파일 에러가 발생한다.
`Base.nv()`와 같이 함수에 *open*{: .keyword }을 붙이지 않으면, *override*{: .keyword }의 여부에 상관없이 하위클래스에서 동일 시그너처럴 갖는 메서드를 선언할 수 없다.
(*open*{: .keyword } 애노테이션이 없는) final 클래스에는 open 멤버가 금지된다.

*override*{: .keyword }를 갖는 멤버는 가 자체로 open이며, 하위클래스에서 오버라이딩할 수 있다. 오버라이딩을 막고 싶다면 *final*{: .keyword }을 사용하면 된다:

``` kotlin
open class AnotherDerived() : Base() {
  final override fun v() {}
}
```

#### Wait! How will I hack my libraries now?!

클래스와 멤버가 기본적으로 final인 오버라이딩에 대한 접근 방식은 한 가지 이슈가 있다. 그것은 바로 라이브러리 설계자가
오버라이딩하도록 의도하지 않은 어떤 메서드를 오버라이딩하거나 어떤 공격적 수정을 추가하기 위해 사용한 라이브러리에서 상속을 사용하기 어렵다는 점이다.
One issue with our approach to overriding (classes and members final by default) is that it would be difficult to subclass something inside the libraries you use to override some method that was not intended for overriding by the library designer, and introduce some nasty hack there.

우리는 다음과 같은 이유로 이는 단점이 아니라 생각한다:

* 어쨋든 이런 류의 hack은 허용하지 않는게 Best practice다
* 유사한 방식을 갖는 다른 언어(C++, C#)을 성공적으로 사용하고 있다
* 실제로 hack을 원한다면, 여전히 방법은 있다. 자바로 해킹 코드를 작성해서 코틀린에서 호출할 수 있고 (*[자바 상호운용](java-interop.html) 참고*), Aspect 프레임워크로 해킹할 수 있다.

### 오버라이딩 규칙

코틀린에서 구현 상속은 다음 규칙을 따른다: 클래스가 바로 상위의 여러 상위클래스에서 같은 멤버 구현을 상속하면,
반드시 이 멤버를 오버라이딩하고 자신의 구현(아마도, 상속받은 것 중의 하나를 사용)을 제공해야 한다.
사용할 상위타입의 구현을 지정하려면, 화살괄호에 상위타입 이름을 지정한 *super*{: .keyword }를 사용한다. 예, `super<Base>`:

``` kotlin
open class A {
  open fun f() { print("A") }
  fun a() { print("a") }
}

interface B {
  fun f() { print("B") } // 인터페이스의 멤버는 기본이 'open'이다
  fun b() { print("b") }
}

class C() : A(), B {
  // 컴파일하려면 f()를 오버라이딩해야 한다:
  override fun f() {
    super<A>.f() // call to A.f()
    super<B>.f() // call to B.f()
  }
}
```

`A`와 `B`를 상속받는 것은 괜찮고, `a()`와 `b()`에는 문제가 없다. 왜냐면 `C`는 이 두 함수에 대해 각각 한 개의 구현만 상속받기 때문이다.
하지만, `f()`의 경우 `C`가 두 개의 구현을 상속받기 때문에, `C`에 `f()`를 오버라이딩해서 모호함을 제거하는 구현을 제공해야 한다.

## 추상 클래스

클래스와 멤버를 *abstract*{: .keyword }로 선언할 수 있다.
추상 멤버는 구현을 갖지 않는다.
추상 클래스나 함수는 open을 붙일 필요가 없다.

추상이 아닌 open 멤버를 추상 멤버로 오버라이딩할 수 있다.

``` kotlin
open class Base {
  open fun f() {}
}

abstract class Derived : Base() {
  override abstract fun f()
}
```

## 컴페니언 오브젝트(Companion Objects)

자바나 C#과 달리 코틀린의 클래스는 정적 메서드가 없다. 대신, 많은 경우 패키지 수준의 함수를 사용할 것을 추천한다.

만약 클래스 인스턴스없이 클래스의 내부에 접근해야하는 함수를 작성하고 싶다면(예, 팩토리 메서드),
그 클래스 안에 [오브젝트 선언](object-declarations.html)의 멤버로 함수를 작성할 수 있다.

좀 더 구체적으로, 클래스 안에 [컴페니언 오브젝트](object-declarations.html#companion-objects)를 선언하면
클래스 이름만으로 자바/C#의 정적 메서드를 호출하는 것처럼 컴페니언 오브젝트의 멤버를 호출할 수 있다.


## 실드 클래스(Sealed Classes)

값이 제한된 타입 집합 중 하나만 가질 수 있고 다른 타입을 가질 수 없도록 하고 싶을 때,
클래스 상속을 제한할 목적으로 실드 클래스를 사용한다. 실드 클래스는 어떤 의미에서 열거형 클래스의 확장이다.
열거 타입은 제한된 값 집합을 가지지만, 각 열거형 상수는 오직 한 개 인스턴스만 존재한다.
반면에 실드 클래스의 하위클래스는 상태를 포함할 수 있는 여러 인스턴스를 가질 수 있다.

실드 클래스를 선언하려면, 클래스 이름 앞에 `sealed` 제한자를 쓰면 된다. 실드 클래스는 하위 클래스를 가질 수 있지만,
모든 하위 클래스는 실드 클래스 선언 자체에 중첩해야 한다.

``` kotlin
sealed class Expr {
    class Const(val number: Double) : Expr()
    class Sum(val e1: Expr, val e2: Expr) : Expr()
    object NotANumber : Expr()
}
```

실드 클래스의 하위 클래스를 확장하는 클래스(indirect inheritors)는 어디든 위치할 수 있다. 실드 클래스 선언 내부에 위치할 필요는 없다.

실드 클래스의 최대 장점은 [`when` 식](control-flow.html#when-expression)과 함께 사용할 수 있다는 점이다.
when 문이 모든 경우를 다루는지 확인할 수 있기 때문에, `else` 절을 추가할 필요가 없다.

``` kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Expr.Const -> expr.number
    is Expr.Sum -> eval(expr.e1) + eval(expr.e2)
    Expr.NotANumber -> Double.NaN
    // 모든 경우를 다루기 때문에 `else` 절이 필요없다
}
```
