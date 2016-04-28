---
type: doc
layout: reference
category: "Interop"
title: "코틀린에서 자바 호출하기"
---

# 코틀린에서 자바 코드 호출하기

코틀린은 자바와의 상호운용성을 염두에 두고 설계했다. 특별한 노력 없이 코틀린에서 기존의 자바 코드를 호출할 수 있고, 또한 자바에서도
비교적 매끄럽게 코틀린 코드를 사용할 수 있다. 이 절에서는 코틀린에서 자바 코드를 호출하는 것에 대한 내용을 자세히 설명한다.

거의 모든 자바 코드를 별 문제없이 사용할 수 있다.

``` kotlin
import java.util.*

fun demo(source: List<Int>) {
  val list = ArrayList<Int>()
  // 자바 콜렉션을 'for'-루프에서 사용:
  for (item in source)
    list.add(item)
  // 연산자 규칙도 작동:
  for (i in 0..source.size() - 1)
    list[i] = source[i] // get과 set 호출
}
```

## Getter와 Setter

getter와 setter를 위한 자바 규칙을 따르는 메서드는(인자가 없고 이름이 `get`으로 시작하는 메서드와 한 개 인자를 갖고 이름이 `set`으로 시작하는 메서드)
코틀린에서 프로퍼티로 표현된다. 다음 예를 보자.

``` kotlin
import java.util.Calendar

fun calendarDemo() {
    val calendar = Calendar.getInstance()
    if (calendar.firstDayOfWeek == Calendar.SUNDAY) {  // getFirstDayOfWeek() 호출
        calendar.firstDayOfWeek = Calendar.MONDAY       // setFirstDayOfWeek() 호출
    }
}
```

자바 클래스가 setter만 가진 경우 코틀린에서 프로퍼티로 보이지 않는다. 왜냐면 코틀린은 아직 쓰기 전용 프로퍼티를 지원하지 않기 때문이다.

## void를 리턴하는 메서드

void를 리턴하는 자바 메서드를 코틀린에서 실행하면 `Unit`을 리턴한다.
그 리턴 값을 사용하려고 하면 그 값을 코틀린 컴파일러는 미리 (`Unit` 임을) 알 수 있으므로
컴파일러가 호출 위치(call site)에 할당한다.

## 코틀린에서 키워드인 자바 식별자를 위한 이스케이프

