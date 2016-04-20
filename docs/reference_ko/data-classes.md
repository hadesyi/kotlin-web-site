---
type: doc
layout: reference
category: "Classes and Objects"
title: "데이터 클래스"
---

# 데이터 클래스

종종 데이터만 갖고 다른 건 하지 않는 클래스를 만든다. 그런 클래스의 표준 기능은 보통 데이터에서 기계적으로 만든다. 코틀린에서는 이를 _데이터 클래스_라 부르며
`data`로 지정한다:

``` kotlin
data class User(val name: String, val age: Int)
```

컴파일러는 주요 생성자에 정의한 모든 프로퍼티로부터 다음의 멤버를 자동으로 생성한다:

  * `equals()`/`hashCode()` 쌍,
  * `"User(name=John, age=42)"` 형식의 `toString()`,
  * 프로퍼티 선언 순서에 따라 프로퍼티별로 대응하는 [`componentN()` 함수](multi-declarations.html),
  * `copy()` 함수 (아래 참고).

이 함수를 클래스 몸체나 베이스 타입에서 상속받을 경우, 생성하지 않는다.

생성한 코드가 일관되고 의미있는 기능을 갖도록 하기 위해 데이터 클래스는 다음을 충족해야 한다:

  * 주요 생성자는 최소 한 개 파라미터가 필요하다;
  * 모든 주요 생성자 파라미터는 `val`이나 `var`로 지정해야 한다;
  * 데이터 클래스는 추상, open, 실드 또는 내부(inner)일 수 없다;
  * 데이터 클래스를 다른 클래스를 확장할 수 없다(인터페이스 구현은 된다).

> JVM에서, 생성한 클래스가 파라미터 없는 생성자를 가져야 하면, 모든 프로퍼티에 대해 기본 값을 지정해야 한다([생성자](classes.html#constructors 참고).
>
> ``` kotlin
> data class User(val name: String = "", val age: Int = 0)
> ```

## 복사

객체를 복사할 때 종종 _일부_ 프로퍼티만 변경하고 나머지는 그대로 유지하고 싶을 때가 있다. 이를 위해 `copy()` 함수를 생성한다.
앞서 `User` 클래스의 경우, 복사 구현은 다음과 같다:

``` kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)     
```     

다음과 같이 코드를 작성할 수 있다.

``` kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

## 데이터 클래스와 분해 선언(Destructuring declarations)

데이터 클래스를 위해 생성한 _컴포넌트 함수_는 [분해 선언](multi-declarations.html)에 데이터를 사용할 수 있도록 한다:

``` kotlin
val jane = User("Jane", 35)
val (name, age) = jane
println("$name, $age years of age") // "Jane, 35 years of age" 출력
```

## 표준 데이터 클래스

표준 라이브러리는 `Pair`와 `Triple`을 제공한다. 많은 경우, 이름을 갖는 데이터 클래스를 사용하는 것이 더 좋은 설계가 된다.
왜냐면, 프로퍼티를 위한 의미있는 이름을 제공함으로써 코드 가독성이 높아지기 때문이다.
