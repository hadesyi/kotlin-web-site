---
type: doc
layout: reference
title: "앤트 사용하기"
description: "이 튜토리얼은 코틀린 코드를 포함한 애플리케이션을 작성하기 위해 앤트를 사용하는 여러 상황을 설명한다."
---

# 앤트 사용하기

## 앤트 태스크 얻기

코틀린은 세 개의 앤트 태스크를 제공한다.

* kotlinc: JVM 대상 한 코틀린 컴파일러
* kotlin2js: 자바스크립트 대상 코틀린 컴파일러
* withKotlin: 표준 *javac* 앤트 태크스를 사용할 때 코틀린 파일을 컴파일하는 태스크

[코틀린 컴파일러]({{site.data.releases.latest.url}})의 *lib* 폴더에 위치한 *kotlin-ant.jar* 라이브러리에 이 태스크가 정의되어 있다.


## JVM 대상으로 코틀린 소스만 컴파일하기

프로젝트가 코틀린 소스 코드로만 구성되어 있을 때 프로젝트를 컴파일하는 가장 쉬운 방법은 *kotlinc* 태스크를 사용하는 것이다.

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <kotlinc src="hello.kt" output="hello.jar"/>
    </target>
</project>
```

${kotlin.lib}는 코틀린 표준 컴파일러의 압축을 푼 폴더를 가리킨다.

## JVM 대상으로 여러 루트를 갖는 코틀린 소스만 컴파일하기

프로젝트가 여러 소스 루트를 가지면 *src* 요소를 사용해서 경로를 정의한다.

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <kotlinc output="hello.jar">
            <src path="root1"/>
            <src path="root2"/>
        </kotlinc>
    </target>
</project>
```

## JVM 대상으로 코틀린과 자바 소스가 함께 컴파일하기

프로젝트에 코틀린과 자바 소스 코드가 함께 있다면 *kotlinc*를 사용할 수도 있지만,
태스크 파라미터 중복을 피하기 위해 *withKotlin* 태스크를 사용할 것을 권한다.

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <delete dir="classes" failonerror="false"/>
        <mkdir dir="classes"/>
        <javac destdir="classes" includeAntRuntime="false" srcdir="src">
            <withKotlin/>
        </javac>
        <jar destfile="hello.jar">
            <fileset dir="classes"/>
        </jar>
    </target>
</project>
```

`<withKotlin>`에 추가 명령행 인자를 지정하려면 `<compilerArg>` 파라미터를 사용하면 된다.
`kotlinc -help`로 사용할 수 있는 전체 인자 목록을 볼 수 있다.
`moduleName` 애트리뷰로 컴파일 할 모듈 이름을 지정할 수 있다.

``` xml
<withKotlin moduleName="myModule">
    <compilerarg value="-no-stdlib"/>
</withKotlin>
```


## 자바 스크립트 대상으로 한 개 소스 폴더를 컴파일하기

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <kotlin2js src="root1" output="out.js"/>
    </target>
</project>
```

## 자바 스크립트 대상으로 prefix, postfix, 그리고 sourcemap 옵션 사용하기

``` xml
<project name="Ant Task Test" default="build">
    <taskdef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <kotlin2js src="root1" output="out.js" outputPrefix="prefix" outputPostfix="postfix" sourcemap="true"/>
    </target>
</project>
```

## 자바스크립트 대상으로 한 개 소스 폴더와 meatInfo 옵션 사용하기

코틀린/자바스크립트 라이브러리로 변환 결과를 배포하길 원한다면 `metaInfo` 옵션을 유용하게 사용할 수 있다.
`metaInfo` 옵션을 `true`로 설정하면 컴파일할 때 바이너리 메타데이터를 가진 JS 파일을 추가로 생성한다. 변환 결과와 함께 이 파일을 배포해야 한다.

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <!-- 바이너리 디스크립터를 포함한 out.meta.js 파일을 생성한다. -->
        <kotlin2js src="root1" output="out.js" metaInfo="true"/>
    </target>
</project>
```

## 레퍼런스

요소와 애트리트뷰 전체 목록을 아래에 나열했다.

### kotlinc와 kotlin2js의 위한 공통 애트리뷰트

| 이름 | 설명 | 필수 | 기본 값 |
|------|-------------|----------|---------------|
| `src`  | 컴파일할 코틀린 소스 파일 또는 디렉토리 | Yes |  |
| `nowarn` | 모든 컴파일 경고를 무시함| No | false |
| `noStdlib` | 클래스패스에 코틀린 표준 라이브러리를 포함하지 않음 | No | false |
| `failOnError` | 컴파일하는 동안 에러를 발견하면 빌드에 실패함 | No | true |

### kotlinc 애트리뷰트

| 이름 | 설명 | 필수 | 기본 값 |
|------|-------------|----------|---------------|
| `output`  | 대상 디렉토리나 jar 파일 이름 | Yes |  |
| `classpath`  | 컴파일 클래스패스 | No |  |
| `classpathref`  | 컴파일 클래스패스 참조 | No |  |
| `includeRuntime`  | `output`이 jar 파일일 때 코틀린 런타임 라이브러리를 jar에 포함할지 여부를 지정 | No | true  |
| `moduleName` | 컴파일 할 모듈 이름 | No | (대상을 지정했다면) 대상 이름 아니면 프로젝트 이름 |


### kotlin2js 애트리뷰트

| 이름 | 설명 | 필수 |
|------|-------------|----------|
| `output`  | 대상 파일 | Yes |
| `library`  | 라이브러리 파일 (kt, dir, jar) | No |
| `outputPrefix`  | 생성할 자바스크립트 파일에 사용할 접두사 | No |
| `outputSuffix` | 생성할 자바스크립트 파일에 사용할 접미사 | No |
| `sourcemap`  | sourcemap 파일을 생성할지 여부 | No |
| `metaInfo`  | 바이너리 디스크립터를 가진 메타데이터 파일을 생성할지 여부 | No |
| `main`  | 컴파일러가 생성한 코드가 메인 함수를 호출하는지 | No |
