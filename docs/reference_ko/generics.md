---
type: doc
layout: reference
category: "Syntax"
title: "지네릭"
---

# 지네릭

자바처럼, 코틀린 클래스도 타입 파라미터를 가질 수 있다:

``` kotlin
class Box<T>(t: T) {
  var value = t
}
```

보통, 이 클래스의 인스턴스를 생성하려면 타입 인자를 제공해야 한다:

``` kotlin
val box: Box<Int> = Box<Int>(1)
```

생성자 인자나 다른 방법으로 파라미터를 유추할 수 있으면, 타입 인자를 생략할 수 있다:

``` kotlin
val box = Box(1) // 1은 Int 타입을 가지므로, 컴파일러는 Box<Int>라고 알아낸다
```

## Variance

자바 타입 시스템에서 가장 복잡한 부분 중 하나가 와일드카드 타입이다(see [자바 지네릭 FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html) 참고).
코틀린은 어떤 것도 갖지 않는다. 대신, declaration-site variance와 type projections의 두 가지를 갖는다.

먼저, 자바에서 미스테리한 와일드카드가 필요한 이유를 생각해보자. 이 문제를 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 28: *Use bounded wildcards to increase API flexibility*에서 설명하고 있다.
첫 째, 자바의 지네릭 타입은 **invariant**이다. 이는 `List<String>`은 `List<Object>`의 하위타입이 **아님**을 의미한다.
왜 그랬을까? 만약 리스트가 **invariant**가 아니면, 그것은 자바 배열보다 나을 게 없다. 왜냐면, 다음 코드가 컴파일은 되지만
런타임에 익셉션이 발생하기 때문이다:

``` java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! 앞에서 언급한 문제 원인이 여기 있다. 자바는 이를 금지한다!
objs.add(1); // 여기서 String의 리스트에 Integer를 넣는다
String s = strs.get(0); // !!! ClassCastException: Integer를 String으로 변환할 수 없다
```
그래서, 자바는 런타임 안정성을 보장하기 위해 이런 것을 금지했다. 하지만, 이는 몇 가지 영향을 준다. 예를 들어, `Collection` 인터페이스의 `addAll()` 메서드를 생각해보자.
이 메서드의 시그너처는 무엇인가? 직관적으로 다음과 같이 넣을 수 있다:

``` java
// Java
interface Collection<E> ... {
  void addAll(Collection<E> items);
}
```

하지만, (완벽하게 안전한) 다음의 간단한 코드를 할 수 없다:

``` java
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from); // !!! addAll의 단순한 선언으로 컴파일되지 않는다:
                   //       Collection<String>은 Collection<Object>의 하위 타입이 아니다
}
```

