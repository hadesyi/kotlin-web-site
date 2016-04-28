---
type: doc
layout: reference
category: "Interop"
title: "자바에서 코틀린 실행하기"
---

# 자바에서 코틀린 실행하기

자바에서 쉽게 코틀린 코드를 실행할 수 있다.

## 프로퍼티

프로퍼티 getter는 *get*-메서드로 setter는 *set*-메서드로 바뀐다.

## 패키지 수준 함수

`example.kt` 파일의 `org.foo.bar` 패키지에 선언한 함수와 프로퍼티는
`org.foo.bar.ExampleKt`라 불리는 자바 클래스에 위치한다.

``` kotlin
// example.kt
package demo

class Foo

fun bar() {
}

```

``` java
// Java
new demo.Foo();
demo.ExampleKt.bar();
```

`@JvmName` 애노테이션을 사용하면 생성할 자바 클래스의 이름을 바꿀 수 있다:

``` kotlin
@file:JvmName("DemoUtils")

package demo

class Foo

fun bar() {
}

```

``` java
// Java
new demo.Foo();
demo.DemoUtils.bar();
```

(같은 패키지나 같은 이름 또는 같은 @JvmName 애노테이션을 사용해서) 생성할 자바 클래스 이름이 같은 파일이 여러 개인 경우 보통 에러가 발생한다.
하지만, 컴파일러는 같은 이름을(파일 이름이나 `@JvmName` 값) 갖는 모든 파일에 선언한 모든 것을 포함하는 단일 자바 파사드 클래스를 만드는 기능을 제공한다.
파사드 생성을 활성화하려면 모든 파일에 @JvmMultifileClass 애노테이션을 사용하면 된다.

``` kotlin
// oldutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun foo() {
}
```

``` kotlin
// newutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun bar() {
}
```

``` java
// Java
demo.Utils.foo();
demo.Utils.bar();
```

## 인스턴스 필드

코틀린 프로퍼티를 자바의 필드로 노출하고 싶다면 `@JvmField` 애노테이션을 프로퍼티에 적용해야 한다.
필드는 해당하는 프로퍼티와 같은 가시성을 갖는다. 프로퍼티가 backing 필드를 갖거나, 가시성이 private이 아니거나,
`open`, `override` 또는 `const` 제한자를 갖지 않거나, 위임 프로퍼티가 아니면
프로퍼티에 `@JvmField`를 붙일 수 없다.

``` kotlin
class C(id: String) {
    @JvmField val ID = id
}
```

``` java
// Java
class JavaClient {
    public String getID(C c) {
        return c.ID;
    }
}
```

