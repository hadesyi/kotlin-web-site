---
type: doc
layout: reference
category: "Syntax"
title: "위임"
---

# 위임

## 클래스 위임

[위임 패턴](https://en.wikipedia.org/wiki/Delegation_pattern)은 상속의 좋은 대안으로 증명되었다.
코틀린은 장식 코드(boilerplate code) 없이 언어 자체에서 위임 패턴을 지원한다.
`Derived` 클래스는 `Base` 인터페이스로부터 상속받을 수 있고, 모든 public 메서드를 지정한 객체로 위임할 수 있다:

``` kotlin
interface Base {
  fun print()
}

class BaseImpl(val x: Int) : Base {
  override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
  val b = BaseImpl(10)
  Derived(b).print() // 10 출력
}
```

`Derived`의 상위타입 목록에 *by*{: .keyword }-clause는 `Derived` 객체 내부에 `b`를 저장하고
컴파일이러가 `Base`의 모든 메서드에 대해 `b`로 위임하는 메서드를 `Derived`에 생성한다는 것을 나타낸다.
