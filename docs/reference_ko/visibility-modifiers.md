---
type: doc
layout: reference
category: "Classes and Objects"
title: "가시성 제한자(Visibility Modifiers)"
---

# 가시성 제한자

클래스, 오브젝트, 인터페이스, 생성자, 함수, 프로퍼티와 프로퍼티의 setter는 _가시성 제한자_를 가질 수 있다(getter는 항상 프로퍼티와 동일한 가시성을 갖는다).
코틀린에는 `private`, `protected`, `internal`, `public`의 네 개의 가시성 제한자가 존재한다.
명시적으로 제한자를 지정하지 않으면 기본적으로 `public` 가시성을 갖는다.

다른 종류의 스코프 선언별로 제한자에 대한 설명은 아래를 참고한다.

## 패키지

함수, 프로퍼티와 클래스, 오브젝트와 인터페이스는 "최상위 레벨"로 선언할 수 있다. 예를 들어 패키지 안에 바로 선언할 수 있다:

``` kotlin
// 파일 이름: example.kt
package foo

fun baz() {}
class Bar {}
```

* 만약 가시성 제한자를 지정하지 않으면 기본으로 `public`을 사용한다. 모든 곳에서 해당 선언에 접근할 수 있다;
* `private`으로 지정하면 해당 선언을 포함한 파일에서만 접근할 수 있다;
* `internal`로 지정하면 같은 모듈 안에서 접근할 수 있다;
* 최상위 레벨 선언은 `protected`로 지정할 수 없다.

예제:

``` kotlin
// 파일 이름: example.kt
package foo

private fun foo() {} // example.kt에서만 접근 가능

public var bar: Int = 5 // 프로퍼티는 모든 곳에서 접근 가능
    private set         // setter는 example.kt에서만 접근 가능

internal val baz = 6    // 같은 모듈 안에서 접근 가능
```

## 클래스와 인터페이스

클래스 안에서 선언할 때:

* `private`은 (클래스의 모든 멤버를 포함한) 클래스 안에서만 접근 가능하다;
* `protected` --- `private`과 동일 + 하위클래스에서 접근 가능하다;
* `internal` --- 선언한 클래스에 접근할 수 있는 *모듈에 속한* 모든 클라이언트가 `internal` 멤버에 접근 가능하다;
* `public` --- 선언한 클래스에 접근할 수 있는 모든 클라이언트가 `public` 멤버에 접근 가능하다.

자바 개발자 대상 *주의*: 코틀린에서 외부 클래스는 자신의 내부 클래스에 있는 private 멤버에 접근할 수 없다.

예제:

``` kotlin
open class Outer {
    private val a = 1
    protected val b = 2
    internal val c = 3
    val d = 4  // 기본으로 public

    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a에 접근 불가
    // b, c 그리고 d에 접근 가능
    // Nested와 e에 접근 가능
}

class Unrelated(o: Outer) {
    // o.a, o.b에 접근 불가
    // o.c(같은 모듈)와 o.d에 접근 가능
    // Outer.Nested에 접근 불가, 그리고 Nested::e에도 접근 불가
}
```

### 생성자

클래스의 주요 생성자에 가시성을 지정하려면 다음 구문을 사용한다(*constructor*{: .keyword } 키워드를 사용해야 한다):

``` kotlin
class C private constructor(a: Int) { ... }
```

여기서 생성자는 private이다. 모든 생성자의 기본 가시성은 `public`이며 클래스에 접근 가능한 모든 곳에서 접근 가능하다(예를 들어, `internal` 클래스의 생성자는 오직 동일 모듈에서만 접근 가능하다).

### 로컬 선언

로컬에 선언한 변수, 함수, 클래스는 가시성 제한자를 가질 수 없다.


## 모듈

`internal` 가시성 제한자는 같은 모듈에서 멤버에 접근할 수 있다. 더 구체적으로 말하면
모듈은 함께 컴파일되는 코틀린 파일 집합이다:

  * IntelliJ IDEA 모듈;
  * Maven이나 Gradle 프로젝트;
  * 한 <kotlinc> Ant 태스크 실행 시 컴파일되는 파일 집합.