[Late-Initialized](properties.html#late-initialized-properties) 프로퍼티도 필드로 노출된다.
필드는 `lateinit` 프로퍼티의 setter와 같은 가시성을 갖는다.

## 정적 필드

네임드 오브젝트나 컴페니언 오브젝트에 선언한 코틀린 프로퍼티는 네임드 오브젝트나 컴페니언 오브젝트를 포함하는 클래스에서
정적 backing 필드를 갖는다.

보통 이 필드는 private이지만 다음 중 한 가지 방법으로 노출할 수 있다:

 - `@JvmField` 애노테이션;
 - `lateinit` 제한자;
 - `const` 제한자.

`@JvmField`을 붙인 프로퍼티는 프로퍼티 자체와 같은 가시성을 갖는 정적 필드가 된다.

``` kotlin
class Key(val value: Int) {
    companion object {
        @JvmField
        val COMPARATOR: Comparator<Key> = compareBy<Key> { it.value }
    }
}
```

``` java
// Java
Key.COMPARATOR.compare(key1, key2);
// Key 클래스에 public static final 필드
```

오브젝트나 컴페니언 오브젝트의 [late-initialized](properties.html#late-initialized-properties) 프로퍼티는
프로퍼티 setter와 같은 가시성을 갖는 정적 backing 필드를 갖는다.

``` kotlin
object Singleton {
    lateinit var provider: Provider
}
```

``` java
// Java
Singleton.provider = new Provider();
// public static non-final field in Singleton class
```

(최상위 수준 또는 클래스에 있는) `const` 프로퍼티는 자바에서 정적 필드가 된다:

``` kotlin
// file example.kt

object Obj {
  const val CONST = 1
}

class C {
    companion object {
        const val VERSION = 9
    }
}

const val MAX = 239
```

자바:

``` java
int c = Obj.CONST;
int d = ExampleKt.MAX;
int v = C.VERSION;
```

## 정적 메서드

앞서 언급했듯이 패키지 수준 함수에 대해 코틀린은 정적 메서드를 생성한다.
또한 코틀린은 네임드 오브젝트나 컴페니언 오브젝트에 정의한 함수에 `@JvmStatic`을 붙이면 정적 메서드를 생성한다.
다음 예를 보자:

``` kotlin
class C {
  companion object {
    @JvmStatic fun foo() {}
    fun bar() {}
  }
}
```

여기서 `foo()`는 자바에서 정적이지만 `bar()`는 아니다:

``` java
C.foo(); // 잘 동작
C.bar(); // 에러: 정적 메서드 아님
```

네임드 오브젝트도 같다:

``` kotlin
object Obj {
    @JvmStatic fun foo() {}
    fun bar() {}
}
```

자바:

``` java
Obj.foo(); // 잘 동작
Obj.bar(); // 에러
Obj.INSTANCE.bar(); // 싱글톤 인스턴스를 통해서 호출
Obj.INSTANCE.foo(); // 역시 동작
```

`@JvmStatic` 애노테이션을 오브젝트 프로퍼티나 컴페니언 오브젝트에 적용할 수 있다.
이는 getter와 setter 메서드를 그 오브젝트나 컴페니언 오브젝트를 포함한 클래스의 정적 메서드로 만든다.


## @JvmName으로 시그너처 충돌 처리하기

때때로 코틀린에서 네임드 함수가 JVM 바이트 코드 상의 이름과 달라야 할 때가 있다.
가장 두드러진 예는 *타입 제거(type erasure)* 때문에 발생한다:

``` kotlin
fun List<String>.filterValid(): List<String>
fun List<Int>.filterValid(): List<Int>
```

이 두 함수는 함께 정의할 수 없다. 왜냐면 JVM 시그너처가 `filterValid(Ljava/util/List;)Ljava/util/List;`로 같기 때문이다.
코틀린에서 같은 이름을 갖는 함수를 정의하고 싶다면 두 함수 중 하나에 (또는 두 함수 모두에) `@JvmName`의 인자로 다른 이름을 지정해야 한다:

``` kotlin
fun List<String>.filterValid(): List<String>

@JvmName("filterValidInt")
fun List<Int>.filterValid(): List<Int>
```

코틀린에서는 같은 이름인 `filterValid`로 접근할 수 있지만 자바에서는 이름이 `filterValid`와 `filterValidInt`가 된다.

함수 `getX()`를 갖는 이름이 `x`인 프로퍼티에 대해서도 같은 트릭을 사용할 수 있다:

``` kotlin
val x: Int
  @JvmName("getX_prop")
  get() = 15

fun getX() = 10
```


## 오버로딩 생성

보통 기본 파라미터 값을 갖는 코틀린 메서드를 작성하면 자바에서는 모든 파라미터가 있는 전체 시그너처만 사용 가능하다.
만약 자바 코드에 오버로드한 여러 메서드를 노출하고 싶다면 @JvmOverloads 애노테이션을 사용할 수 있다.

``` kotlin
@JvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") {
    ...
}
```

기본 값을 가진 모든 파라미터에 대해 오버로드 메서드를 생성한다. 기본 값을 가진 파라미터와 그 오른쪽에 위치한 파라미터를 파라미터 목록에서 제거한다.
이 예는 다음 메서드를 생성한다:

``` java
// 자바
void f(String a, int b, String c) { }
void f(String a, int b) { }
void f(String a) { }
```

생성자와 정적 메서드에도 이 애노테이션을 적용할 수 있다. 인터페이스에 정의한 메서드를 포함해서 추상 메서드에는 사용할 수 없다.

[보조 생성자](classes.html#secondary-constructors)에서 설명한 것처럼, 클래스가 모든 생성자 파라미터에 대해 기본 값을 가지면
인자 없는 public 생성자를 생성한다.
이는 @JvmOverloads 애노테이션을 지정하지 않아도 적용된다.


## 체크드 익셉션

코틀린에는 체크드 익셉션이 없다.
따라서 보통은 코틀린 함수의 자바 시그너처는 익셉션을 선언하지 않는다.
다음과 같은 코틀린 함수를 가졌다고 해보자:

``` kotlin
// example.kt
package demo

fun foo() {
  throw IOException()
}
```

그리고 자바에서 위 함수를 호출할 때 익셉션을 catch하고 싶다고 하자:

``` java
// 자바
try {
  demo.Example.foo();
}
catch (IOException e) { // 에러: foo()는 throws 목록에 IOException을 선언하지 않았다
  // ...
}
```

`foo()`가 `IOException`을 선언하고 있지 않으므로 자바 컴파일러는 에러를 발생한다.
이 문제를 피하려면 코틀린 코드에 `@Throws` 애노테이션을 사용하면 된다.

``` kotlin
@Throws(IOException::class)
fun foo() {
    throw IOException()
}
```

## Null-안전성

자바에서 코틀린 함수를 호출할 때 non-null 파라미터에 *null*{: .keyword }을 전달하는 것을 막을 수가 없다.
이런 이유로 코틀린은 non-null을 요구하는 모든 public 함수에 대해 런타임 검사를 추가한다.
이것이 자바 코드에서 즉시 `NullPointerException`을 얻는 방법이다.

## 기변(Variant) 지네릭

코틀린이 [선언-위치 가변](generics.html#declaration-site-variance)을 사용하면,
자바 코드에서 이를 사용하는 두 가지 선택이 있다. 다음 클래스와 이 클래스를 사용하는 두 함수를 보자:

``` kotlin
class Box<out T>(val value: T)

interface Base
class Derived : Base

fun boxDerived(value: Derived): Box<Derived> = Box(value)
fun unboxBase(box: Box<Base>): Base = box.value
```

이 함수를 자바로 변환하는 가장 단순한 방법은 다음과 같다:

``` java
Box<Derived> boxDerived(Derived value) { ... }
Base unboxBase(Box<Base> box) { ... }
```

문제는 코틀린에서는 `unboxBase(boxDerived("s"))`가 가능하지만 자바에서는 가능하지 않다는 것이다.
왜냐면 자바에서 `Box` 클래스는 `T` 파라미터에 대해 *무공변(invariant)*이라서 `Box<Derived>`가 `Box<Base>`의 하위타입이 아니기 때문이다.
자바에서 이게 동작하도록 하려면 `unboxBase`를 다음과 같이 정의해야 한다:

``` java
Base unboxBase(Box<? extends Base> box) { ... }  
```  

여기서 사용-위치 가변(use-site variance)을 이용해서 선언-위치 가변(declaration-site variance)을 흉내내기 위해
자바의 *와일드카드 타입*(`? extends Base`)을 사용했다. 왜냐면 자바로는 이렇게밖에 못하기 때문이다.

코틀린 API를 자바에서 쓸 수 있게 하기 위해 파라미터로 쓰이는 `Box<Super>`를 (공변으로 정의한 Box인) `Box<? extends Super>`로 생성한다(또는 반공변으로 정의한 Foo인 `Foo<? super Bar>`로 생성).
그것이 리턴 값이면 와일드카드를 생성하지 않는다. 그렇지 않을 경우 자바 클라이언트가 그것을 처리해야 하기 때문이다(그리고 이는 보통의 자바 코딩 방식과 반대된다).
따라서 위 예의 함수는 실제로 다음과 같이 번역된다:

``` java
// 리턴 타입 - 와일드카드 없음
Box<Derived> boxDerived(Derived value) { ... }

// 파라미터 - 와일드카드
Base unboxBase(Box<? extends Base> box) { ... }
```

노트: 인지 타입이 final이면, 보통 와일드카드를 생성해도 얻는게 없기 때문에 `Box<String>`은 인자 위치에 상관없이 항상 `Box<String>`이다.

기본적으로 와일드카드를 생성할 수 없는 곳에서 와일드카드가 필요하면 `@JvmWildcard` 애노테이션을 사용할 수 있다:

``` kotlin
fun boxDerived(value: Derived): Box<@JvmWildcard Derived> = Box(value)
// 다음으로 번역
// Box<? extends Derived> boxDerived(Derived value) { ... }
```

반면에 와일드카드 생성이 필요 없으면 `@JvmSuppressWildcards`를 사용한다:

``` kotlin
fun unboxBase(box: Box<@JvmSuppressWildcards Base>): Base = box.value
// 다음으로 번역
// Base unboxBase(Box<Base> box) { ... }
```

노트: `@JvmSuppressWildcards`을 개별 타입 인자에만 사용할 수 있는 것은 아니다. 함수나 클래스처럼 전체 선언에도 사용할 수 있다. 이는 선언 안의 모든 와일드카드를 제한한다.

### Nothing 타입의 번역

`Nothing` 타입은 특별하다. 왜냐면 자바에 이와 딱들어맞는 요소가 없기 때문이다. 사실 `java.lang.Void`를 포함한 모든 자바 레퍼런스 타입은
값으로 `null`을 가질 수 있는데 `Nothing`은 그것조차 안 된다. 따라서 자바에서 이 타입을 완벽하게 표현할 수는 없다.
이것이 코틀린이 `Nothing` 타입을 사용하는 인자에 raw 타입을 생성하는 이유이다:

``` kotlin
fun emptyList(): List<Nothing> = listOf()
// is translated to
// List emptyList() { ... }
```
