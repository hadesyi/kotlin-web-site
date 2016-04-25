---
type: doc
layout: reference
title: "메이븐 사용하기"
description: "이 튜토리얼은 코틀린 코드를 포함한 애플리케이션을 작성하기 위해 메이븐을 사용하는 여러 상황을 설명한다."
---

# 메이븐 사용하기

## 플러그인과 버전

*kotlin-maven-plugin*은 코틀린 소스와 모듈을 컴파일한다. 현재는 메이븐 v3만 지원한다.

*kotlin.version*으로 사용할 코틀린 버전을 정의한다. 코틀린 릴리즈와 버전 간의 관계를 아래 표시했다:

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


## 의존

코틀린은 애플리케이션 사용할 수 있는 방대한 표준 라이브러리를 갖고 있다.
pom 파일에 다음 의존을 설정한다.

``` xml
<dependencies>
    <dependency>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-stdlib</artifactId>
        <version>${kotlin.version}</version>
    </dependency>
</dependencies>
```

## 코틀린 소스 코드만 컴파일하기

소스 코드를 컴파일하려면 <build> 태그에 소스 디렉토리를 지정한다:

``` xml
<sourceDirectory>${project.basedir}/src/main/kotlin</sourceDirectory>
<testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>
```

코틀린 메이븐 플러그인에 소스 컴파일을 참조해야 한다:

``` xml

<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>

    <executions>
        <execution>
            <id>compile</id>
            <goals> <goal>compile</goal> </goals>
        </execution>

        <execution>
            <id>test-compile</id>
            <goals> <goal>test-compile</goal> </goals>
        </execution>
    </executions>
</plugin>
```

## 코틀린과 자바 소스 컴파일하기

자바와 코틀린을 함께 사용하는 애플리케이션 코드를 컴파일하려면, 자바 컴파일러 전에 코틀린 컴파일을 실행해야 한다.
메이븐에서는 maven-compiler-plugin 전에 kotlin-maven-plugin를 실행해야 함을 의미한다.

먼저 실행하려면 코틀린 컴파일 과정을 이전 단계인 process-sources로 옮기면 된다(더 나은 방법을 알고 있다면 자유롭게 제안해 달라):

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>

    <executions>
        <execution>
            <id>compile</id>
            <phase>process-sources</phase>
            <goals> <goal>compile</goal> </goals>
        </execution>

        <execution>
            <id>test-compile</id>
            <phase>process-test-sources</phase>
            <goals> <goal>test-compile</goal> </goals>
        </execution>
    </executions>
</plugin>
```

## OSGi

OSGi에 대한 지원은 [Kotlin OSGi 페이지](kotlin-osgi.html)를 참고한다.

## 예제

모든 메이븐 프로젝트 예제는 [GitHub 리포지토리에 직접 다운로드](https://github.com/JetBrains/kotlin-examples/archive/master/maven.zip)할 수 있다.
