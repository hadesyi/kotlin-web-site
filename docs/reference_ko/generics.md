---
type: doc
layout: reference
category: "Syntax"
title: "지네릭"
---

# 지네릭

자바처럼 코틀린 클래스도 타입 파라미터를 가질 수 있다:

``` kotlin
class Box<T>(t: T) {
  var value = t
}
```

이 클래스의 인스턴스를 생성하려면 타입 인자를 제공해야 한다:

``` kotlin
val box: Box<Int> = Box<Int>(1)
```

생성자 인자나 다른 방법으로 파라미터를 유추할 수 있으면 타입 인자를 생략할 수 있다:

``` kotlin
val box = Box(1) // 1은 Int 타입을 가지므로 컴파일러는 Box<Int>라고 알아낸다
```

## 가변(Variance)

자바 타입 시스템에서 가장 복잡한 부분 중 하나가 와일드카드 타입이다([자바 지네릭 FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html) 참고).
코틀린은 어떤 것도 갖지 않는다 대신, 선언-위치 가변(declaration-site variance)과 타입 프로젝션(type projection)의 두 가지를 갖는다.

먼저 자바에서 미스테리한 와일드카드가 필요한 이유를 생각해보자. 이 문제를 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html),
Item 28: *Use bounded wildcards to increase API flexibility*에서 설명하고 있다.
첫째, 자바의 지네릭 타입은 **무공변(invariant)**이다. 이는 `List<String>`은 `List<Object>`의 하위타입이 **아님**을 의미한다.
왜 그랬을까? 만약 리스트가 **무공변(invariant)**이 아니면 자바 배열보다 나을 게 없다. 왜냐면 다음 코드가 컴파일은 되지만
런타임에 익셉션이 발생하기 때문이다:

``` java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! 앞에서 언급한 문제 원인이 여기 있다. 자바는 이를 금지한다!
objs.add(1); // 여기서 String의 리스트에 Integer를 넣는다
String s = strs.get(0); // !!! ClassCastException: Integer를 String으로 변환할 수 없다
```
그래서 자바는 런타임 안정성을 보장하기 위해 이런 것을 금지했다. 하지만 이는 몇 가지 영향을 준다. 예를 들어 `Collection` 인터페이스의 `addAll()` 메서드를 생각해보자.
이 메서드의 시그너처는 무엇인가? 직관적으로 다음과 같이 작성할 수 있다:

``` java
// Java
interface Collection<E> ... {
  void addAll(Collection<E> items);
}
```

하지만 (완벽하게 안전한 코드임에도) 다음의 간단한 코드를 만들 수 없다:

``` java
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from); // !!! addAll의 단순한 선언으로는 컴파일되지 않는다:
                   //       Collection<String>은 Collection<Object>의 하위 타입이 아니다
}
```

