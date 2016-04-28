---
type: doc
layout: reference
category: "Syntax"
title: "타입 검사와 변환"
---

# 타입 검사와 변환

## `is`와 `!is` 연산자

`is`와 `!is` 연산자를 사용하면 런타임에 객체가 주어진 타입인지 아닌지 검사할 수 있다:

``` kotlin
if (obj is String) {
  print(obj.length)
}

if (obj !is String) { // !(obj is String)와 동일
  print("Not a String")
}
else {
  print(obj.length)
}
```

## 스마트 변환

코틀린은 컴파일러가 불변 값에 대해 `is`-검사를 추적해서 필요하면 자동으로 (안전한) 변환을 추가하기 때문에,
많은 경우 명시적인 변환 연산을 사용할 필요가 없다:

``` kotlin
fun demo(x: Any) {
  if (x is String) {
    print(x.length) // x를 자동으로 String으로 변환한다
  }
}
```

컴파일러가 똑띠하기 때문에 타입 비일치 검사에서 리턴하면 안전하게 변환할 수 있다는 것을 안다.

``` kotlin
  if (x !is String) return
  print(x.length) // x를 자동으로 String으로 변환한다
```

`&&`와 `||`의 우측에서도 안전하게 변환한다.:

``` kotlin
  // `||`의 우측에 대해 x를 자동으로 문자열로 변환
  if (x !is String || x.length == 0) return

  // `&&`의 우측에 대해 x를 자동으로 문자열로 변환
  if (x is String && x.length > 0)
      print(x.length) // x를 자동으로 String으로 변환한다
```

_스마트 변환_은 [*when*{: .keyword }-식](control-flow.html#when-expressions)과
[*while*{: .keyword }-루프](control-flow.html#while-loops)에도 동작한다.

``` kotlin
when (x) {
  is Int -> print(x + 1)
  is String -> print(x.length + 1)
  is IntArray -> print(x.sum())
}
```

컴파일러가 타입 검사와 사용 사이에 변수가 바뀌지 않는다는 것을 보장할 수 없으면 스마트 변환을 할 수 없다.
더 구체적으로 스마트 변환은 다음 규칙에 따라 가능하다.

  * *val*{: .keyword } 로컬 변수 - 항상;
  * *val*{: .keyword } 프로퍼티 - 프로퍼티가 private이나 internal이거나 또는 프로퍼티를 선언한 같은 모듈에서 검사를 수행한 경우. open 프로퍼티나 커스텀 getter를 가진 프로퍼티에는 스마트 변환을 적용할 수 없다;
  * *var*{: .keyword } 로컬 변수  - 검사와 사용 사이에 변수가 바뀌지 않고 그것을 수정하는 람다에 캡처되지 않는 경우;
  * *var*{: .keyword } 프로퍼티 - 절대 안 됨 (변수를 언제 어디서든 수정할 수 있으므로)


## "안전하지 않은" 변환 연산자

보통 변환 연산자는 변환을 할 수 없으면 익셉션을 발생한다. 그래서 이 변환을 *안전하지 않다고* 부른다.
안전하지 않은 변환은 *as*{: .keyword } 중위 연산자로 한다([연산자 우선순위](grammar.html#operator-precedence) 참고):

``` kotlin
val x: String = y as String
```

String 타입은 [nullable](null-safety.html)이 아니므로 *null*{: .keyword }을 `String`으로 변환할 수 없다. 예를 들어 `y`가 null이면 이 코드는 익셉션이 발생한다.
자바 변환 세만틱과 맞추기 위해 다음과 같이 변환 피연산자로 nullable 타입을 사용해야 한다.

``` kotlin
val x: String? = y as String?
```

## "안전한" (nullable) 변환 연산자

익셉션 발생을 피하려면 *안전하게* *as?*{: .keyword } 변환 연산자를 사용할 수 있다. 이 연산자는 실패시  *null*{: .keyword }을 리턴한다:

``` kotlin
val x: String? = y as? String
```

*as?*{: .keyword }의 우측 피연산자가 non-null 타입 `String`임에도 불구하고 변환 결과가 nullable 임에 주목하자.