*in*{: .keyword }, *object*{: .keyword }, *is*{: .keyword } 등 코틀린 키워드 중에서 일부는 자바에서 올바른 식별자이다.
자바 라이브러리가 메서드 이름으로 코틀린 키워드를 사용하면 역따옴표(`) 문자로 메서드 이름을 이스케이프해서 호출할 수 있다.

``` kotlin
foo.`is`(bar)
```

## Null-안전성과 플랫폼 타입

자바에서 모든 레퍼런스는 *null*{: .keyword }일 수 있는데 이는 자바에서 오는 객체는 코틀린의 엄격한 null-안정성을 쓸모없게 만든다.
코틀린은 자바 선언 타입을 *플랫폼 타입*으로 별도 처리한다. 이 타입에 대해서는 null-검사를 완화해서
그 타입에 대한 안전 보장 수준을 자바 정도로 맞춘다([아래](#mapped-types) 참고).

다음 예를 보자.

``` kotlin
val list = ArrayList<String>() // non-null (생성자 결과)
list.add("Item")
val size = list.size() // non-null (int 타입)
val item = list[0] // 플랫폼 타입 유추 (일반 자바 객체)
```

플랫폼 타입 변수의 메서드를 호출하면 코틀린은 컴파일 시점에서 null 가능성 에러를 발생하지 않는다.
하지만 널포인터 익셉션이나 코틀린이 null 전파를 막기 위해 생성하는 assertion 때문에 런타임에 메서드 호출에 실패할 수 있다.

``` kotlin
item.substring(1) // 컴파일은 허용, item == null 이면 익셉션이 발생할 수 있음
```

플랫폼 타입은 *non-denotable*로 이는 언어에서 직접 그 타입을 쓸 수 없음을 의미한다.
플랫폼 값을 코틀린 변수에 할당할 때, 타입 추론에 기대거나(위 예제에서 `item`처럼 변수는 추론한 플랫폼 타입을 갖는다)
원하는 타입을 지정할 수 있다(둘 다 nullable과 non-null 타입을 허용한다):

``` kotlin
val nullable: String? = item // 허용, 항상 동작
val notNull: String = item // 허용, 런타임에 실패할 수 있음
```

non-null 타입을 선택하면 컴파일러가 할당 시점에 assetion을 발생할 수 있다. 이는 코틀린 non-null 변수가 null을 갖는 것을 막아준다.
또한 non-null 값을 기대하는 코틀린 함수에 플랫폼 값을 전달하면 assertion을 발생한다.
이를 종합하면 컴파일러는 null이 프로그램에 전파되는 것을 최대한 막는다(지네릭때문에 완전히 막지는 못한다).

### 플랫폼 타입을 위한 기호

위에서 말한 것처럼 프로그램에서 플랫폼 타입을 직접 지정할 수 없기에 언어에 플랫폼 타입을 위한 구문이 없다.
그럼에도 불구하고 컴파일러와 IDE는 플랫폼 타입을 표현해야 할 필요가 있기에(에러 메시지나 파라미터 정보 등)
다음 기호를 사용한다.

* `T!`는 "`T` 또는 `T?`"를 의미한다,
* `(Mutable)Collection<T>!`는 "`T`의 자바 콜렉션은 불변이거나 아닐 수 있고, nullable이거나 아닐 수 있다"를 의미한다,
* `Array<(out) T>!`는 "`T`(또는 하위 타입)의 배열은 nullable이거나 아니다"를 의미한다,

### Nullability 애노테이션

nullability 애노테이션을 가진 자바 타입은 플랫폼 타입이 아닌 실제 nullable이나 non-null 코틀린 타입으로 표현한다.
현재 컴파일러는 [JetBrains의 nullability 애노테이션](https://www.jetbrains.com/idea/help/nullable-and-notnull-annotations.html)(`org.jetbrains.annotations` 패키지의 `@Nullable`과 `@NotNull`)을 지원한다.

## 매핑한 타입

코틀린은 자바 타입을 특별하게 처리한다. 자바 타입을 *그대로* 로딩하지 않고 해당하는 코틀린 타입으로 매핑한다.
매핑은 컴파일 타임에만 일어나며 런타임 표현은 그대로 유지된다.
자바의 기본 데이터 타입은 해당하는 코틀린 타입으로 매핑된다([플랫폼 타입](#platform-types)을 유념한다).

| **자바 타입** | **코틀린 타입**  |
|---------------|------------------|
| `byte`        | `kotlin.Byte`    |
| `short`       | `kotlin.Short`   |
| `int`         | `kotlin.Int`     |
| `long`        | `kotlin.Long`    |
| `char`        | `kotlin.Char`    |
| `float`       | `kotlin.Float`   |
| `double`      | `kotlin.Double`  |
| `boolean`     | `kotlin.Boolean` |
{:.zebra}

기본 타입이 아닌 일부 내장 클래스도 매핑한다.

| **자바 타입** | **코틀린 타입**  |
|---------------|------------------|
| `java.lang.Object`       | `kotlin.Any!`    |
| `java.lang.Cloneable`    | `kotlin.Cloneable!`    |
| `java.lang.Comparable`   | `kotlin.Comparable!`    |
| `java.lang.Enum`         | `kotlin.Enum!`    |
| `java.lang.Annotation`   | `kotlin.Annotation!`    |
| `java.lang.Deprecated`   | `kotlin.Deprecated!`    |
| `java.lang.Void`         | `kotlin.Nothing!`    |
| `java.lang.CharSequence` | `kotlin.CharSequence!`   |
| `java.lang.String`       | `kotlin.String!`   |
| `java.lang.Number`       | `kotlin.Number!`     |
| `java.lang.Throwable`    | `kotlin.Throwable!`    |
{:.zebra}

코틀린에서 콜렉션 타입은 읽기 전용이거나 변경 가능할 수 있으므로 다음과 같이 자바 콜렉션을 매핑한다(이 표의
모든 코틀린 타입은 `kotlin` 패키지에 위치한다):

| **자바 타입** | **코틀린 읽기 전용 타입**  | **코틀린 수정 가능 타입** | **로딩한 플랫폼 타입** |
|---------------|------------------|----|----|
| `Iterator<T>`        | `Iterator<T>`        | `MutableIterator<T>`            | `(Mutable)Iterator<T>!`            |
| `Iterable<T>`        | `Iterable<T>`        | `MutableIterable<T>`            | `(Mutable)Iterable<T>!`            |
| `Collection<T>`      | `Collection<T>`      | `MutableCollection<T>`          | `(Mutable)Collection<T>!`          |
| `Set<T>`             | `Set<T>`             | `MutableSet<T>`                 | `(Mutable)Set<T>!`                 |
| `List<T>`            | `List<T>`            | `MutableList<T>`                | `(Mutable)List<T>!`                |
| `ListIterator<T>`    | `ListIterator<T>`    | `MutableListIterator<T>`        | `(Mutable)ListIterator<T>!`        |
| `Map<K, V>`          | `Map<K, V>`          | `MutableMap<K, V>`              | `(Mutable)Map<K, V>!`              |
| `Map.Entry<K, V>`    | `Map.Entry<K, V>`    | `MutableMap.MutableEntry<K,V>` | `(Mutable)Map.(Mutable)Entry<K, V>!` |
{:.zebra}

자바 배열은 [아래](java-interop.html#java-arrays) 언급한 것처럼 매핑한다.

| **자바 타입** | **코틀린 타입**  |
|---------------|------------------|
| `int[]`       | `kotlin.IntArray!` |
| `String[]`    | `kotlin.Array<(out) String>!` |
{:.zebra}

## 코틀린에서 자바 지네릭

코틀린의 지네릭은 자바와 약간 다르다([지네릭](generics.html) 참고). 자바 타입을 코틀린에 임포트할 때 일부 변환이 발생한다.

* 자바 와일드 카드를 타입 프로젝션으로 변환
  * `Foo<? extends Bar>`는 `Foo<out Bar!>!`이 된다.
  * `Foo<? super Bar>`는 `Foo<in Bar!>!`이 된다.

* 자바의 raw 타입을 스타-프로젝션으로 변환
  * `List`는 `List<*>!`, 즉 `List<out Any?>!`이 된다.

자바처럼 코틀린도 런타임에 지네릭을 유지하지 않으므로 객체는 생성자에 전달한 실제 타입 파라미터에 대한 정보를 갖지 않는다.
즉 `ArrayList<Integer>()`와 `ArrayList<Character>()`는 구분되지 않는다.
이는 지네릭을 *is*{: .keyword }-검사에 사용할 수 없게 만든다.
코틀린은 스타-프로젝션 지네릭 타입에 대한 *is*{: .keyword }-검사만 허용한다.

``` kotlin
if (a is List<Int>) // 에러: 실제 Int의 List인지 검사할 수 없다
// but
if (a is List<*>) // OK: List의 내용에 대해 보장하지 않는다
```

## 자바 배열

코틀린 배열은 자바와 달리 무공변(invariant)하다. 이는 코틀린에서 `Array<Any>`에 `Array<String>`를 할당할 수 없음을 의미하며, 가능한 런타임 실패를 막아준다.
또한 하위클래스 배열을 코틀린 메서드의 상위클래스 배열 파라미터에 전달하는 것도 막아준다. 자바 메서드는 (`Array<(out) String>!` 형식의 [플랫폼 타입](#platform-types)) 이를 허용한다.

자바 플랫폼에서는 박싱/언박싱 비용을 없애기 위해 배열에 기본 데이터타입을 사용한다.
코틀린은 이런 구현 상세를 감추므로, 자바 코드를 사용하려면 우회방법이 필요하다.
이를 위해 모든 기본 데이터타입의 배열을 위한 별도 클래스(`IntArray`, `DoubleArray`, `CharArray` 등)를 제공한다.
이 클래스는 `Array` 클래스와는 관련이 없으며 최대 성능을 위해 자바의 기본 데이터타입 배열로 컴파일된다.

int 배열을 받는 자바 메서드가 있다고 가정하자:

``` java
public class JavaArrayExample {

    public void removeIndices(int[] indices) {
        // code here...
    }
}
```

코틀린에서 기본 데이터타입의 배열을 전달하려면 다음과 같이 할 수 있다.

``` kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndices(array)  // int[]를 메서드에 전달
```

JVM 바이트 코드로 컴파일할 때 컴파일러는 배열 접근을 최적화해서 추가 오버헤드를 없앤다.

``` kotlin
val array = arrayOf(1, 2, 3, 4)
array[x] = array[x] * 2 // get()과 set()을 호출하지 않음
for (x in array) // Iterator를 생성하지 않음
  print(x)
```

인덱스로 접근할 때도 오버헤드가 발생하지 않는다.

``` kotlin
for (i in array.indices) // Iterator를 생성하지 않음
  array[i] += 2
```

마지막으로 *in*{: .keyword }-검사도 오버헤드가 없다.

``` kotlin
if (i in array.indices) { // (i >= 0 && i < array.size)와 동일
  print(array[i])
}
```

## 자바 가변인자

가변 개수의 인자를 갖는 메서드를 사용하는 자바 클래스가 종종 있다.

``` java
public class JavaArrayExample {

    public void removeIndices(int... indices) {
        // code here...
    }
}
```

이 경우 `IntArray`를 파라미터로 전달하려면 spread 연산자인 `*`를 사용해야 한다.

``` kotlin
val javaObj = JavaArray()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```

현재는 가변 인자 메서드에 *null*{: .keyword }을 전달할 수 없다.

## 연산자

자바는 메서드를 연산자 구문으로 사용할 수 있는 방법이 없기 때문에,
코틀린은 올바른 이름과 시그너처를 가진 모든 자바 메서드를 연산자 오버로딩과 다른 규칙(`invoke()` 등)으로 사용할 수 있도록 한다.
자바 메서드를 중위 호출 구문을 이용해서 호출하는 것을 허용하지 않는다.


## 체크드 익셉션

코틀린의 모든 익셉션은 언체크드인데 이는 컴파일러가 익셉션 catch를 강제하지 않는다는 것을 의미한다.
따라서 체크드 익셉션을 선언한 자바 메서드를 호출할 때 코틀린은 익셉션 cath를 강제하지 않는다.

``` kotlin
fun render(list: List<*>, to: Appendable) {
  for (item in list)
    to.append(item.toString()) // 자바는 여기에 IOException catch를 요구한다
}
```

## Object 메서드

자바 타입을 코틀린에 임포트할 때 `java.lang.Object` 타입의 모든 레퍼런스는 `Any`로 바뀐다.
`Any`는 플랫폼에 특화되어 있지 않기에 멤버로 `toString()`, `hashCode()`, `equals()`만 선언하고 있다.
따라서 `java.lang.Object`의 다른 멤버를 사용할 수 있도록 코틀린은 [확장 함수](extensions.html)를 사용한다.

### wait()/notify()

[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) Item 69는 `wait()`와 `notify()`보다 병렬 유틸리티를 쓰라고 제안하고 있다.
`Any` 타입 레퍼런스에는 이 메서드를 사용할 수 없다.
만약 그 메서드를 실제로 호출해야 한다면 `java.lang.Object`로 변환하면 된다.

```kotlin
(foo as java.lang.Object).wait()
```

### getClass()

객체의 타입 정보를 구하려면 javaClass 확장 프로퍼티를 사용한다.

``` kotlin
val fooClass = foo.javaClass
```

자바의 `Foo.class` 대신에 Foo::class.java를 사용한다.


``` kotlin
val fooClass = Foo::class.java
```

### clone()

`clone()`을 오버라이드하려면, `kotlin.Cloneable`을 확장해야 한다.

```kotlin

class Example : Cloneable {
  override fun clone(): Any { ... }
}
```

[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 11: *Override clone judiciously*를 잊지 말자.

### finalize()

`finalize()`를 오버라이드하려면 *override*{:.keyword} 키워드를 사용하지 않고 단순히 메서드를 선언만 하면 된다.

```kotlin
class C {
  protected fun finalize() {
    // finalization logic
  }
}
```

자바 규칙에 따라 `finalize()`는 *private*{: .keyword }이면 안 된다.

## 자바 클래스를 상속하기

코틀린 클래스의 상위타입으로 최대 한 개의 자바 클래스를(자바 인터페이스는 여러 개) 사용할 수 있다.

## 정적 멤버 접근

자바 클래스의 정적 멤버는 이 클래스의 "컴페니언 오브젝트"를 만든다. 값으로 "컴페니언 오브젝트"를 전달할 수 없지만
멤버에 직접 접근할 수는 있다.

``` kotlin
if (Character.isLetter(a)) {
  // ...
}
```

## 자바 리플렉션

자바 리플렉션은 코틀린 클래스에 동작하며 반대로도 된다. 위에서 말한 것처럼 `java.lang.Class`로 자바 리플렉션을 사용하려면
`instance.javaClass`나 `ClassName::class.java`를 사용할 수 있다.

이 외에 코틀린 프로퍼티를 위한 자바 getter/setter 메서드나 backing 필드 구하기, 자바 필드를 위한 `KProperty`, `KFunction`을 위한 자바 메서드나 생성자 구하기 그리고 그 반대 기능을 지원한다.

## SAM 변환

자바 8과 같이 코틀린은 SAM 변환을 지원한다. 이는 코틀린 함수 리터럴을 (인터페이스 파라미터 타입이 코틀린 함수의 파라미터 타입에 매칭되는 함수)
자동으로 한 개의 non-default 메서드를 갖는 자바 인터페이스 구현으로 변경한다는 것을 의미한다.

SAM 인터페이스의 인스턴스를 생성하러면 SAM 변환을 사용할 수 있다.

``` kotlin
val runnable = Runnable { println("This runs in a runnable") }
```

...메서드 호출에서 SAM 변환:

``` kotlin
val executor = ThreadPoolExecutor()
// Java signature: void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```

자바 클래스가 함수형 인터페이스를 받는 메서드를 여러 개 가지면 람다를 특정 SAM 타입으로 변환하는 어댑터 함수를 호출해서
사용할 메서드를 선택할 수 있다. 컴파일러는 필요한 곳에 이 어댑터 함수를 생성한다.

``` kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```

SAM 변환은 인터페이스에만 적용되며, 추상 클래스의 경우 추상 메서드 한 개만 가진 경우라도 적용되지 않는다.

또한 이 기능은 자바 상호운용에 대해서만 동작한다. 코틀린은 적당한 함수 타입을 갖고 있기 때문에
함수를 코틀린 인터페이스 구현으로 자동 변환하는 기능은 필요 없고 따라서 지원하지 않는다.

## 코틀린에서 JNI 사용하기

네이티브 (C나 C++) 코드로 구현한 함수를 선언하려면 `external` 제한자를 사용하면 된다.

``` kotlin
external fun foo(x: Int): Double
```

이를 뺀 나머지는 자바와 정확하게 같은 방식으로 동작한다.
