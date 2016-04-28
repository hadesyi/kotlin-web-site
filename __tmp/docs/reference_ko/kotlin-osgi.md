---
type: doc
layout: reference
category: "Tools"
title: "코틀린과 OSGi"
---

# 코틀린과 OSGi

코틀린 OSGi 지원 기능을 활성화려면 일반 코틀린 라이브러리 대신 `kotlin-osgi-bundle`를 포함하면 된다.
`kotlin-osgi-bundle`가 `kotlin-runtime`, `kotlin-stdlib` 그리고 `kotlin-reflect`를 모두 포함하고 있으므로 이 세 의존을 제거한다.
또한 외부 코틀린 라이브러리를 포함한 경우 주의해야 한다.
대부분 일반 코틀린 의존은 OSGi를 지원하지 않으므로 사용하면 안 되고 프로젝트에서 제거해야 한다.

## 메이븐

메이븐 프로젝트에 코틀린 OSGi 번들을 포함하기:

```xml
   <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-osgi-bundle</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
    </dependencies>
```

외부 라이브러리에서 표준 라이브러리 제외하기("* 제외"는 메이븐 3에서만 동작한다).

```xml
        <dependency>
            <groupId>some.group.id</groupId>
            <artifactId>some.library</artifactId>
            <version>some.library.version</version>

            <exclusions>
                <exclusion>
                    <groupId>org.jetbrains.kotlin</groupId>
                    <artifactId>*</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

## Gradle

그래들 프로젝트에 `kotlin-osgi-bundle` 포함하기:

```groovy
compile "org.jetbrains.kotlin:kotlin-osgi-bundle:$kotlinVersion"
```

의존성 전이로 포함되는 기본 코틀린 라이브러리를 제외하려면 다음 방법을 사용한다.

```groovy
dependencies {
 compile (
   [group: 'some.group.id', name: 'some.library', version: 'someversion'],
   .....) {
  exclude group: 'org.jetbrains.kotlin'
}
```

## FAQ

#### 왜 모든 코틀린 라이브러리에 필요한 매니페스트 옵션을 넣지 않았나?

메니페스트 옵션이 OSGi를 지원하는 가장 선호하는 방법이긴 하지만, 아쉽게도
["패키지 분리" 문제](http://wiki.osgi.org/wiki/Split_Packages)라 불리는 문제 때문에 할 수 없다.
이 문제는 쉽게 제거할 수 없고 아직 그렇게 큰 변경을 할 계획이 없다.
`Require-Bundle` 피처가 있지만 이 역시 최고의 선택은 아니며 사용을 추천하지 않는다.
그래서 SOGi를 위한 별도 아티팩트를 만들기로 결정했다.