(우리는 이를 힘들게 배웠다. [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 25:
*Prefer lists to arrays*를 참고하자.)

이런 이유로 실제 `addAll()` 시그너처는 다음과 같다:

``` java
// Java
interface Collection<E> ... {
  void addAll(Collection<? extends E> items);
}
```
**와일드카드 타입 인자** `? extends E`는 이 메서드가 `E` 자체가 아닌 `E`의 *하위타입*의 객체 콜렉션을 허용한다는 것을 말한다.
이는 items에서 안전하게 `E`로 **읽을** 수 있지만(이 콜렉션의 요소는 E의 하위클래스의 인스턴스이다),
`E`의 어떤 하위타입인지 모르기 때문에 items에 **쓸 수 없다**는 것을 의미한다.
이런 제한을 해소하기 위해 `Collection<String>`이 `Collection<? extends Object>`의 하위타입이 되도록 기능을 추가했다.
"전문 용어"로 **extends**\-bound(**upper** bound)를 갖는 와일드카드를 사용해서 타입을 **공변(convariant)**으로 만들었다.

이 트릭이 왜 작동하는지 이해하는데 있어 핵심은 다소 단순하다: 만약 콜렉션에서 아이템을 **가져올** 수만 있다면 `String` 콜렉션에서 `Object`를 읽는 것은 괜찮다.
역으로 콜렉션에 항목을 _넣을_ 수만 있다면 `Object` 콜렉션에 `String`을 넣는 건 괜찮다.
자바에서 `List<? super String>`가 `List<Object>`의 **상위타입**이 된다.

후자를 **반공변(contravariance)**이라 부르며, `List<? super String>`에 인자로 String을 받는 메서드만 호출할 수 있다(예를 들어,
  `add(String)`이나 `set(int, String)`을 호출할 수 있다).
반면에, `List<T>`에서 `T`를 리턴하는 어떤 것을 호출하면 `String`이 아닌 `Object`를 얻게 된다.

Joshua Blochs는 이 객체는 **Producer**에서만 **읽을** 수 있고, **Consumer**로만 **쓸** 수 있다고 했다.
Joshua Blochs는 "*유연함을 최대한 얻으려면 producer나 consumer를 표현하는 입력 파라미터에 와일드카드 타입을 사용하라*"고 권하고 있으며,
다음과 같이 기억을 쉽게 할 수 있는 약자를 제시했다.

*PECS는 Producer-Extends, Consumer-Super를 의미한다.*

*주의*: producer 객체를 사용한다면, 예를 들어 `List<? extends Foo>`, 이 객체에 대해 `add()`나 `set()`을 호출하는 것을 허용하지 않지만,
이것이 이 객체가 **불변(immutable)**인 것을 의미하는 것은 아니다. 예를 들어, 리스트의 모든 항목을 삭제하기 위해 `clear()`를 호출하는 것은 가능하다.
왜냐면, `clear()`는 어떤 파라미터도 갖지 않기 때문이다. 와일드카드(또는 다른 종류의 가변variance)가 보장하는 것은 **타입 안정성**이다. 불변은 완전히 다른 얘기다.

### 선언-위치 가변(Declaration-site variance)

`Source<T>` 지네릭 인터페이스에 `T`를 파라미터로 갖는 메서드는 없고 단지 `T`를 리턴하는 메서드만 있다고 하자:

``` java
// Java
interface Source<T> {
  T nextT();
}
```

이때 `Source<Object>` 타입 변수에 `Source<String>` 인스턴스를 할당하는 것은 완전히 안전한 것이다. 여기엔 어떤 consumer 메서드도 호출하지 않는다.
하지만 자바는 이를 알지 못하기 때문에 이를 금지한다.

``` java
// Java
void demo(Source<String> strs) {
  Source<Object> objects = strs; // !!! 자바는 허용하지 않음
  // ...
}
```

이 문제를 고치려면 `Source<? extends Object>` 타입 객체를 선언해야 하는데, 이는 다소 의미가 없다. 왜냐면, 전과 같이 그런 변수에 동일하게 메서드를 호출할 수 있기 때문에, 더 복잡한 타입에 값을 추가하지 않기 때문이다. 하지만, 컴파일러는 이를 알지 못한다.
To fix this, we have to declare objects of type `Source<? extends Object>`, which is sort of meaningless, because we can call all the same methods on such a variable as before, so there's no value added by the more complex type. But the compiler does not know that.

코틀린은 컴파일러에 이런 류의 내용을 설명하는 방법이 존재한다. 이를 **선언-위치 가변(declaration-site variance)**이라고 부른다. 소스 코드의 **타입 파라미터** `T`에 애노테이션을 붙여서 `Source<T>`의 멤버가 **리턴**(생성)만 하고 소비(consume)하지 않는다고 할 수 있다.
이를 위해 **out** 제한자를 제공한다:

``` kotlin
abstract class Source<out T> {
  abstract fun nextT(): T
}

fun demo(strs: Source<String>) {
  val objects: Source<Any> = strs // T는 out 파라미터이므로 OK
  // ...
}
```

일반 규칙: 클래스 `C`의 타입 파라미터 `T`를 **out**으로 선언하면 타입 파라미터는 오직 `C` 멤버의 **out**\-위치에만 올 수 있다. 하지만 리턴에서 `C<Base>`는 안전하게 `C<Derived>`의 상위타입이 될 수 있다.

"전문 용어"로 클래스 `C`는 파라미터 `T`에 **공변(covariant)** 하다 또는 `T`는 **공변(covariant)** 타입 파라미터라고 말한다.
`C`를 `T`의 **consumer**가 아닌 T`의 **producer**로 생각할 수 있다.

**out** 제한자는 **가변(variance) 애노테이션**이라 부르며, 타입 파라미터 선언 위치에 제공하기 때문에 **선언-위치 가변(declaration-site variance)**에 대한 것이다.
이는 자바가 타입을 사용할 때 와일드카드로 타입을 공변(covariant)하게 만드는 **사용-위치 가변(use-site variance)**인 것과 다르다.

**out**과 더불어 코틀린은 대체 가변(variance) 애노테이션인 **in**을 제공한다. **in**은 타입 파라미터를 **반공변(contravariant)**으로 만들어 준다. 이는 오직 consume만 될 수 있으며
produce 할 수 없다. 반공변(contravariant) 클래스의 좋은 예가 `Comparable`이다:

``` kotlin
abstract class Comparable<in T> {
  abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
  x.compareTo(1.0) // 1.0은 Number의 상위 타입은 Double 타입을 갖는다
  // 그래서, Comparable<Double> 타입 변수를 x에 할당할 수 있다
  val y: Comparable<Double> = x // OK!
}
```

단어 **in**과 **out**이 자명하므로(이미 꽤 오랜 시간 C#에서 성공적으로 사용하고 있다)
위에서 언급한 기억하기 위한 PECS가 실제로 필요 없고 더 상위 목표를 위해 바깔 수 있다고 생각한다:

**[실존주의](http://en.wikipedia.org/wiki/Existentialism) 변환: Consumer in, Producer out\!** :-)

## 타입 프로젝션(Type projections)

### 사용-위치 가변(Use-site variance): 타입 프로젝션

타입 파라미터 T를 *out*으로 선언하는 것은 매우 편리하며, 사용 위치에서 하위타입 관련 문제가 없다.
좋다. 그런데 문제의 클래스를 `T`만 리턴하도록 실제로 제약할 수 있을 때, 그것으로 못하는 건 무얼까?
Array가 좋은 예이다:

``` kotlin
class Array<T>(val size: Int) {
  fun get(index: Int): T { /* ... */ }
  fun set(index: Int, value: T) { /* ... */ }
}
```

이 클래스는 `T`에 대해 공변(covariant)도 반공변(contravariant)도 될 수 없다. 게다가 유연하지 않는 부분을 강제한다. 다음 함수를 보자:

``` kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
  assert(from.size == to.size)
  for (i in from.indices)
    to[i] = from[i]
}
```

이 함수는 한 배열에서 다른 배열로 항목을 복사한다. 실제로 함수 실행을 시도해보자:

``` kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3)
copy(ints, any) // 에러: expects (Array<Any>, Array<Any>)
```

여기서 익숙한 문제가 발생한다. `Array<T>`는 `T`에 대해 **무공변(invariant)**하므로 `Array<Int>`와 `Array<Any>`는 서로 상대방의 하위타입이 아니다.
왜 그럴까? copy는 나쁜 짓을 **할지 모르기** 때문이다. 예를 들어, String을 `from`에 **쓰려고** 시도하는데 실제로 `from`에 `Int` 배열을 전달했다면,
나중에 `ClassCastException`이 발생할 수 있다.

여기서 우리가 원하는 것은 `copy()`가 그런 나쁜 짓을 하지 않는 것을 보장하는 것이다. 우리는 이 메서드가 `from`에 **쓰지** 못하도록 막길 원하며, 다음과 같이 이를 할 수 있다:

``` kotlin
fun copy(from: Array<out Any>, to: Array<Any>) {
 // ...
}
```

여기서 발생한 것을 **타임 프로젝션(type projection)**이라고 한다. 이 코드에서 `from`은 단순 배열이 아닌 **제한된(projected)** 배열이다. 오직 타입 파라미터 `T`를 리턴하는 메서드만 호출할 수 있다.
이 예의 경우 `get()`만 호출할 수 있다. 이것이 코틀린의 **사용-위치 가변(use-site variance)** 접근 방식이다. 자바의 `Array<? extends Object>`에 해당하지만 더 간단한 방법이다.

**in**을 이용해서 타입을 프로젝션할 수 있다:

``` kotlin
fun fill(dest: Array<in String>, value: String) {
  // ...
}
```

`Array<in String>`은 자바의 `Array<? super String>`에 해당하며 `CharSequence`의 배열이나 `Object`의 배열을 `fill()` 함수에 전달할 수 있다.

### 스타-프로젝션

때때로 타입 인자에 대해 알지 못하지만 안전한 방법으로 인자를 사용하고 싶을 때가 있다.
여기서 안전한 방법은 지네릭 타입에 그런 프로젝션을 정의해서, 지네릭 타입의 모든 컨크리트 인스턴스가 그 프로젝션의 하위 타입이 되도록 하는 것이다.

코틀린은 이를 위해 **스타-프로젝션(star-projection)**이라 불리는 구문을 제공한다:

 - `Foo<out T>`에 대해, `T`가 uppber bound `TUpper`를 갖는 공변(covariant) 타입 파라미터라면 `Foo<*>`은 `Foo<out TUpper>`와 같다. 이는 `T`를 몰라도 안전하게 `Foo<*>`에서 `TUpper` 값을 *읽을* 수 있다는 것을 의미한다.
 - `Foo<in T>`에 대해, `T`가 반공변(contravariant) 타입 파라미터라면 `Foo<*>`는 `Foo<in Nothing>`와 같다. 이는 `T`를 모를 때 안전하게 `Foo<*>`에 *쓸 수 없다는* 것을 의미한다.
 - `Foo<T>`에 대해, `T`가 uppber bound `TUpper`를 갖는 무공변(invariant) 타입 파라미터라면, `Foo<*>`는 값을 읽을 때는 `Foo<out TUpper>`와 동일하고 값을 쓸 때는 `Foo<in Nothing>`와 동일하다.

지네릭 타입이 여러 타입 파라미터를 가질 경우 각각 독립적으로 프로젝션할 수 있다.
예를 들어, `interface Function<in T, out U>` 타입을 정의하면, 다음의 스타-프로젝션을 생각할 수 있다:

 - `Function<*, String>`은 `Function<in Nothing, String>`을 의미한다;
 - `Function<Int, *>`은 `Function<Int, out Any?>`를 의미한다;
 - `Function<*, *>`은 `Function<in Nothing, out Any?>`을 의미한다.

*주의*: 스타-프로젝션은 자바의 raw 타입과 매우 유사하지만 안전하다.

# 지네릭 함수

클래스만 타입 파라미터를 가질 수 있는 건 아니다. 함수도 가질 수 있다. 함수 이름 앞에 타입 파라미터를 위치시키면 된다:

``` kotlin
fun <T> singletonList(item: T): List<T> {
  // ...
}