(자바에서 우리는 이를 힘들게 배웠다. [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 25:
*Prefer lists to arrays*를 참고한다)

이것이 실제 `addAll()` 시그너처가 다음과 같은 이유이다:

``` java
// Java
interface Collection<E> ... {
  void addAll(Collection<? extends E> items);
}
```
**와일드카드 타입 인자** `? extends T`는 이 메서드가 `T` 자체가 아닌 `T`의 *하위 타입*의 객체 콜렉션을 허용한다는 것을 말한다.
이는 items에서 안전하게 `T`로 **읽을** 수 있지만(이 콜렉션의 요소는 T의 하위클래스의 인스턴스이다),
`T`의 어떤 하위타입인지 모르기 때문에 items에 **쓸 수 없다**는 것을 의미한다.
이런 제한을 해소하기 위해, `Collection<String>`이 `Collection<? extends Object>`의 하위타입이 되도록 기능을 추가했다.
"전문 용어"로, **extends**\-bound를 갖는 워일드카드를 타입 **convariant**로 만들었다.
In "clever words", the wildcard with an **extends**\-bound (**upper** bound) makes the type **covariant**.

The key to understanding why this trick works is rather simple: if you can only **take** items from a collection, then using a collection of `String`s
and reading `Object`s from it is fine. Conversely, if you can only _put_ items into the collection, it's OK to take a collection of
`Object`s and put `String`s into it: in Java we have `List<? super String>` a **supertype** of `List<Object>`.

The latter is called **contravariance**, and you can only call methods that take String as an argument on `List<? super String>`
(e.g., you can call `add(String)` or `set(int, String)`), while
if you call something that returns `T` in `List<T>`, you don't get a `String`, but an `Object`.

Joshua Bloch calls those objects you only **read** from **Producers**, and those you only **write** to **Consumers**. He recommends: "*For maximum flexibility, use wildcard types on input parameters that represent producers or consumers*", and proposes the following mnemonic:

*PECS stands for Producer-Extends, Consumer-Super.*

*NOTE*: if you use a producer-object, say, `List<? extends Foo>`, you are not allowed to call `add()` or `set()` on this object, but this does not mean
that this object is **immutable**: for example, nothing prevents you from calling `clear()` to remove all items from the list, since `clear()`
does not take any parameters at all. The only thing guaranteed by wildcards (or other types of variance) is **type safety**. Immutability is a completely different story.

### Declaration-site variance

Suppose we have a generic interface `Source<T>` that does not have any methods that take `T` as a parameter, only methods that return `T`:

``` java
// Java
interface Source<T> {
  T nextT();
}
```

Then, it would be perfectly safe to store a reference to an instance of `Source<String>` in a variable of type `Source<Object>` -- there are no consumer-methods to call. But Java does not know this, and still prohibits it:

``` java
// Java
void demo(Source<String> strs) {
  Source<Object> objects = strs; // !!! Not allowed in Java
  // ...
}
```

To fix this, we have to declare objects of type `Source<? extends Object>`, which is sort of meaningless, because we can call all the same methods on such a variable as before, so there's no value added by the more complex type. But the compiler does not know that.

In Kotlin, there is a way to explain this sort of thing to the compiler. This is called **declaration-site variance**: we can annotate the **type parameter** `T` of Source to make sure that it is only **returned** (produced) from members of `Source<T>`, and never consumed.
To do this we provide the **out** modifier:

``` kotlin
abstract class Source<out T> {
  abstract fun nextT(): T
}

fun demo(strs: Source<String>) {
  val objects: Source<Any> = strs // This is OK, since T is an out-parameter
  // ...
}
```

The general rule is: when a type parameter `T` of a class `C` is declared **out**, it may occur only in **out**\-position in the members of `C`, but in return `C<Base>` can safely be a supertype
of `C<Derived>`.

In "clever words" they say that the class `C` is **covariant** in the parameter `T`, or that `T` is a **covariant** type parameter.
You can think of `C` as being a **producer** of `T`'s, and NOT a **consumer** of `T`'s.

The **out** modifier is called a **variance annotation**, and  since it is provided at the type parameter declaration site, we talk about **declaration-site variance**.
This is in contrast with Java's **use-site variance** where wildcards in the type usages make the types covariant.

In addition to **out**, Kotlin provides a complementary variance annotation: **in**. It makes a type parameter **contravariant**: it can only be consumed and never
produced. A good example of a contravariant class is `Comparable`:

``` kotlin
abstract class Comparable<in T> {
  abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
  x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
  // Thus, we can assign x to a variable of type Comparable<Double>
  val y: Comparable<Double> = x // OK!
}
```

We believe that the words **in** and **out** are self-explaining (as they were successfully used in C# for quite some time already),
thus the mnemonic mentioned above is not really needed, and one can rephrase it for a higher purpose:

**[The Existential](http://en.wikipedia.org/wiki/Existentialism) Transformation: Consumer in, Producer out\!** :-)

## Type projections

### Use-site variance: Type projections

It is very convenient to declare a type parameter T as *out* and have no trouble with subtyping on the use site. Yes, it is, when the class in question **can** actually be restricted to only return `T`'s, but what if it can't?
A good example of this is Array:

``` kotlin
class Array<T>(val size: Int) {
  fun get(index: Int): T { /* ... */ }
  fun set(index: Int, value: T) { /* ... */ }
}
```

This class cannot be either co\- or contravariant in `T`. And this imposes certain inflexibilities. Consider the following function:

``` kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
  assert(from.size == to.size)
  for (i in from.indices)
    to[i] = from[i]
}
```

This function is supposed to copy items from one array to another. Let's try to apply it in practice:

``` kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3)
copy(ints, any) // Error: expects (Array<Any>, Array<Any>)
```

Here we run into the same familiar problem: `Array<T>` is **invariant** in `T`, thus neither of `Array<Int>` and `Array<Any>`
is a subtype of the other. Why? Again, because copy **might** be doing bad things, i.e. it might attempt to **write**, say, a String to `from`,
and if we actually passed an array of `Int` there, a `ClassCastException` would have been thrown sometime later.

Then, the only thing we want to ensure is that `copy()` does not do any bad things. We want to prohibit it from **writing** to `from`, and we can:

``` kotlin
fun copy(from: Array<out Any>, to: Array<Any>) {
 // ...
}
```

What has happened here is called **type projection**: we said that `from` is not simply an array, but a restricted (**projected**) one: we can only call those methods that return the type parameter
`T`, in this case it means that we can only call `get()`. This is our approach to **use-site variance**, and corresponds to Java's `Array<? extends Object>`,
but in a slightly simpler way.

You can project a type with **in** as well:

``` kotlin
fun fill(dest: Array<in String>, value: String) {
  // ...
}
```

`Array<in String>` corresponds to Java's `Array<? super String>`, i.e. you can pass an array of `CharSequence` or an array of `Object` to the `fill()` function.

### Star-projections

Sometimes you want to say that you know nothing about the type argument, but still want to use it in a safe way.
The safe way here is to define such a projection of the generic type, that every concrete instantiation of that generic type would be a subtype of that projection.

Kotlin provides so called **star-projection** syntax for this:

 - For `Foo<out T>`, where `T` is a covariant type parameter with the upper bound `TUpper`, `Foo<*>` is equivalent to `Foo<out TUpper>`. It means that when the `T` is unknown you can safely *read* values of `TUpper` from `Foo<*>`.
 - For `Foo<in T>`, where `T` is a contravariant type parameter, `Foo<*>` is equivalent to `Foo<in Nothing>`. It means there is nothing you can *write* to `Foo<*>` in a safe way when `T` is unknown.
 - For `Foo<T>`, where `T` is an invariant type parameter with the upper bound `TUpper`, `Foo<*>` is equivalent to `Foo<out TUpper>` for reading values and to `Foo<in Nothing>` for writing values.

If a generic type has several type parameters each of them can be projected independently.
For example, if the type is declared as `interface Function<in T, out U>` we can imagine the following star-projections:

 - `Function<*, String>` means `Function<in Nothing, String>`;
 - `Function<Int, *>` means `Function<Int, out Any?>`;
 - `Function<*, *>` means `Function<in Nothing, out Any?>`.

*Note*: star-projections are very much like Java's raw types, but safe.

# Generic functions

Not only classes can have type parameters. Functions can, too. Type parameters are placed before the name of the function:

``` kotlin
fun <T> singletonList(item: T): List<T> {
  // ...
}

fun <T> T.basicToString() : String {  // extension function
  // ...
}
```

If type parameters are passed explicitly at the call site, they are specified **after** the name of the function:

``` kotlin
val l = singletonList<Int>(1)
```

# Generic constraints

The set of all possible types that can be substituted for a given type parameter may be restricted by **generic constraints**.

## Upper bounds

The most common type of constraint is an **upper bound** that corresponds to Java's *extends* keyword:

``` kotlin
fun <T : Comparable<T>> sort(list: List<T>) {
  // ...
}
```

The type specified after a colon is the **upper bound**: only a subtype of `Comparable<T>` may be substituted for `T`. For example

``` kotlin
sort(listOf(1, 2, 3)) // OK. Int is a subtype of Comparable<Int>
sort(listOf(HashMap<Int, String>())) // Error: HashMap<Int, String> is not a subtype of Comparable<HashMap<Int, String>>
```

The default upper bound (if none specified) is `Any?`. Only one upper bound can be specified inside the angle brackets.
If the same type parameter needs more than one upper bound, we need a separate **where**\-clause:

``` kotlin
fun <T> cloneWhenGreater(list: List<T>, threshold: T): List<T>
    where T : Comparable,
          T : Cloneable {
  return list.filter { it > threshold }.map { it.clone() }
}
```
