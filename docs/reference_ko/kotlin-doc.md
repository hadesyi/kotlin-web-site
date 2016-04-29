---
type: doc
layout: reference
category: "Tools"
title: "코틀린 코드 문서화"
---

# 코틀린 코드 문서화

코틀린 코드를 문서화하기 위해 사용하는 언어를 **KDoc**이라 한다. 이것의 핵심은 블록 태그를 위한 Javadoc 구문(코틀린에 맞는 요소를 지원하기 위해 확장)과 인라인 마크업을 위해 마크다운을 조합한 것이다.

## 문서 생성하기

[Dokka](https://github.com/Kotlin/dokka)라는 도구를 이용해서 코틀린 문서를 생성한다.
사용법은 [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md) 문서를 참고한다.

Dokka는 그래들, 메이븐, 앤트 플러그인을 갖고 있어서 빌드 과정에 문서 생성을 통합할 수 있다.

## KDoc 구문

JavaDoc처럼 KDoc 주석도 `/**`로 시작해서 `*/`로 끝난다. 모든 주석 줄은 애스터리크(*)를 포함할 수 있지만 애스터리크는 주석 내용에 포함되진 않는다.

보통 주석 내용의 첫 번째 단락(첫 번째 빈 행이 올 때까지)에는 요소의 요약 설명을 넣고 자세한 내용은 그 뒤에 넣는다.

모든 블록 태그는 새 줄에서 `@` 문자로 시작한다.

다음은 KDoc을 이용해서 클래스를 문서화한 예이다.

``` kotlin
/**
 * A group of *members*.
 *
 * This class has no useful logic; it's just a documentation example.
 *
 * @param T the type of a member in this group.
 * @property name the name of this group.
 * @constructor Creates an empty group.
 */
class Group<T>(val name: String) {
    /**
     * Adds a [member] to this group.
     * @return the new size of the group.
     */
    fun add(member: T): Int { ... }
}
```

## 블록 태그

현재 KDoc은 다음 블록 태그를 지원한다.

#### `@param <name>`

함수의 파라미터나 클래스, 프로퍼티, 함수의 타입 파라미터 값을 문서화한다.
설명과 파라미터 이름을 더 잘 구분하고 싶다면 파라미터 이름을 대괄호로 둘러쌀 수 있다.
그래서 다음 두 구문은 동일하다.

```
@param name description.
@param[name] description.
```

#### `@return`

함수의 리턴 값을 문서화한다.

#### `@constructor`

클래스의 주요 생성자를 문서화한다.

#### `@property <name>`

지정한 이름을 가진 클래스의 프로퍼티를 문서화한다. 주요 생성자에 선언한 (프로퍼티 정의 앞에 주석을 직접 넣는게 어색한)
프로퍼티를 문서화하는데 이 태그를 사용할 수 있다.

#### `@throws <class>`, `@exception <class>`

메서드가 발생할 수 있는 익셉션을 문서화한다. 코틀린에는 체크드 익셉션이 없기 때문에
발생 가능한 모든 익셉션을 문서화할 것이라고 기대하진 않는다. 하지만 클래스 사용자에게 익셉션에 대한 정보가 유용하면
이 태그를 사용하면 된다.

#### `@sample <identifier>`

지정한 이름을 가진 함수의 몸체를 현재 요소의 문서화 결과에 삽입한다.
해당 요소를 어떻게 사용하는지 예를 보여주기 위한 용도로 사용한다.

#### `@see <identifier>`

문서의 **See Also** 블록에 지정한 클래스나 메서드에 대한 링크를 추가한다.

#### `@author`

문서화 대상 요소의 작성자를 지정한다.

#### `@since`

문서화 대상 요소를 추가한 소프트웨어 버전을 지정한다.

#### `@suppress`

문서 생성 대상에서 요소를 제외한다. 모듈의 공식 API에 포함은 되지 않지만 외부에 노출해야 하는 요소를 문서화할 때 사용할 수 있다.

> KDoc은 `@deprecated`를 지원하지 않는다. 대신 `@Deprecated`를 사용해야 한다.
{:.note}


## 인라인 마크업

인라인 마크업을 위해 KDoc은 정규 [Markdown](http://daringfireball.net/projects/markdown/syntax) 구문을 사용하며
코드에 있는 다른 요소에 링크하기 위한 약식 구문 지원을 확장했다.

### 요소에 링크하기

다른 요소(클래스, 메서드, 프로퍼티 또는 파라미터)에 링크하려면 단순히 대괄호 안에 이름을 넣으면 된다.

```
Use the method [foo] for this purpose.
```

링크에 커스텀 라벨을 지정하고 싶다면 마크다운의 레퍼런스-스타일 구문을 사용한다.

```
Use [this method][foo] for this purpose.
```

링크에 전체 이름을 사용할 수도 있다. JavaDoc과 달리 전체 이름은 항상 컴포넌트를 구분할 때 점 문자를 사용해야 하며, 이는 메서드 이름 전이라도 마찬가지다.

```
Use [kotlin.reflect.KClass.properties] to enumerate the properties of the class.
```

링크에 있는 이름을 해석할 때에는 문서화할 대상 안에서 사용할 이름과 같은 규칙을 사용한다.
특히, 이는 현재 파일에 이름을 임포트하면 KDoc 주석에서 그 이름을 사용할 때 완전한 이름을 사용할 필요가 없다는 것을 뜻한다.

KDoc은 링크에서 오버로딩한 멤버를 찾기 위한 별도 구문을 제공하지 않는다. 코틀린 문서 생성 도구는
같은 페이지에 있는 모든 오버로딩 함수를 문서화에 넣기 때문에, 오버로딩한 함수 중에서 링크 적용 대상을 식별할 필요가 없다.
