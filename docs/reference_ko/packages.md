---
type: doc
layout: reference
category: "Syntax"
title: "패키지"
---

# 패키지

소스 파일은 패키지 선언으로 시작한다.

``` kotlin
package foo.bar

fun baz() {}

class Goo {}

// ...
```

클래스나 함수와 같은 소스 파일의 모든 내용은 선언한 패키지에 속한다.
위 예의 경우 `baz()`의 전체 이름은 `foo.bar.baz`이고 `Goo`의 전체 이름은 `foo.bar.Goo`이다.

패키지를 지정하지 않으면, 파일의 모든 내용은 이름이 없는 "기본(default)" 패키지에 속한다.

## 임포트

기본 임포트 외에 각 파일은 자신의 import 디렉티브를 포함할 수 있다.
임포트 구문은 [grammar](grammar.html#import)에서 설명한다.

다음은 한 개 이름을 임포트하는 예이다.

``` kotlin
import foo.Bar // Bar에 전체 이름을 사용하지 않고 접근할 수 있다.
```

해당 범위의 모든 요소(패키지, 클래스, 오브젝트 등)에 접근할 수도 있다:

``` kotlin
import foo.* // 'foo'의 모든 요소에 접근 가능
```

이름이 충돌하면, *as*{: .keyword } 키워드로 로컬에서 사용할 이름을 변경해서 충돌을 피할 수 있다:

``` kotlin
import foo.Bar // Bar로 접근
import bar.Bar as bBar // bBar는 'bar.Bar'를 의미
```

`import` 키워드는 클래스뿐만 아니라 다른 것도 임포트할 수 있다:

  * 최상위 레벨 함수와 프로퍼티;
  * [오브젝트 선언](object-declarations.html#object-declarations)에 선언한 함수와 프로퍼티;
  * [열거형 상수](enum-classes.html)

자바와 달리, 코틀린은 별도의 "import static" 구문을 지원하지 않는다. 모든 선언을 `import` 키워드를 사용해서 임포트할 수 있다.

## 최상위 선언의 가시성

최상위 선언이 *private*{: .keyword }이면, 그것이 선언된 파일에 대해 private이다([가시성 제한자](visibility-modifiers.html) 참고)
