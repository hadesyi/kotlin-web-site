---
type: doc
layout: reference
category: "Syntax"
title: "This 식"
---

# This 식

현재 _리시버_를 표시할 때 *this*{: .keyword } 식을 사용한다.

* [클래스](classes.html#inheritance)의 멤버에서 *this*{: .keyword }는 그 클래스의 현재 객체를 참조한다.
* [확장 함수](extensions.html)나 [리시버를 지정한 함수 리터럴](lambdas.html#function-literals-with-receiver)에서 *this*{: .keyword }는 점의 왼쪽 편에 전달한 _리시버_ 파라미터를 나타낸다.

*this*{: .keyword }에 한정자가 없으면 _가장 안쪽을 둘러싼 스코프_를 참조한다.
다른 스코프에서 *this*{: .keyword }를 참조하려면 _라벨 한정자_를 사용한다.

## 한정한 *this*{: .keyword }
{:#qualified}

외부 스코프([클래스](classes.html)나 [확장 함수](extensions.html) 또는
라벨이 붙은 [리시버를 사용하는 함수 리터럴](lambdas.html#function-literals-with-receiver))에서 *this*{: .keyword }에 접근하려면
`this@label`을 사용한다. `@label`은 *this*{: .keyword }로 접근하려는 스코프에 대한 [라벨](returns.html)이다.

``` kotlin
class A { // implicit label @A
  inner class B { // implicit label @B
    fun Int.foo() { // implicit label @foo
      val a = this@A // A의 this
      val b = this@B // B의 this

      val c = this // foo()의 리시버인 Int
      val c1 = this@foo // foo()의 리시버인 Int

      val funLit = lambda@ fun String.() {
        val d = this // funLit의 리시버
      }


      val funLit2 = { s: String ->
        // 둘러싼 람다 식이 리시버를 갖지 않으므로
        // foo()의 리시버
        val d1 = this
      }
    }
  }
}
```