fun <T> T.basicToString() : String {  // 확장 함수
  // ...
}
```

타입 파라미터를 호출 위치(call site)에서 명시적으로 전달하려면 함수 이름 **뒤에** 지정한다:

``` kotlin
val l = singletonList<Int>(1)
```

# 지네릭 제약

주어진 타입 파라미터를 교체하는 모든 가능한 타입은 **지네릭 제약**에 따라 제한된다.

## Upper bounds

가장 일반적인 제약은 자바의 *extends* 키워드에 해당하는 **upper bound**이다:

``` kotlin
fun <T : Comparable<T>> sort(list: List<T>) {
  // ...
}
```

콜론 뒤에 지정한 타입이 **upper bound**이다. `Comparable<T>`의 하위타입만 `T`를 대체할 수 있다. 다음 예를 보자.

``` kotlin
sort(listOf(1, 2, 3)) // OK. Int는 Comparable<Int>의 하위타입이다.
sort(listOf(HashMap<Int, String>())) // 에러: HashMap<Int, String>은 Comparable<HashMap<Int, String>>의 하위타입이 아니다.
```

지정하지 않을 경우 기본 upper bound는 `Any?`이다. 화살괄호 안에 오직 한 개의 upper bound만 지정할 수 있다.
동일 타입 파라미터에 대해 한 개 이상의 upper bound가 필요하면, 별도의 **where**\-절을 사용해야 한다:

``` kotlin
fun <T> cloneWhenGreater(list: List<T>, threshold: T): List<T>
    where T : Comparable,
          T : Cloneable {
  return list.filter { it > threshold }.map { it.clone() }
}
```
