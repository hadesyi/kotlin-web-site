---
type: doc
layout: reference
category: "Syntax"
title: "리턴과 점프"
---

# 리턴과 점프

코틀린은 세 가지 structural 점프 연산자를 제공한다.

* *return*{: .keyword }. 기본적으로 가장 가깝게 둘러 싼 함수나 [anonymous function](lambdas.html#anonymous-functions)에서 리턴한다.
* *break*{: .keyword }. 가장 가깝게 둘러 싼 루프를 끝낸다.
* *continue*{: .keyword }. 가장 가깝게 둘러 싼 루프의 다음으로 넘어간다.

## Break와 Continue 라벨

코틀린의 모든 식은 *label*{: .keyword }로 표식을 가질 수 있다.
라벨은 `@` 기호 뒤에 구분자를 갖는다. 예를 들어 `abc@`, `fooBar@`는 유효한 라벨이다([문법](grammar.html#label) 참고).
식에 라벨을 붙이려면, 식 앞에 라벨을 위치시키면 된다.

``` kotlin
loop@ for (i in 1..100) {
  // ...
}
```

이제, 라벨로 *break*{: .keyword }나 *continue*{: .keyword }를 한정할 수 있다.

``` kotlin
loop@ for (i in 1..100) {
  for (j in 1..100) {
    if (...)
      break@loop
  }
}
```

라벨로 한정한 *break*{: .keyword }는 그 라벨이 붙은 루프 이후로 실행 지점을 이동한다.
*continue*{: .keyword }는 루프의 다음 반복을 처리한다.


## 라벨에 리턴하기

코틀린은 함수 리터럴, 로컬 함수와 객체 식에서 함수를 중첩할 수 있다.
한정한 *return*{: .keyword }는 outer 함수에서 리턴할 수 있도록 한다.
가장 중요한 쓰임새는 람다식에서 리턴하는 것이다. 다음 코드를 보자:

``` kotlin
fun foo() {
  ints.forEach {
    if (it == 0) return
    print(it)
  }
}
```

*return*{: .keyword } 식은 가장 가깝게 둘러싼 함수인 `foo`에서 리턴한다.
(비-로컬(non-local) 리턴은 [인라인 함수](inline-functions.html))로 전달한 람이식만 지원한다.)
만약 람다식에서 리턴하고 싶다면 라벨을 붙여서 *return*{: .keyword }을 한정해야 한다:

``` kotlin
fun foo() {
  ints.forEach lit@ {
    if (it == 0) return@lit
    print(it)
  }
}
```

이제, 이 return은 람다식에서 리턴한다. 종종, 임의 라벨을 사용하는게 더 편리하다:
그 라벨은 람다가 전달된 함수와 같은 이름을 갖는다.

``` kotlin
fun foo() {
  ints.forEach {
    if (it == 0) return@forEach
    print(it)
  }
}
```

다른 방법으로, 람다식 대신 [임의 함수](lambdas.html#anonymous-functions)를 사용할 수 있다.
임의 함수에서 *return*{: .keyword } 문은 임의 함수 자체에서 리턴한다.

``` kotlin
fun foo() {
  ints.forEach(fun(value: Int) {
    if (value == 0) return
    print(value)
  })
}
```

값을 리턴할 때, 파서는 한정한 리턴에 운선순위를 준다.

``` kotlin
return@a 1
```

이는 "라벨 식 `(@a 1)`을 리턴한다"가 아닌 "라벨 `@a`에 `1`을 리턴한다"를 의미한다.
