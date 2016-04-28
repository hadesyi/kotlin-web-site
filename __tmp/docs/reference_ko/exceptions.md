---
type: doc
layout: reference
category: "Syntax"
title: "익셉션"
---

# 익셉션

## 익셉션 클래스

코틀린의 모든 익셉션 클래스는 `Throwable` 클래스의 자식이다.
모든 익셉션은 메시지, 스택 트레이스 그리고 선택적으로 원인을 갖는다.

익셉션 객체를 발생하려면 *throw*{: .keyword }-식을 사용한다.

``` kotlin
throw MyException("Hi There!")
```

익셉션을 잡으려면 *try*{: .keyword }-식을 사용한다.

``` kotlin
try {
  // some code
}
catch (e: SomeException) {
  // handler
}
finally {
  // optional finally block
}
```

*catch*{: .keyword } 블록은 없거나 한 개 이상 존재할 수 있다. *finally*{: .keyword } 블록은 생략할 수 있다.
하지만, 최소한 한 개의 *catch*{: .keyword } 블록이나 *finally*{: .keyword } 블록이 있어야 한다.

### try는 식이다

*try*{: .keyword }는 식이며 리턴 값을 가질 수 있다.

``` kotlin
val a: Int? = try { parseInt(input) } catch (e: NumberFormatException) { null }
```

*try*{: .keyword } 블록의 마지막 식이나 *catch*{: .keyword } 블록(또는 블록들)의 마지막 식을 *try*{: .keyword }-식의 리턴 값으로 사용한다.
*finally*{: .keyword } 블록의 내용은 식 결과에 영향을 주지 않는다.

## 체크트 익셉션

코틀린에는 체크트 익셉션이 없다. 없는데는 여러 이유가 있는데 간단한 예로 알아보자.

다음은 `StringBuilder` 클래스가 구현한 JDK 인터페이스이다.

``` java
Appendable append(CharSequence csq) throws IOException;
```

이 시그너처가 무엇을 말하나? 이는 문자열을 어딘가에 저장할 때마다(예를 들어, `StringBuilder`, 어떤 종류의 로그, 콘솔 등) `IOException`을 캐치해야 한다.
왜냐면 IO를 수행할지도 모르기 때문이다(`Writer`도 `Appendable`을 구현하고 있다).
따라서 도처에서 다음과 같은 코드를 작성하게 된다.

``` kotlin
try {
  log.append(message)
}
catch (IOException e) {
  // 안전해야 함
}
```

이는 좋지 않다. [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 65: *Don't ignore exceptions* 절을 보자.

Bruce Eckel는 [자바에 체크드 익셉션이 필요한가?](http://www.mindview.net/Etc/Discussions/CheckedExceptions) 라고 말했다.

> 작은 프로그램 조사에서는 익셉션 규약이 개발자 생산성과 코드 품질을 높여준다는 결론을 이끌지만, 대형 소프트웨어 프로젝트에서의 경험은 오히려 생산성을 낮추고 코드 품질 상승에 거의 효과가 없음이 밝혀졌다.

이와 관련된 글:

* [Java's checked exceptions were a mistake](http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html) (Rod Waldhoff)
* [The Trouble with Checked Exceptions](http://www.artima.com/intv/handcuffs.html) (Anders Hejlsberg)

## 자바 상호운용

자바 상호운용에 대한 정보는 [자바 상호운용](java-interop.html)의 익셉션 절을 참고하자.
