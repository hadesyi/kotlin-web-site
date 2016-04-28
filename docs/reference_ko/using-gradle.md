---
type: doc
layout: reference
title: "그래들 사용하기"
---

# 그래들 사용하기

그래들에서 코틀린을 빌드하려면 [*kotlin-gradle* 플러그인을 설정](#plugin-and-versions)하고 프로젝트에 [플러그인을 적용](#targeting-the-jvm)하고 [*kotlin-stdlib* 의존을 추가](#configuring-dependencies)해야 한다. IntelliJ IDEA에서는 Tools | Kotlin | Configure Kotlin를 사용하면 자동으로 처리한다.

## 플러그인과 버전

*kotlin-gradle-plugin*은 코틀린 소스와 모듈을 컴파일한다.

보통 사용할 코틀린 버전은 *kotlin_version* 프로퍼티로 정의한다.

``` groovy
buildscript {
   ext.kotlin_version = '<version to use>'

   repositories {
     mavenCentral()
   }

   dependencies {
     classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
   }
}
```

코틀린 릴리즈와 버전 간의 관계를 아래 표시했다.

<table>
<thead>
<tr>
  <th>Milestone</th>
  <th>Version</th>
</tr>
</thead>
<tbody>
{% for entry in site.data.releases.list %}
<tr>
  <td>{{ entry.milestone }}</td>
  <td>{{ entry.version }}</td>
</tr>
{% endfor %}
</tbody>
</table>

## JVM 대상

JVM을 대상으로 하려면 kotlin 플러그인을 적용하면 된다.

``` groovy
apply plugin: "kotlin"
```

코틀린 소스와 자바 소스를 같은 폴더나 다른 폴더에 위치시킬 수 있다. 기본 규칙은 다른 폴더를 사용하는 것이다.

``` groovy
project
    - src
        - main (root)
            - kotlin
            - java
```

기본 규칙을 사용하지 않을 경우 해당하는 *sourceSets* 프로퍼티를 수정해야 한다.

``` groovy
sourceSets {
    main.kotlin.srcDirs += 'src/main/myKotlin'
    main.java.srcDirs += 'src/main/myJava'
}
```

## 자바스크립트 대상

자바 스크립트가 대상이면 다른 플러그인을 사용한다.

``` groovy
apply plugin: "kotlin2js"
```

이 플러그인은 코틀린 파일만 처리하므로 (동일 프로젝트에 자바 파일을 포함하고 있다면) 코틀린과 자바 파일을 별도로 구분하는게 좋다. JVM도 대상으로 하면서 기본 규칙을 사용하지 않으면, *sourceSets*으로 소스 폴더를 지정해야 한다.

``` groovy
sourceSets {
    main.kotlin.srcDirs += 'src/main/myKotlin'
}
```

재사용 가능한 라이브러리를 만드려면 바이너리 디스크립터를 가진 JS 파일을 추가로 생성하기 위해 `kotlinOptions.metaInfo`를 사용한다.
이 파일을 변환 결과와 함께 배포해야 한다.

``` groovy
compileKotlin2Js {
	kotlinOptions.metaInfo = true
}
```


## 안드로이드 대상

안드로이드의 그래들 모델은 일반적인 그래들과 약간 다르다. 따라서 코틀린으로 작성한 안드로이드 프롤젝트를 빌드하려면 *kotlin* 대신 *kotlin-android* 플러그인을 사용해야 한다.

``` groovy
buildscript {
    ...
}
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
```

### 안드로이드 스튜디오

안드로이드 스튜디오를 사용하면 android에 다음 코드를 추가해야 한다.

``` groovy
android {
  ...

  sourceSets {
    main.java.srcDirs += 'src/main/kotlin'
  }
}
```

이 설정은 안드로이드 스튜디오가 코틀린 디렉토리를 소스 루트로 사용하게 한다. 따라서 프로젝트 모델을 IDE에 로딩할 때 올바르게 인식한다.



## 의존 설정

kotlin-gradle-plugin 의존과 함께 코틀린 표준 라이브러리에 대한 의존을 추가해야 한다.

``` groovy
buildscript {
   ext.kotlin_version = '<사용할 버전>'
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
  }
}

apply plugin: "kotlin" // 또는 자바 스크립트가 대상이면 apply plugin: "kotlin2js"

repositories {
  mavenCentral()
}

dependencies {
  compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}
```

프로젝트에서 코틀린 리플렉션이나 테스트 기능을 사용하려면 해당 의존을 추가로 넣는다.

``` groovy
compile "org.jetbrains.kotlin:kotlin-reflect:$kotlin_version"
testCompile "org.jetbrains.kotlin:kotlin-test:$kotlin_version"
```


## OSGi

OSGi 지원은 [Kotlin OSGi 페이지](kotlin-osgi.html)를 참고한다.

## 예제

[코틀린 리포지토리](https://github.com/jetbrains/kotlin)에 있는 예제:

* [코틀린](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/hello-world)
* [자바와 코틀린 함께 사용](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/mixed-java-kotlin-hello-world)
* [안드로이드](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-mixed-java-kotlin-project)
* [자바스크립트](https://github.com/JetBrains/kotlin/tree/master/libraries/tools/kotlin-gradle-plugin/src/test/resources/testProject/kotlin2JsProject)
