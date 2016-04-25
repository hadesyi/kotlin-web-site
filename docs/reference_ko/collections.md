---
type: doc
layout: reference
category: Other
title: "콜렉션"
---

# 콜렉션

많은 다른 언어와 달리 코틀린은 (리스트, 집합, 맵 등 콜렉션에 대해) 변경가능 콜렉션과 변경불가 컬렉션을을 구분한다. 언제 콜렉션을 수정할 수 있는 정확하게 제어하는 것은 버그를 제거하고 좋은 API를 설계하는데 도움이 된다.

변경 가능 콜렉션의 읽기 전용 _뷰_와 실제 변경 불가 콜렉션의 차이점을 처음부터 이해하는 것이 중요하다. 둘 다 만들기 쉽지만, 타입 시스템은 차이를 표현하지 않으므로, (차이가 중요하다면) 콜렉션이 둘 중 무엇인지 추적하는 것은 당신 몫이다.

코틀린의 `List<out T>` 타입은 `size`나 `get`과 같은 읽기 전용 연산을 제공하는 인터페이스이다. 자바와 비슷하게, `Iterable<T>`를 상속한 `Collection<T>`를 상속한다. 리스트를 변경할 수 있는 메서드는 `MutableList<T>` 리스트에 정의되어 있다. 집합과 맵도 동일한 방식으로 `Set<out T>/MutableSet<T>` 그리고 `Map<K, out V>/MutableMap<K, V>`로 구분되어 있다.

리스트와 집합의 기본적인 사용법은 아래와 같다:

``` kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers)        // "[1, 2, 3]" 출력
numbers.add(4)
println(readOnlyView)   // "[1, 2, 3, 4]" 출력
readOnlyView.clear()    // -> 컴파일되지 않음

val strings = hashSetOf("a", "b", "c", "c")
assert(strings.size == 3)
```

코틀린은 리스트나 집합을 만들기 위한 구문 요소가 없다. `listOf()`, `mutableListOf()`, `setOf()`, `mutableSetOf()`와 같은 표준 라이브러리의 메서드를 사용한다.

성능이 중요하지 않은 코드는 맵을 생성할 때 `mapOf(a to b, c to d)` [이디엄](idioms.html#read-only-map)을 사용할 수 있다.

앞서 `readOnlyView` 변수는 numbers와 같은 리스트를 참조하고, 하부 리스트가 바뀌면 함께 바뀐다는 점에 유의하자. 리스트에 대한 유일한 참조가 읽기 전용 벼수라면, 콜레션을 완전히 불변(immutable)으로 할 것을 고려할 수 있다. 불변 콜렉션을 만드는 간단한 방법은 다음과 같다.

``` kotlin
val items = listOf(1, 2, 3)
```

현재, `listOf` 메서드는 배열 리스트를 이용해서 구현했는데, 향후에 변경할 수 없는 것을 알고 있다는 점을 활용해서 메모리 사용이 더 효율적인 완전한 불변 콜렉션 타입을 리턴하도록 구현할 것이다.

읽기 전용 타입은 [covariant](generics.html#variance)하다. 이는 Rectangle가 Shapre를 상속받은 경우, `List<Rectangle>`를 `List<Shape>`에 할당할 수 있다는 것을 뜻한다. 변경가능 콜렉션은 런타임에 실패가 발생할 수 있기 때문에, 이를 허용하지 않는다.

특정 시점에 콜렉션의 스냅샷을 호출자에 리턴하는데 그것을 변경하지 않도록 보장하고 싶을 때가 있다:

``` kotlin
class Controller {
    private val _items = mutableListOf<String>()
    val items: List<String> get() = _items.toList()
}
```

`toList` 확장 메서드는 단지 리스트 항목을 복사하기 때문에, 리턴된 리스트가 절대로 바뀌지 않음을 보장한다.

리스트와 집합에 대해 익숙해지면 좋은 몇 가지 유용한 확장 함수가 있다.

``` kotlin
val items = listOf(1, 2, 3, 4)
items.first == 1
items.last == 4
items.filter { it % 2 == 0 }   // Returns [2, 4]
rwList.requireNoNulls()
if (rwList.none { it > 6 }) println("No items above 6")
val item = rwList.firstOrNull()
```

... 또한, sort, zip, fold, reduce와 같은 유틸리티도 존재한다.

맵도 동일 패턴을 따른다. 다음과 같이 쉽게 생성하고 사용할 수 있다:

``` kotlin
val readWriteMap = hashMapOf("foo" to 1, "bar" to 2)
println(readWriteMap["foo"])
val snapshot: Map<String, Int> = HashMap(readWriteMap)
```
