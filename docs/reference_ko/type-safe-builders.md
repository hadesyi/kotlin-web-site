---
type: doc
layout: reference
category: "Syntax"
title: "타입-안전 그루비-스타일 빌더"
---

# 타입-안전 빌더

[빌더](http://www.groovy-lang.org/dsls.html#_nodebuilder) 개념은 *그루비* 커뮤니티에서 더 유명하다.
빌더는 반쯤 선언적인 방법으로 데이터를 정의할 수 있도록 해 준다. 빌더의 좋은 예로
[XML 생성](http://www.groovy-lang.org/processing-xml.html#_creating_xml),
[컴포넌트 배치](http://www.groovy-lang.org/swing.html),
[3D 장면 묘사](http://www.artima.com/weblogs/viewpost.jsp?thread=296081) 등이 있다.

많은 유스케이스를 위해 코틀린은 *타입-검사(type-check)* 빌더를 제공한다. 이 빌더는 예로 든 것을 그루비 자체에서 만든 동적-타입 구현보다 더 매력적으로 만들어준다.

나머지 경우를 위해 코틀린은 동적 타입 빌더를 지원한다.

## 타입-안전 빌더 예제

다음 코드를 보자:

``` kotlin
import com.example.html.* // 아래 선언 참고

fun result(args: Array<String>) =
  html {
    head {
      title {+"XML encoding with Kotlin"}
    }
    body {
      h1 {+"XML encoding with Kotlin"}
      p  {+"this format can be used as an alternative markup to XML"}

      // 애트리뷰트와 텍스트 컨텐트를 가진 엘리먼트
      a(href = "http://kotlinlang.org") {+"Kotlin"}

      // 혼합 컨텍트
      p {
        +"This is some"
        b {+"mixed"}
        +"text. For more see the"
        a(href = "http://kotlinlang.org") {+"Kotlin"}
        +"project"
      }
      p {+"some text"}

      // 생성된 컨텍트
      p {
        for (arg in args)
          +arg
      }
    }
  }
```

이 코드는 완전히 올바른 코틀린 코드이다.
이 코드를 브라우저에서 수정하고 실행해 볼 수 있다[(여기)](http://try.kotlinlang.org/#/Examples/Longer examples/HTML Builder/HTML Builder.kt).

## 동작 방식

코틀린에서 타입-안전 빌더를 구현하는 기법을 차례대로 살펴보자.
먼저 만들고 싶은 모델을 정의해야 한다. 이 예제의 경우 HTML 태그의 모델을 정의할 필요가 있다.
몇 개 클래스로 쉽게 이 모델을 만들 수 있다.
예를 들어, `HTML` 클래스는 `<html>` 태그를 표현하며, `<head>`와 `<body>`를 자식으로 정의한다.
([아래](#declarations) 이 클래스의 선언을 참고한다.)

이제 왜 다음과 같은 코드를 작성할 수 있는지 보자:

``` kotlin
html {
 // ...
}
```

`html`은 실제로 [람다 식](lambdas.html)을 인자로 받는 함수 호출이다.
이 함수는 다음과 같이 정의되어 있다:

``` kotlin
fun html(init: HTML.() -> Unit): HTML {
  val html = HTML()
  html.init()
  return html
}
```

이 함수는 이름이 `init`인 파라미터를 갖는다. 이 파라미터 자체도 함수이다.
`init` 함수 타입은 _리시버를 갖는 함수 타입_인 `HTML.() -> Unit`이다.
이는 함수에 `HTML` 타입의 인스턴스(리시버)를 전달해야 하고 함수 안에서 그 인스턴스의 멤버를 호출할 수 있음을 의미한다.
*this*{: .keyword } 키워드로 리시버에 접근할 수 있다:

``` kotlin
html {
  this.head { /* ... */ }
  this.body { /* ... */ }
}
```

(`head`와 `body`는 `html`의 멤버 함수이다.)

여기서 보통 *this*{: .keyword }를 생략할 수 있다. 이 코드는 이미 빌더와 같은 모양이다:

``` kotlin
html {
  head { /* ... */ }
  body { /* ... */ }
}
```

그러면 이 함수 호출은 무엇을 할까? 위에 정의한 `html` 함수의 몸체를 보자.
이 함수는 새로운 `HTML` 인스턴스를 생성하고, 인자로 전달받은 함수를 호출해서 생성한 인스턴스를 초기화하고(이 예제에서는
`HTML` 인스턴스의 `head`와 `body`를 호출한다),
그 인스턴스를 리턴한다.
이것이 정확하게 빌더가 해야 하는 것이다.

`HTML` 클래스의 `head`와 `body` 함수는 `html`과 비슷하게 정의한다.
유일한 차이점은 둘러싼 `HTML` 인스턴스의 `childrel` 콜렉션에 생성한 인스턴스를 추가하는 것이다.

``` kotlin
fun head(init: Head.() -> Unit) : Head {
  val head = Head()
  head.init()
  children.add(head)
  return head
}

fun body(init: Body.() -> Unit) : Body {
  val body = Body()
  body.init()
  children.add(body)
  return body
}
```

실제 이 두 함수는 같은 것을 하므로 지네릭 버전인 `initTag`를 만들 수 있다:

``` kotlin
  protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
    tag.init()
    children.add(tag)
    return tag
  }
```

이제 두 함수가 매우 간단해진다:

``` kotlin
fun head(init: Head.() -> Unit) = initTag(Head(), init)

fun body(init: Body.() -> Unit) = initTag(Body(), init)
```

`<head>`와 `<body>` 태그를 생성할 때 두 함수를 사용할 수 있다.

여기서 논의하는 것 중 다른 하나는 태그 몸체에 텍스트를 추가하는 것이다. 앞서 예제에서 다음과 같이 추가했다.

``` kotlin
html {
  head {
    title {+"XML encoding with Kotlin"}
  }
  // ...
}
```

기본적으로 단순히 태그 몸체에 문자열을 넣는데, 그 앞에 `+`가 있다.
따라서 이는 접두 `unaryPlus()` 오프레이션을 실행하는 함수 호출이다.
실제로 이 오퍼레이션을 `TagWithText` 추상 클래스(`Title`의 부모)의 멤버인 `unaryPlus()` 확장 함수로 정의했다:

``` kotlin
fun String.unaryPlus() {
  children.add(TextElement(this))
}
```

따라서, 여기서 접두문자 `+`는 문자열을 `TextElement`의 인스턴스로 감싸고 그 인스턴스를 `children` 콜렉션에 추가해서,
그것이 태그 트리에 알맞은 부분이 되도록 한다.

이 모든 것이 `com.example.html` 패키지에 정의되어 있는데 위 벌더 예제는 처음에 이 패키지를 임포트한다.
다음 절에서 이 패키지의 전체 정의를 읽을 수 있다.

## `com.example.html` 패키지의 전체 정의

다음 코드는 `com.example.html` 패키지의 코드이다(위 예제에서 사용하는 요소만 포함).
이는 HTML 트리를 만든다. [확장 함수](extensions.html)와
[리시버를 가진 람다](lambdas.html#function-literals-with-receiver)를 많이 쓰고 있다.

<a name='declarations'></a>

``` kotlin
package com.example.html

interface Element {
    fun render(builder: StringBuilder, indent: String)
}

class TextElement(val text: String) : Element {
    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent$text\n")
    }
}

abstract class Tag(val name: String) : Element {
    val children = arrayListOf<Element>()
    val attributes = hashMapOf<String, String>()

    protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
        tag.init()
        children.add(tag)
        return tag
    }

    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent<$name${renderAttributes()}>\n")
        for (c in children) {
            c.render(builder, indent + "  ")
        }
        builder.append("$indent</$name>\n")
    }

    private fun renderAttributes(): String? {
        val builder = StringBuilder()
        for (a in attributes.keys) {
            builder.append(" $a=\"${attributes[a]}\"")
        }
        return builder.toString()
    }


    override fun toString(): String {
        val builder = StringBuilder()
        render(builder, "")
        return builder.toString()
    }
}

abstract class TagWithText(name: String) : Tag(name) {
    operator fun String.unaryPlus() {
        children.add(TextElement(this))
    }
}

class HTML() : TagWithText("html") {
    fun head(init: Head.() -> Unit) = initTag(Head(), init)

    fun body(init: Body.() -> Unit) = initTag(Body(), init)
}

class Head() : TagWithText("head") {
    fun title(init: Title.() -> Unit) = initTag(Title(), init)
}

class Title() : TagWithText("title")

abstract class BodyTag(name: String) : TagWithText(name) {
    fun b(init: B.() -> Unit) = initTag(B(), init)
    fun p(init: P.() -> Unit) = initTag(P(), init)
    fun h1(init: H1.() -> Unit) = initTag(H1(), init)
    fun a(href: String, init: A.() -> Unit) {
        val a = initTag(A(), init)
        a.href = href
    }
}

class Body() : BodyTag("body")
class B() : BodyTag("b")
class P() : BodyTag("p")
class H1() : BodyTag("h1")

class A() : BodyTag("a") {
    public var href: String
        get() = attributes["href"]!!
        set(value) {
            attributes["href"] = value
        }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```
