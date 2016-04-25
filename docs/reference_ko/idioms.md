---
type: doc
layout: reference
category: "Basics"
title: "이디엄(Idioms)"
---

# 이디엄(Idioms)

코틀린에서 자주 사용하는 코딩 방식을 모은 것이다. 여러분도 자주 사용하는 코딩 방식이 있다면 풀 리퀘스트를 날려 공헌해보자.

### DTO 만들기 (POJO/POCO)

``` kotlin
data class Customer(val name: String, val email: String)
```

이 코드는 다음 기능을 가진 `Customer` 클래스를 제공한다:

* 모든 프로퍼티에 대한 getter (*var*{: .keyword } 프로퍼티는 setter 포함)
* `equals()`
* `hashCode()`
* `toString()`
* `copy()`
* 모든 프로퍼티에 대해 `component1()`, `component2()`, ... ([데이터 클래스](data-classes.html) 참고)


### 함수 파라피터의 기본 값

``` kotlin
fun foo(a: Int = 0, b: String = "") { ... }
```

### 리스트 필터

``` kotlin
val positives = list.filter { x -> x > 0 }
```

또는 다음과 같이 짧게:

``` kotlin
val positives = list.filter { it > 0 }
```

### 문자열 인터폴레이션(삽입)

``` kotlin
println("Name $name")
```

### 인스턴스 검사

``` kotlin
when (x) {
    is Foo -> ...
    is Bar -> ...
    else   -> ...
}
```

### map/list pair 탐색

``` kotlin
for ((k, v) in map) {
    println("$k -> $v")
}
```

`k`, `v`에 아무 이름이나 붙여도 된다.

### 범위 사용

``` kotlin
for (i in 1..100) { ... }
for (x in 2..10) { ... }
```

### 읽기 전용 리스트

``` kotlin
val list = listOf("a", "b", "c")
```

### 읽기 전용 맵

``` kotlin
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
```

### 맵 접근

``` kotlin
println(map["key"])
map["key"] = value
```

### 지연(lazy) 프로퍼티

``` kotlin
val p: String by lazy {
    // 문자열 계산
}
```

### 확장 함수

``` kotlin
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```

### 싱글톤 생성

``` kotlin
object Resource {
    val name = "Name"
}
```

### if not null 단축 표현

``` kotlin
val files = File("Test").listFiles()

println(files?.size)
```

### if not null과 else 단축 표현

``` kotlin
val files = File("Test").listFiles()

println(files?.size ?: "empty")
```

### null이면 문장 실행하기

``` kotlin
val data = ...
val email = data["email"] ?: throw IllegalStateException("Email is missing!")
```

### null이 아니면 실행하기

``` kotlin
val data = ...

data?.let {
    ... // null이 아니면 이 블록을 실행
}
```

### when 문장에서 리턴하기

``` kotlin
fun transform(color: String): Int {
    return when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
}
```

### 'try/catch' 식

``` kotlin
fun test() {
    val result = try {
        count()
    } catch (e: ArithmeticException) {
        throw IllegalStateException(e)
    }

    // result로 작업
}
```

### 'if' 식

``` kotlin
fun foo(param: Int) {
    val result = if (param == 1) {
        "one"
    } else if (param == 2) {
        "two"
    } else {
        "three"
    }
}
```

### `Unit`을 리턴하는 메서드를 빌더(Builder) 스타일로 사용

``` kotlin
fun arrayOfMinusOnes(size: Int): IntArray {
    return IntArray(size).apply { fill(-1) }
}
```


### 한 개 식을 갖는 함수

``` kotlin
fun theAnswer() = 42
```

이 코드는 다음 코드와 동일하다.

``` kotlin
fun theAnswer(): Int {
    return 42
}
```

코드를 더 짧게 하기 위해 이 코드를 다른 이디엄과 함께 사용할 수 있다. 다음은 *when*{: .keyword } 식과 함께 사용한 예이다.

``` kotlin
fun transform(color: String): Int = when (color) {
    "Red" -> 0
    "Green" -> 1
    "Blue" -> 2
    else -> throw IllegalArgumentException("Invalid color param value")
}
```

### 객체 인스턴스의 메서드 여러 번 호출하기 ('with')

``` kotlin
class Turtle {
    fun penDown()
    fun penUp()
    fun turn(degrees: Double)
    fun forward(pixels: Double)
}

val myTurtle = Turtle()
with(myTurtle) { // 100 픽셀 정사각형 그리기
    penDown()
    for(i in 1..4) {
        forward(100.0)
        turn(90.0)
    }
    penUp()
}
```

### 자바 7의 try-with-resources

``` kotlin
val stream = Files.newInputStream(Paths.get("/some/file.txt"))
stream.buffered().reader().use { reader ->
    println(reader.readText())
}
```

### 지네릭 타입 정보가 필요한 지네릭 함수를 위한 간편 형식

``` kotlin
//  public final class Gson {
//     ...
//     public <T> T fromJson(JsonElement json, Class<T> classOfT) throws JsonSyntaxException {
//     ...

inline fun <reified T: Any> Gson.fromJson(json): T = this.fromJson(json, T::class.java)
```
