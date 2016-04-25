---
type: doc
layout: reference
category: FAQ
title: "자바와 비교"
---

# 자바와 비교

## 코틀린에 있는 몇 가지 자바 이슈

코틀린은 자바가 겪고 있는 일련의 문제를 해결한다.

* [타입 시스템에서 null 참조를 제어](null-safety.html)한다
* [raw 타입 없음](java-interop.html)
* 코틀린의 배열은 [invariant](basic-types.html#Arrays)하다
* 자바의 SAM-변환과 달리 코틀린은 올바른 [function types](lambdas.html#function-types)을 갖는다
* 와일드카드 없는 [Use-site variance](generics.html#use-site-variance)
* 코틀린은 체크드 [익셉션](exceptions.html)이 없다

## 코틀린에 없고 자바에 있는 것

* [체크드 익셉션](exceptions.html)
* 클래스가 아닌 [기본 타입](basic-types.html)
* [정적 멤버](classes.html)
* [비-private 필드](properties.html)
* [와일드카드-타입](generics.html)

## 자바에 없고 코틀린에 있는 것

* [람다 식](lambdas.html) + [안리안 함수](inline-functions.html) = 훌륭한 커스텀 제어 구조
* [확장 함수](extensions.html)
* [Null-안전성](null-safety.html)
* [스마트 타입변환](typecasts.html)
* [문자열 템플릿](basic-types.html#strings)
* [프로퍼티](properties.html)
* [주요 생성자](classes.html)
* [필드-클래스 위임](delegation.html)
* [변수와 프로퍼티 타입을 위한 타입 추론](basic-types.html)
* [싱글톤](object-declarations.html)
* [Declaration-site variance & Type projections](generics.html)
* [Range 식](ranges.html)
* [연산자 오버로딩](operator-overloading.html)
* [컴페니언 오브젝트](classes.html#companion-objects)
* [데이터 클래스](data-classes.html)
* [읽기 전용 콜렉션과 변경 가능 콜렉션 인터페이스 분리](collections.html)
