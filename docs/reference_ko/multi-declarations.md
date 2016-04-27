---
type: doc
layout: reference
category: "Other"
title: "분리 선언(Destructuring Declarations)"
---

# 분리 선언

다음 코드처럼 객체의 값을 여러 변수로 분리하면 편리할 때가 있다.

``` kotlin
val (name, age) = person
```

이 구문을 _분리 선언(destructuring declaration)_이라고 부른다. 분리 선언은 한 번에 여러 변수를 생성한다.
위 코드는 `name`과 `age` 두 변수를 선언하고 각 변수를 독립적으로 사용할 수 있다:

``` kotlin
println(name)
println(age)
```

분리 선언은 다음 코드로 컴파일된다:

``` kotlin
val name = person.component1()
val age = person.component2()
```

`component1()`과 `component2()` 함수는 코틀린에서 폭넓게 사용하는 _원칙(principle of conventions)_을 적용한 다른 예이다(`+`와 `*` 같은 연산자, *for*{: .keyword }-루프 등을 참고).
분리 선언의 우측에 위치한 것은 필요한 개수의 component 함수를 갖고 있으면 된다.
물론 `component3()`과 `component4()` 등 두 개 이상도 가능하다.

분린 선언에서 사용하려면 `componentN()` 함수에 `operator` 키워드를 적용해야 한다.

*for*{: .keyword }-루프에도 분리 선언이 가능하다.

``` kotlin
for ((a, b) in collection) { ... }
```

변수 `a`와 `b`는 콜렉션의 `component1()`와 `component2()`를 호출한 결과를 값으로 갖는다.

## 예제: 함수에서 두 값 리턴하기

함수에서 두 값을 리턴해야 한다고 가정해보자. 예를 들어, 결과 객체와 어떤 종류의 상태 값을 리턴해야 한다.
이를 하는 간단한 방법은 [_데이터 클래스_](data-classes.html)를 만들고 그 인스턴스를 리턴하는 것이다:

``` kotlin
data class Result(val result: Int, val status: Status)
fun function(...): Result {
    // 계산

    return Result(result, status)
}

// 이제, 이 함수 사용하기:
val (result, status) = function(...)
```

데이터 클래스는 `componentN()` 함수를 자동으로 선언하므로 분리 선언이 가능하다.

**주의**: 위 코드에서 표준 클래스 `Pair`와 `Pair<Int, Status>`를 리턴하는 `function()`을 사용할 수도 있다.
하지만, 데이터에 알맞은 이름을 붙이는게 보통 더 좋다.

## 예제: 분리 선언과 맵

다음은 아마도 맵을 탐색하는 가장 좋은 방법일 것이다:

``` kotlin
for ((key, value) in map) {
   // 키와 값으로 무언가를 한다
}
```

이게 되려면 다음을 충족해야 한다.

* `iterator()` 함수를 제공해서 맵을 값 시퀀스로 제공해야 한다,
* 각 요소를 `component1()`과 `component2()` 함수를 제공하는 페어로 제공해야 한다.

사실 표준 라이브러리가 이 확장을 제공한다:

``` kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()

```  

따라서 맵을(또한 데이터 클래스의 인스턴스 콜렉션을) 사용하는 *for*{: .keyword }-루프에서 자유롭게 분리 선언을 사용할 수 있다.
