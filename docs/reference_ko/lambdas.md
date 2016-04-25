---
type: doc
layout: reference
category: "Syntax"
title: "고차 함수와 람다"
---

# 고차 함수와 람다

## 고차 함수

고차 함수는 파라미터로 함수를 받거나 함수를 리턴하는 함수이다.
좋은 예가 `lock()`이다. 이 함수는 락 오브젝트와 함수를 받아 락을 구하고 함수를 실행하고 락을 해제한다:

``` kotlin
fun <T> lock(lock: Lock, body: () -> T): T {
  lock.lock()
  try {
    return body()
  }
  finally {
    lock.unlock()
  }
}
```

위 코드를 살펴보자. `body`는 [함수 타입](#function-types)인 `() -> T`를 갖는다. body에는 파라미터가 없고 `T` 타입 값을 리턴하는 함수를 전달해야 한다.
`lock`으로 보호하는 동안 *try*{: .keyword } 블록에서 body 함수를 실행하고 그 결과를 `lock()` 함수의 결과로 리턴한다.

`lock()` 함수를 호출하려면, 인자로 다른 함수를 전달하면 된다([함수 레퍼런스](reflection.html#function-references) 참고).

``` kotlin
fun toBeSynchronized() = sharedResource.operation()

val result = lock(lock, ::toBeSynchronized)
```

또 다른 간편한 방법은 [람다 식](#lambda-expressions-and-anonymous-functions)을 전달하는 것이다.

``` kotlin
val result = lock(lock, { sharedResource.operation() })
```

람다 식은 [뒤에서](#lambda-expressions-and-anonymous-functions) 설명하지만, 이 절을 이해하는데 필요한 람다 식에 대한 내용을 아래 요약했다.

* 람다 식은 항상 중괄호로 둘러 싼다.
* `->` 전에 파라미터를(존재하면) 선언하고(파라미터 타입은 생략 가능),
* `->` 뒤에 몸체가 온다(존재하면).

코틀린에서 함수에 마지막 파라미터로 함수를 전달하면, 괄호 밖에 파라미터를 지정할 수 있다.

``` kotlin
lock (lock) {
  sharedResource.operation()
}
```

고차 함수의 다른 예는 `map()`이다:

``` kotlin
fun <T, R> List<T>.map(transform: (T) -> R): List<R> {
  val result = arrayListOf<R>()
  for (item in this)
    result.add(transform(item))
  return result
}
```

이 함수는 다음과 같이 호출할 수 있다.

``` kotlin
val doubled = ints.map { it -> it * 2 }
```

함수 호출할 때 람다가 유일한 인자일 경우 함수 호출시 사용하는 괄호를 완전히 생략할 수 있다.

다른 유용한 규칙으로, 함수 리터럴의 파라미터가 한 개면 파라미터 선언을 (`->` 포함) 생략할 수 있고, 파라미터 이름이 `it`이 된다.

``` kotlin
ints.map { it * 2 }
```

이 규칙은 [LINQ-스타일](http://msdn.microsoft.com/en-us/library/bb308959.aspx)로 코드를 작성할 수 있도록 해 준다.

``` kotlin
strings.filter { it.length == 5 }.sortBy { it }.map { it.toUpperCase() }
```

## 인라인 함수

때때로 [인라인 함수](inline-functions.html)를 사용하면 고차 함수의 성능을 향상할 수 있는 이점이 있다.

## 람다 식과 임의 함수

람다 식 또는 임의 함수는 "함수 리터럴"로 함수 선언 없이 식으로 바로 전덜할 수 있다. 다음 예를 보자:

``` kotlin
max(strings, { a, b -> a.length() < b.length() })
```

`max` 함수는 두 번째 인자로 함수 값을 받는 고차 함수이다.
두 번째 인자는 그 자체가 함수인 함수 리터럴 식이다. 함수로서 이 식은 다음과 동일하다.

``` kotlin
fun compare(a: String, b: String): Boolean = a.length() < b.length()
```

### 함수 타입

함수가 다른 함수를 파라미터로 받으려면, 그 파라미터를 함수로 지정해야 한다.
예를 들어, 위에서 보여준 `max` 함수는 다음과 같이 정의한다.

``` kotlin
fun <T> max(collection: Collection<T>, less: (T, T) -> Boolean): T? {
  var max: T? = null
  for (it in collection)
    if (max == null || less(max, it))
      max = it
  return max
}
```

`less` 파라미터 타입은 `(T, T) -> Boolean` 함수 타입으로, 이 함수는 두 개의 `T` 타입 파라미터를 갖고 `Boolean` 타입을 리턴하며, 첫 번째 파라미터가 두 번째 파라미터보다 작으면 true를 리턴한다.

코드에서 네 번째 줄을 보면 `less`를 함수로 사용한다. 두 개의 `T` 타입 인자를 호출할 때 전달하고 있다.

함수 타입은 위와 같이 작성하거나 또는 각 파라미터에 의미를 문서화하고 싶다면, 네임드 파라미터를 사용한다.

``` kotlin
val compare: (x: T, y: T) -> Int = ...
```

### 람다 식 구문

람다 식(함수 타입 리터럴)의 완전한 구문은 다음과 같다.

``` kotlin
val sum = { x: Int, y: Int -> x + y }
```

람다 식은 항상 괄호로 둘러 싼다,
완전한 구문 형식에서는 괄혼 안에 파라미터 선언이 위치하며 타입 지정을 생략할 수 있다,
`->` 부호 다음에 몸체가 위치한다.
생략 가능한 것을 모두 생략하면, 다음과 같이 작성할 수 있다.

``` kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

람다 식은 파라미터를 한 개만 갖는 경우가 빈번하다.
코틀린이 그 시그너처를 알아낼 수 있으면, 파라미터 선언을 생략할 수 있으며,
코틀린이 자동으로 `it` 이름의 파라미터를 선언한다.

``` kotlin
ints.filter { it > 0 } // 이 리터럴의 타입은 '(it: Int) -> Boolean'이다.
```

함수의 마지막 파라미터가 함수면, 인자 목록을 갖는 괄호 밖에 람다 식을 전달할 수 있다.
[callSuffix](grammar.html#call-suffix) 문법을 참고한다.

### 임의 함수

위 람다 식 구문에서 빠진 게 하나 있는데, 함수 리턴 타입으로 지정하는 방법이다.
많은 경우 리턴 타입을 자동으로 유추하기 때문에 지정하지 않아도 된다.
하지만, 명시적으로 지정하고 싶다면 _임의 함수_ 구문을 사용할 수 있다.

``` kotlin
fun(x: Int, y: Int): Int = x + y
```

임의 함수는 이름이 없는 걸 제외하면 일반 함수 선언과 비슷하다. 몸체는 식이나 블록일 수 있다.
몸체는 (위 코드처럼) 식이거나 블록일 수 있다:

``` kotlin
fun(x: Int, y: Int): Int {
  return x + y
}
```

일반 함수와 동일한 방법으로 파라미터와 리턴 타입을 지정한다. 문맥에서 파라미터 타입을 유추할 수 있으면, 타입은 생략 가능하다.

``` kotlin
ints.filter(fun(item) = item > 0)
```

임의 함수에 대한 리턴 타입 유추는 일반 함수와 동일하게 동작한다.
식 몸체를 가진 임의 함수의 경우 리턴 타입을 자동으로 유추하고 블록 몸체를 가진 임의 함수의 경우 명시적으로 지정해야 한다(아니면 `Unit`으로 가정한다).

임의 함수 파라미터는 항상 괄호 안에 전달해야 한다. 괄호 밖에 함수를 위치시킬 수 있는 약식 구문은 람다 식만 가능하다.

람다 식과 임의 함수의 또 다른 차이점은 [non-local 리턴](inline-functions.html#non-local-returns)의 동작에 있다.
라벨 없는 *return*{: .keyword } 문은 항상 *fun*{: .keyword } 키워드로 선언한 함수에서 리턴한다.
이는 람다 식 안에서 *return*{: .keyword }을 사용하면 둘러 싼 함수에서 리턴하는 반면에
임의 함수 안에서 *return*{: .keyword }을 사용하면 임의 함수 자체에서 리턴함을 의미한다.

### 클로저

람다 식 또는 임의 함수 (또는 [로컬 함수](functions.html#local-functions)) 그리고 [오브젝트 식](object-declarations.html#object-expressions))은
그것의 _클로저_에 (즉, 외부 범위에 선언한 변수) 접근할 수 있다.
자바와 달리 클로저로 캡처한 변수를 수정할 수 있다.

``` kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
  sum += it
}
print(sum)
```


### 리시버를 갖는 함수 리터럴

코틀린은 함수 리터럴을 실행할 때 _리서버 객체_를 지정할 수 있다.
함수 리터럴 몸체 안에서 추가 한정자 없이 리시버 객체의 메서드를 호출할 수 있다.
이는 확장 함수와 유사하게, 함수 몸체 안에서 리시버 객체의 멤버에 접근할 수 있다.
이걸 사용하는 가장 중요한 예 중 하나가 [Type-safe Groovy-style builders](type-safe-builders.html)이다.

이런 함수 리터럴 타입은 리서버를 갖는 함수 타입이다.

``` kotlin
sum : Int.(other: Int) -> Int
```

이제 리시버 객체의 함수처럼 함수 리터럴을 호출할 수 있다:

``` kotlin
1.sum(2)
```

임의 함수 구문을 사용하면 함수 리터럴에 직접 리시버 타입을 지정할 수 있다.
이는 리시버를 갖는 함수 타입 변수를 선언하고 이를 나중에 사용해야 할 때 유용하다.

``` kotlin
val sum = fun Int.(other: Int): Int = this + other
```

문맥에서 리시버 타입을 유추할 수 있다면, 람다 식을 리시버를 가진 함수 리터럴로 사용할 수 있다.

``` kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
  val html = HTML()  // 리시버 객체 생성
  html.init()        // 람다에 리시버 객체를 전달
  return html
}


html {       // 리시버를 가진 람다 시작
    body()   // 리시버 객체의 메서드 호출
}
```
