---
type: doc
layout: reference
category: "Syntax"
title: "오브젝트 식과 선언"
---

# 오브젝트 식과 선언

때때로 상속한 클래스를 만들지 않고 어떤 클래스를 아주 약간 변경한 객체를 만들고 싶을 때가 있다.
자바에서는 이때 *임의 inner 클래스*를 사용한다.
코틀린은 이 개념을 *오브젝트 식*과 *오브젝트 선언*으로 약간 일반화했다.

## 오브젝트 식

특정 타입을 상속받은 임의 클래스의 객체를 생성할 때 다음과 같이 코드를 작성한다:

``` kotlin
window.addMouseListener(object : MouseAdapter() {
  override fun mouseClicked(e: MouseEvent) {
    // ...
  }

  override fun mouseEntered(e: MouseEvent) {
    // ...
  }
})
```

상위타입이 생성자를 가지면, 알맞은 생성자 파라미터를 전달해야 한다.
상위 타입을 여러 개면 콜론 뒤에 콤마로 구분해서 지정한다:


``` kotlin
open class A(x: Int) {
  public open val y: Int = x
}

interface B {...}

val ab = object : A(1), B {
  override val y = 15
}
```

만약 별도 상위타입이 없는 "단순히 객체"가 필요하다면, 다음 코드를 사용할 수 있다:

``` kotlin
val adHoc = object {
  var x: Int = 0
  var y: Int = 0
}
print(adHoc.x + adHoc.y)
```

자바의 임의 inner 클래스와 비슷하게, object 식의 코드는 외부 스코프의 변수에 접근할 수 있다.
(자바와 달리 final 변수로 제한되지 않는다.)

``` kotlin
fun countClicks(window: JComponent) {
  var clickCount = 0
  var enterCount = 0

  window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
      clickCount++
    }

    override fun mouseEntered(e: MouseEvent) {
      enterCount++
    }
  })
  // ...
}
```

## 오브젝트 선언

[싱글톤](http://en.wikipedia.org/wiki/Singleton_pattern)은 매우 유용한 패턴이다. 코틀린은 (스캍라를 따라해서) 쉽게 싱글톤을 선언할 수 있도록 했다:

``` kotlin
object DataProviderManager {
  fun registerDataProvider(provider: DataProvider) {
    // ...
  }

  val allDataProviders: Collection<DataProvider>
    get() = // ...
}
```

이를 *오브젝트 선언*이라고 한다. *object*{: .keyword } 키워드 뒤에 이름이 있으면, 이는 _식_을 말하는 것이 아니다.
오브젝트는 변수에 할당할 수 없고, 이름으로 참조할 수 있다. 오브젝트는 상위타입을 가질 수 있다:

``` kotlin
object DefaultListener : MouseAdapter() {
  override fun mouseClicked(e: MouseEvent) {
    // ...
  }

  override fun mouseEntered(e: MouseEvent) {
    // ...
  }
}
```

**주의**: 오브젝트는 로컬일 수 없다(예를 들어, 함수 안에 바로 중첩하는 것). 하지만, non-innter 클래스나 다른 오브젝트 선언에 중첩할 수는 있다.


### 컴페니언 오브젝트

클래스 안의 오브젝트 선언은 *companion*{: .keyword } 키워드를 붙일 수 있다:

``` kotlin
class MyClass {
  companion object Factory {
    fun create(): MyClass = MyClass()
  }
}
```

컴페니언 오브젝트의 멤버는 클래스 이름을 한정자로 사용해서 간단히 호출할 수 있다:

``` kotlin
val instance = MyClass.create()
```

컴페니언 오브젝트의 이름을 생략하면, `Companion`을 이름으로 사용한다:

``` kotlin
class MyClass {
  companion object {
  }
}

val x = MyClass.Companion
```

컴페니언 오브젝트의 멤버가 다른 언어의 정적 멤버처럼 보이긴 하지만, 런타임에는 실제 객체의 인스턴스 멤버이다.
예를 들어, 다음과 같이 인터페이스를 구현할 수 있다:

``` kotlin
interface Factory<T> {
  fun create(): T
}


class MyClass {
  companion object : Factory<MyClass> {
    override fun create(): MyClass = MyClass()
  }
}
```

하지만, JVM에서, `@JvmStatic` 애노테이션을 사용하면, 실제 정적 메서드와 필드로 생성된 컴페니언 오브젝트의 멤버를 가질 수 있다.
이에 대한 내용은 [자바 상호운용](java-interop.html#static-methods-and-fields) 절을 참고한다.


### 오브젝트 식과 오브젝트 선언의 세만틱 차이

오브젝트 식과 오브젝트 선언은 한 가지 중요한 차이가 있다:
There is one important semantic difference between object expressions and object declarations:

* 오브젝트 선언은 최초에 접근할 때까지 초기화를 (**lazily**) 미룬다.
* 오브젝트 식은 사용할 때 **즉시** 실행(초기화)된다.
