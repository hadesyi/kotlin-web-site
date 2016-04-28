---
type: doc
layout: reference
category: "Syntax"
title: "프로퍼티와 필드"
---

# 프로퍼티와 필드

## 프로퍼티 선언

코틀린 클래스는 프로퍼티를 가질 수 있다.
*var*{: .keyword } 키워드로 변경 가능 프로퍼티를 선언하고 *val*{: .keyword } 키워드로 읽기 전용 프로퍼티를 선언한다.

``` kotlin
public class Address {
  public var name: String = ...
  public var street: String = ...
  public var city: String = ...
  public var state: String? = ...
  public var zip: String = ...
}
```

프로퍼티를 사용하려면 자바 필드처럼 단순히 이름으로 참조하면 된다.

``` kotlin
fun copyAddress(address: Address): Address {
  val result = Address() // 코틀린은 'new' 키워드가 없다
  result.name = address.name // accessor 실행
  result.street = address.street
  // ...
  return result
}
```

## Getter와 Setter

프로퍼티 선언의 전체 구문은 다음과 같다.

``` kotlin
var <propertyName>: <PropertyType> [= <property_initializer>]
  [<getter>]
  [<setter>]
```

initializer, getter, setter는 선택 사항이다. initializer나 오버라이딩한 베이스 클래스 멤버에서 타입을 유추할 수 있다면 프로퍼티 타입을 생략해도 된다.

예:

``` kotlin
var allByDefault: Int? // 에러: initializer 필요, 기본 getter와 setter 포함
var initialized = 1 // Int 타입, 기본 getter와 setter 가짐
```

읽기 전용 프로퍼티 선언의 전체 구문은 변경 가능 프로퍼티와 두 가지가 다르다. `var` 대신에 `val`로 시작하고 setter를 허용하지 않는다.

``` kotlin
val simple: Int? // Int 타입과 기본 getter를 갖고, 생성자에서 초기화해야 한다
val inferredType = 1 // Int 타입과 기본 getter를 갖는다
```

프로퍼티 선언에서 바로 뒤에 일반 함수와 유사한 커스텀 accessor를 작성할 수 있다. 다음은 커스텀 getter의 예다.

``` kotlin
val isEmpty: Boolean
  get() = this.size == 0
```

다음은 커스텀 setter의 예다.

``` kotlin
var stringRepresentation: String
  get() = this.toString()
  set(value) {
    setDataFromString(value) // 문자열을 파싱해서 다른 프로퍼티에 할당한다
  }
```

보통 setter의 파라미터 이름으로 `value`를 사용하지만 원하는 다른 이름을 사용해도 된다.

accessor의 기본 구현을 변경하지 않고 가시성을 변경하거나 애노테이션을 적용하고 싶다면
몸체 선언 없이 accessor를 정의할 수 있다.

``` kotlin
var setterVisibility: String = "abc"
  private set // setter는 private이고 기본 구현을 갖는다

var setterWithAnnotation: Any? = null
  @Inject set // setter에 @Inject를 붙인다
```

### Backing 필드

코틀린 클래스는 필드를 가질 수 없다. 하지만 커스텀 accessor를 사용하면 backing 필드가 필요할 때가 있다.
이를 위해 코틀린은 `field` 식별자를 사용해서 접근할 수 있는 backing 필드를 제공한다.

``` kotlin
var counter = 0 // 초기화 값을 backing 필드에 직접 쓴다
  set(value) {
    if (value >= 0)
      field = value
  }
```

`field` 식별자는 프로퍼티 accessor에서만 사용할 수 있다.

accessor 몸체에서 backing 필드를 사용하면(또는 기본 구현을 사용하면) 컴파일러는 backing 필드를 생성한다. 그렇지 않으면 생성하지 않는다.

예를 들어 다음 경우 backing 필드가 없다.

``` kotlin
val isEmpty: Boolean
  get() = this.size == 0
```

### Backing 프로퍼티

"기본(implicit) backing 필드" 방식과 맞지 않는 것을 하고 싶다면 *backing 프로퍼티*로 대체할 수 있다.

``` kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
  get() {
    if (_table == null)
      _table = HashMap() // 타입 파라미터를 유추
    return _table ?: throw AssertionError("Set to null by another thread")
  }
```

기본 getter와 setter로 private 프로퍼티에 접근하는 것을 최적화해서 함수 호출에 따른 오버헤드가 없기 때문에 모든 점에서 자바와 같다.


## 컴파일 타임 상수

컴파일 타임에 값을 알 수 있는 프로퍼티는 `const` 제한자를 이용해서  _컴파일 타임 상수_로 지정할 수 있다.
이 프로퍼티는 다음 요건을 충족해야 한다.

  * 최상위 레벨 또는 `오브젝트`의 멤버
  * `String`이나 기본 타입 값으로 초기화
  * 커스텀 getter 없음

이 프로퍼티는 애노테이션에서 사용할 수 있다.

``` kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```


## 초기화 지연(Late-Initialized) 프로퍼티

보통 non-null 타입을 갖는 프로퍼티를 선언하면 생성자에서 초기화해야 한다.
하지만 이게 늘 편한 것은 아니다. 예를 들어 의존 주입이나 단위 테스트의 셋업 메서드에서 프로퍼티를 초기화할 수 있다.
이 경우 생성자에 non-null initializer를 제공할 수 없지만 클래스 몸체 안에서 프로퍼티를 참조할 때 null 검사를 하고 싶지는 않을 것이다.

이 경우를 처리하기 위해 프로퍼티를 `lateinit`로 제한할 수 있다.

``` kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // null 검사 없이 직접 접근
    }
}
```

이 제한자는 (주요 생성자가 아닌) 클래스 몸체에 선언되고 커스텀 getter나 setter를 갖지 않는 `var` 프로퍼티에만 사용할 수 있다.
프로퍼티 타입은 non-null이어야 하고 기본 타입이면 안 된다.

초기화되기 전에 `lateinit` 프로퍼티에 접근하면 초기화되지 않은 프로퍼티에 접근했음을 정확하게 알려주기 위해 특수한 익셉션을 발생한다.

## 프로퍼티 오버라이딩

[멤버 오버라이딩](classes.html#overriding-members) 참고

## 위임 프로퍼티

대부분 프로퍼티는 단순히 backing 필드에서 값을 읽거나 쓴다.
반면에 커스텀 getter와 setter를 갖는 프로퍼티는 프로퍼티의 행위를 구현할 수 있다.
프로퍼티 행위 구현에는 프로퍼티가 어떻게 작동하는지에 대한 어떤 공통 패턴이 있다. 키로 맵에서 읽기, 데이터베이스 접근하기, 접근 시 리스너에 통지하기 등이 공톤 패턴에 해당한다.

이런 공통 행위는 [_위임 프로퍼티_](delegated-properties.html)를 사용하는 라이브러리로 구현할 수 있다.
