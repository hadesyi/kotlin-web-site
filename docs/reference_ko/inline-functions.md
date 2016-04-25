---
type: doc
layout: reference
category: "Syntax"
title: "인라인 함수"
---

# 인라인 함수

[고차 함수](lambdas.html)를 사용하면 런타임에 어느 정도 불이익이 발생한다. 각 함수는 객체이고, 함수 몸체에서 접근하는 변수를 캡처한다.
(함수 객체와 클래스를 위한)메모리 할당과 버추얼 호출로 런타임에 부하가 발생한다.

하지만, 많은 경우 람다식을 인라인해서 이런 부하를 제거할 수 있다.
위의 `lock()` 함수가 이런 상황의 좋은 예이다. `lock()` 함수는 호출 위치에 쉽게 인라인할 수 있다.
다음을 보자.

``` kotlin
lock(l) { foo() }
```

파라미터를 위한 함수 객체를 생성하고 호출을 생성하는 대신, 컴파일러는 다음 코드를 만들어낼 수 있다.

``` kotlin
l.lock()
try {
  foo()
}
finally {
  l.unlock()
}
```

이게 우리가 애초에 원한 것이다. 그렇지 않나?

컴파일러가 이렇게 할 수 있으려면, `lock()` 함수에 `inline` 제한자를 붙인다:


``` kotlin
inline fun lock<T>(lock: Lock, body: () -> T): T {
  // ...
}
```

`inline` 제한자는 함수 자체와 함수에 전달하는 람다에 영향을 준다. 이것 모두 호출 위치에 인라인 된다.

인라인을 하면 생성된 코드가 증가할 수 있지만, 합리적으로 잘 활용하면(큰 함수는 인라인하지 않는 식으로), 성능에서 (특히 루프의 "megamorphic" 호출 위치는 더 많은) 보상을 받게 된다.

## noinline

인라인 함수에 전달된 람다 중 일부만 인라인 되길 원하면, `noinline` 제한자로 함수 파라미터를 지정하면 된다:

``` kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
  // ...
}
```

인라인 가능한 람다를 인라인 함수 안에서만 호출할 수 있거나 또는 인라인 가능한 인자로 전달할 수 있다. 하지만, `noinline`은 필드에 저장하거나 주변에 전달하는 것과 같은 방식으로 다룰 수 있다.
Inlinable lambdas can only be called inside the inline functions or passed as inlinable arguments,
but `noinline` ones can be manipulated in any way we like: stored in fields, passed around etc.

인라인 함수가 인라인 가능한 함수 파라미터가 없고 [reified type parameters](#reified-type-parameters)가 없으면,
컴파일러는 그 인라인 함수가 이점이 없다는 경고를 발생한다.(인라인이 필요하다 확신하면 이 경고를 무시해도 된다.)

## 비-로컬 리턴

코틀린에서, 이름 가진 함수나 임의 함수에서 나가려면 한정하지 않은 일반 `return`만 사용할 수 있다.
이는 람다에서 나가려면, [라벨](returns.html#return-at-labels)을 사용해야 한다는 것을 의미한다.
람다 안에서 단순 `return`은 허용하지 않는데, 그 이유는 람다는 둘러싼 함수를 리턴할 수 없기 때문이다.

``` kotlin
fun foo() {
  ordinaryFunction {
     return // 에러: `foo`를 여기서 리턴할 수 없다
  }
}
```

만약 람다를 전달한 함수가 인라인되면, 리턴도 같이 인라인된다. 그래서 다음을 허용한다.

``` kotlin
fun foo() {
  inlineFunction {
    return // OK: 람다를 인라인한다
  }
}
```

이 리턴(람다에 위치하지만 둘러싼 함수를 나가는)을 *비-로컬* 리턴이라 부른다.
이런 종류 리턴을 인라인 함수가 둘러싼 루프에서 사용하곤 한다.

``` kotlin
fun hasZeros(ints: List<Int>): Boolean {
  ints.forEach {
    if (it == 0) return true // hasZeros에서 리턴
  }
  return false
}
```

일부 인라인 함수는 파라미터로 전달받은 람다를 호출할 때 함수 몸체에서 직접 호출하지 않고 다른 실행 컨텍스트를 통해(예, 로컬 객체나 중첩 함수) 호출해야 할 수 있다.
이 경우 람다에서 비-로컬 흐름 제어를 허용하지 않는다.
이를 지정하려면, 람다 파라미터에 `crossinline` 제한자를 붙이면 된다.

``` kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ...
}
```


> 인라인된 람다에서 `break`와 `continue`는 아직 사용할 수 없는데, 앞으로 지원할 계횏이다.

## Reified type parameters

때때로 파라미터로 전달한 타입에 접근해야 할 때가 있다:

``` kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p?.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T
}
```

이 코드는 노드가 특정 타입을 가졌는지 확인하기 위해 트리를 탐색하고 리플렉션을 사용한다.
모두 좋은데, 호출하는 코드가 이쁘지(pretty ^^) 않다:

``` kotlin
myTree.findParentOfType(MyTreeNodeType::class.java)
```

실제로는 이 함수에 단순히 타입을 전달해서 다음과 같이 호출하길 원하는 것이다.

``` kotlin
myTree.findParentOfType<MyTreeNodeType>()
```

이렇게 할 수 있도록, 인라인 함수는 *reified type parameters*를 지원하며, 이를 사용한 코드는 다음과 같다:

``` kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p?.parent
    }
    return p as T
}
```

타입 파라미터에 `reified` 제한자를 적용하면, 마치 클래스처럼 타입 파라미터에 접근할 수 있다.
인라인 함수이므로, 리플렉션이 필요 없고, `!is`와 `as` 같은 일반 연산자가 동작한다.
또한, 앞서 언급한 `myTree.findParentOfType<MyTreeNodeType>()`처럼 호출할 수 있댜:

많은 경우 리플렉션이 필요없긴 하지만, reified type parameter에 리플렉션을 사용할 수 있다:

``` kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
  println(membersOf<StringBuilder>().joinToString("\n"))
}
```

(인라인이 아닌) 일반 함수는 reified parameters를 가질 수 없다.
런타임 표현을 갖지 않는 타입(reified type parameter가 아니거나 `Nothing`과 같은 가공 타입)은
reified type parameter를 위한 인자로 사용할 수 없다.

저수준 설명은 [스펙 문서](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md) 참고.
