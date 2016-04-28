---
type: doc
layout: reference
category: "Syntax"
title: "함수"
---

# 함수

## 함수 선언

*fun*{: .keyword }을 사용해서 함수를 선언한다.

``` kotlin
fun double(x: Int): Int {
}
```

## 함수 사용

전통적인 방식으로 함수를 호출한다.

``` kotlin
val result = double(2)
```


멤버 함수를 호출할 때에는 점 부호를 사용한다.

``` kotlin
Sample().foo() // Sample 클래스의 인스턴스를 생성하고 foo 호출
```

### 중위 표현

다음의 경우 중위 표현(Infix notation)을 사용해서 함수를 호출할 수도 있다.

* 멤버 함수이거나 [확장 함수](extensions.html)일 때
* 파라미터를 한 개 가질 때
* `infix` 키워드로 지정했을 때

``` kotlin
// Int에 대한 확장 함수
infix fun Int.shl(x: Int): Int {
...
}

// 중위 표현을 사용해서 확장 함수 호출

1 shl 2

// 다음 코드와 같음

1.shl(2)
```

### 파라미터

함수 파라미터는 *name*: *type*와 같은 파스칼 표기법을 사용해서 정의한다. 파라미터는 콤마로 구분한다. 각 파라미터는 반드시 타입을 지정해야 한다.

``` kotlin
fun powerOf(number: Int, exponent: Int) {
...
}
```

### 기본 인자

함수 파라미터는 기본 값을 가질 수 있다. 해당 인자를 생략하면 이 기본 값을 사용한다.
이는 다른 언어 대비 오버로딩을 줄일 수 있도록 해 준다.

``` kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size()) {
...
}
```

기본 값은 타입 뒤에 **=**와 값을 사용해서 지정한다.

### 네임드 인자

함수 파라미터는 함수를 호출할 때 이름을 가질 수 있다. 파라미터 개수가 많거나 기본 값을 가진 경우 이름을 사용하면 매우 편리하다.

다음 함수를 보자.

``` kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
...
}
```

기본 인자를 사용하면 이 함수를 다음과 같이 호출할 수 있다.

``` kotlin
reformat(str)
```

하지만 기본 값을 사용하지 않고 호출하면 다음과 같은 코드가 된다.

``` kotlin
reformat(str, true, true, false, '_')
```

이름 인자를 사용하면 코드 가독성을 높일 수 있다.

``` kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
  )
```

모든 인자가 필요하지 않으면 더 간결해진다.

``` kotlin
reformat(str, wordSeparator = '_')
```

이름 인자 구문은 자바 함수를 호출할 때는 사용할 수 없다는 점에 유의하자. 왜냐면 자바 바이트코드는 함수 파라미터의 이름을 유지하지 않기 때문이다.


### Unit 리턴 함수

어떤 값도 리턴하지 않는 함수의 리턴 타입은 `Unit`이다. `Unit` 타입의 값은 `Unit` 한 개만 존재한다.
이 값을 명시적으로 리턴할 필요는 없다.

``` kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` 또는 `return` 생략 가능
}
```

`Unit` 리턴 타입 선언도 생략할 수 있다. 위 코드는 다음과 동일하다.

``` kotlin
fun printHello(name: String?) {
    ...
}
```

### 단일 식 함수

함수가 단일 식을 리턴하면, 중괄호를 생략할 수 있고 **=** 부호 뒤에 몸체를 지정할 수 있다.

``` kotlin
fun double(x: Int): Int = x * 2
```

이 경우 컴파일러가 리턴 타입을 유추할 수 있으므로 리턴 타입 지정을 [생략](#explicit-return-types)할 수 있다.

``` kotlin
fun double(x: Int) = x * 2
```

### 리턴 타입 지정

블록 몸체를 갖는 함수는 `Unit`을 리턴하는 것이 아니라면 반드시 리턴 타입을 지정해야 한다.
([생략할 수 있는 경우](#unit-returning-functions) 참고)
블록 몸체를 가진 함수는 몸체에 제어 흐름이 복잡할 수 있고 코드를 읽는 사람 입장에서 (또는 컴파일러 입장에서도) 리턴 타입이 명확하지 않을 수 있기 때문에
리턴 타입을 유추하지 않는다.


### 가변 인자 (Varargs)

함수 파라미터를 (보통 마지막 파라미터를) `vararg` 제한자로 지정할 수 있다.

``` kotlin
fun <T> asList(vararg ts: T): List<T> {
  val result = ArrayList<T>()
  for (t in ts) // ts는 Array
    result.add(t)
  return result
}
```

가변 인자를 사용하면 함수에 인자 개수를 가변적으로 전달할 수 있다.

```kotlin
  val list = asList(1, 2, 3)
```

`T` 타입의 `vararg` 파라미터는 함수 안에서 `T` 배열로 접근할 수 있다. 예의 경우 `ts` 변수는 `Array<out T>` 타입을 갖는다.

오직 한 개 파라미터만 `vararg`로 지정할 수 있다. 만약 `vararg` 파라미터가 마지막 파라미터가 아니면 그 뒤 값은 이름 인자 구문을 이용해서 전달한다. 파라미터가 함수 타입이면 괄호 밖에 람다를 전달하는 방식을 사용할 수 있다.

`vararg` 함수를 호출할 때 `asList(1, 2, 3)`처럼 인자를 한 개씩 전달할 수 있다.
만약 배열을 함수에 가변 인자로 전달하고 싶다면 **펼침** 연산자(배열 앞에 붙이는 `*`)를 사용하면 된다:

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

## 함수 범위

코틀린은 파일에서 함수를 최상위 레벨로 선언할 수 있다. 즉, 자바, C#이나 스칼라와 같은 언어처럼 함수를 갖는 클래스를 만들 필요가 없다.
최상위 레벨 함수뿐만 이나라 로컬 함수, 멤버 함수, 확장 함수로도 선언할 수 있다.

### 로컬 함수

코틀린은 로컬 함수를 지원한다. 다음과 같이 다른 함수 안에 함수를 선언할 수 있다.

``` kotlin
fun dfs(graph: Graph) {
  fun dfs(current: Vertex, visited: Set<Vertex>) {
    if (!visited.add(current)) return
    for (v in current.neighbors)
      dfs(v, visited)
  }

  dfs(graph.vertices[0], HashSet())
}
```

로컬 함수는 다른 함수의 로별 변수에 접근 가능하므로(클로저) 위 코드에서 *visited*를 로컬 변수로 바꿀 수 있다.

``` kotlin
fun dfs(graph: Graph) {
  val visited = HashSet<Vertex>()
  fun dfs(current: Vertex) {
    if (!visited.add(current)) return
    for (v in current.neighbors)
      dfs(v)
  }

  dfs(graph.vertices[0])
}
```

### 멤버 함수

멤버 함수는 클래스나 오브젝트 안에 정의한 함수이다.

``` kotlin
class Sample() {
  fun foo() { print("Foo") }
}
```

멤버 함수를 호출할 때는 점 부호를 사용한다.

``` kotlin
Sample().foo() // Sample 클래스의 인스턴스를 생성하고 foo 호출
```

클래스와 멤버 오버라이딩에 대한 내용은 [클래스](classes.html)와 [상속](classes.html#inheritance)을 참고한다.

## 지네릭 함수

함수는 함수 이름 앞에 화살괄호를 사용해서 지네릭 파라미터를 지정할 수 있다.

``` kotlin
fun <T> singletonList(item: T): List<T> {
  // ...
}
```

지네릭 함수에 대한 내용은 [지네릭](generics.html)을 참고한다.

## 인라인 함수

인라인 함수는 [여기](inline-functions.html)에서 설명한다.

## 확장 함수

확장 함수는 [여기](extensions.html)에서 설명한다.

## 고차 함수와 람다

고차 함수와 람다는 [여기](lambdas.html)에서 설명한다.

## 꼬리 재귀 함수

코틀린은 [꼬리 재귀](https://en.wikipedia.org/wiki/Tail_call)로 알려진 함수형 프로그래밍 스타일을 지원한다.
꼬리 재귀는 재귀 함수를 스택오버플로우 걱정이 없는 루프로 바꾸는 알고리즘을 허용한다.
함수에 `tailrec` 제한자를 붙이고 컴파일러가 재귀를 최적화할 수 있는 요건을 충족하면 빠르고 효율적인 루프 기반 버전으로 바꾼다.

``` kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```

이 코드는 수학의 상수값인 코사인의 고정소수점을 계산한다. 이 코드는 1.0에서 시작해서 더 이상 값이 바뀌지 않을 때까지 Math.cos를 반복해서 호출한다. 결과는 0.7390851332151607이다. 이 코드는 다음의 전통적인 방식과 같다.

``` kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```

`tailrec` 제한자가 가능하려면 함수는 반드시 마지막에 자신을 호출해야 한다. 재귀 호출 뒤에 다른 코드가 있다면 꼬리 재귀를 사용할 수 없다. try/catch/finally 블록에서는 사용할 수 없다. 현재 재귀 호출은 JVM 기반에서만 지원한다.
