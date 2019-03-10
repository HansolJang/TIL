- 컬렉션, 문자열, 정규식을 다루기 위한 함수
- 이름 붙인 인자, 디폴트 파라미터 값, 중위 호출 문법 사용
- 확장 함수와 확장 프로퍼티를 사용해 자바 라이브러리 적용
- 최상위 및 로컬 함수와 프로퍼티를 사용해 코드 구조화

지금까지는 자바와 비슷하지만 더 간결히 표현하는 코틀린 문법을 봤다.

하지만 함수 정의와 호출은 코틀린 스타일로 달라졌다.

## 3.1 코틀린에서 컬렉션 만들기

- 코틀린에서는 몇가지 컬렉션을 간편하게 만들 수 있다
- 자바의 컬렉션을 그대로 가져다 쓰는 것

```
    val set = hashSetOf(1, 7, 53)
    
    val list = arrayListOf(2, 14, 3)
    
    val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
    
    println(set.last())   // 53
    println(list.max())   // 14
```

- 자바 클래스에 없는 `last()` 와 `max()` 와 같은 함수는 어디 존재할까?

## 3.2 함수를 호출하기 쉽게 만들기

- 상황: 자바에 기본적으로 구현되어있는 `toString()` 을 고치고 싶다.

```
    val list = arrayListOf(1, 2, 3)
    println(list)
    
    >>> [1, 2, 3]
    // (1; 2; 3) 으로 고칠 수 없을까?
```

```
    // 함수를 정의할 때 디폴트 값을 명시할 수 있다
    fun <T> joinToString(collection: Collection<T>,
    										separator: String = ", ",
    										prefix: String = "",
    										postfix: String = ""): String {
    		val result = StringBuilder(prefix)
    		for ((index, element) in collection.withIndex()) {
    				if (index > 0) result.append(seperator)
    				result.append(element)
    		}
    		result.append(postfix)
    		return result.toString()
    }

    val list = arrayListOf(1, 2, 3)
    // 함수 호출 코드만 보고 어떻게 동작할지 감이 잘 안잡힌다.
    println(list.joinToString(list, "; ", "(", ")"))
    
    >>> (1; 2; 3)
    
    // 코틀린에서는 인자 앞에 이름을 명시할 수 있다.
    println(joinToString(list, seperator = ";", prefix = "(", postfix = ")"))
    
    // 파라미터에 디폴트값이 있으므로 함수를 오버로딩하지 않고, 인자를 취사선택할 수 있다.
    joinToString(list)
    
    jotinToString(list, "; ")
    
    joinToString(list, postfix = ";", prefix = "#")
```

- 자바에는 디폴트 파라미터라는 개념이 없다.
- 자바에서 코틀린 함수를 호출할 땐 인자를 다 넣어줘야한다.
- 자바에서 자주 불리는 함수라면, 코틀린 함수에 `@JvmOverloads` 를 추가하자.
    - 저 어노테이션이 붙은 함수는 코틀린 컴파일러가 자동으로 인자가 다른 오버로딩한 자바 메소드를 만들어 준다.
- **코틀린에서는 함수를 클래스 안에만 선언할 필요가 없다**

- 자바에서는 객체 지향적으로 설계하므로, 객체에 들어가기 애매한 함수들을 기능 별로 묶어 Utils 와 같이 정적인 메소드만 모아 놓는 클래스를 자주 만들게 된다.
- 하지만, ***함수를 최상위 수준에 위치시킨다면?*** 모든 파일은 그 함수를 import 하게 될 것이다.

joinToString() 함수를 strings 패키지에 직접 넣어보자.

join.kt
```

    // @file:JvmName("StringFunctions")
    package strings
    
    // 코틀린에서는 함수 단독으로 호출
    // 자바에서는 JoinKt.joinToString()으로 호출
    fun joinToString(...): String { ... }
    
    // 최상위에 상수도 정의할 수 있다
    // const = public static final
    const val UNIX_LINE_ SEPARATOR = "\n"
```

- 클래스가 없이 strings 패키지에 joinToString 함수를 넣었다.
- JVM이 컴파일할 때 `JoinKt`라는 클래스를 자동으로 만들고 코드를 실행해준다.

## 3.3 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

확장함수: 어떤 클래스의 *멤버 함수*처럼 호출하지만, 클래스 *밖에 선언*되어 있는 함수

```
    package strings
    
    // this 생략 가능
    fun String.lastChar(): Char = this.get(this.length - 1)

    println("Kotlin".lastChar())    // 'n'
```

- receiver type: 확장함수를 받는 클래스 이름 (`String`)
- receiver object: 확장함수가 호출되는 대상이 되는 값 (식 안의 `this`)
- *캡슐화는 깨지지 않는다*
    - `private`, `protected`와 같이 보호 되어있는 멤버 함수나 멤버 변수는 확장 함수에서 사용할 수 없다


`joinToString()`을 모든 컬렉션의 확장 함수로 정의하자
```
    fun <T> Collection<T>.joinToString(seperator: String = ", ",
    																	prefix: String = "(", 
    																	postfix: String = ")"): String {
    		val result = StringBuilder(prefix)
    		for ((index, element) in this.withIndex()) {
    				if (index > 0) result.append(seperator)
    				result.append(element)
    		}
    		result.append(postfix)
    		return result.toString()
    }
```

- **확장함수는 오버라이드할 수 없다.**
```
    fun View.showOff() = println("I'm a View!")
    fun Button.showOff() = println("I'm a Button!")
    
    // 실제로 가리키는 것은 버튼이지만
    // I'm a View!가 출력된다.
    val view = Button()
    view.showOff()
```

- 멤버함수와 같은 이름, 파라미터의 확장함수를 정의한다면 **멤버함수**가 호출된다.

- val, var 형의 확장프로퍼티를 선언할 수 있다.
```
    val String.lastChar: Char
        get() = get(length - 1)
    
    var StringBuilder.lastChar: Char
        get() = get(length - 1)
        set(value: Char) {
            this.setCharAt(length - 1, value)
        }
```

## 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

- 가변길이 인자

### 가변길이 인자
```
    // Java: T... values
    fun listOf<T>(vararg values: T): List<T> {...}
    
    // 명시적으로 펼쳐줘야하지만 * 스프레드 연산자가 자동으로 펼쳐
    fun main(args: Array<String>) {
            val list = listOf("element", *args)
    }
```

### 값의 쌍 다루기: 중위 호출과 구조 분해 선언

```
    val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
    
    1 to "one"
    1.to("one")
```

- `to` 는 키워드가 아니다. 일반 메소드를 중위 호출한 것이다.

    infix fun Any.to(other: Any) = Pair(this, other)

- 중위호출하려면 함수앞에 `infix` 키워드를 추가해주면 된다.
- **파라미터가 1개** 뿐인 일반 메소드나 확장함수에 가능하다.

```
    // pair.first, pair.second 로 접근해야함
    val pair = 1 to "one"
    
    // 구조를 분해해 number, name 각각 접근 가능
    val (number, name) = 1 to "one"
    
    // 구조 분해로 index와 element 접근 가능
    for((index, element) in collection.withIndex()) {
     ...
    }
```

## 3.5 문자열과 정규식 다루기

- 문자열 나누기
    - 자바에서는 `split()`의 파라미터로 정규식을 받기 때문에 "."을 넘길 경우 모든 문자를 뜻해 문자열을 분리할 수 없다.
    - 코틀린에선 정규식, 구분자 여러개 등을 지정할 수 있는 확장함수를 제공한다.

```
    // 결과: []
    "12.345-6.A".split(".")
```

- 정규식과 3중 따옴표로 묶은 문자열

`/Users/documents/kotlin-book/chapter1.doc`

위와 같은 경로를 디렉터리, 파일이름, 확장자로 구분해보자.
```
    fun parsePath(path: String) {
            val directory = path.substringBeforeLast("/")   // /Users/documents/kotlin-book
            val fullName = path.substringAfterLast("/")     // chapter1.doc
            val fileName = fullName.substringBeforeLast(".")   // chapter1
            val extension = fullName.substringAfterLast(".")   // doc
    }

    fun parsePath(path: String) {
            val regex = """(.+)/(.+)\.(.+)""".toRegex()
            val matchResult = regex.matchEntire(path)
            if (matchResult != null) {
                    val (dir, filname, ext) = matchResult.destructured
            }
    }
```

- 정규식의 특성에 의해 가장 마지막 `/`까지의 문자열이 첫번째 결과가 된다.
- 3중 따옴표에선 `.`을 찾아내기 위해 `\\.` 이 아니라 `\.`이라고 명시한다.
```
    val kotlinLogo = """|  //
                                            .| //
                                            .|/ \"""
    
    println(kotlinLogo.trimMargin("."))
```

- `.`으로 여백을 구분해주지 않으면 띄어쓰기가 모두 출력된다.
- 들여쓰기를 해야할 땐 규칙을 정해 구분자를 써주도록 한다.

## 3.6 코드 다듬기: 로컬 함수와 확장

**DRY: Don't Repeat Yourself**

- 로컬 함수
    - 함수 내부의 함수로써 바깥 함수의 변수에 접근이 가능하다.

```
    fun saveUser(user: User) {
            // 중복!
        if (user.name.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: empty Name")
        }
    
            // 중복!
        if (user.address.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: empty Address")
        }
    
        // Save user to the database
    }
```

- `isEmpty()` 로 validate 하는 부분만 따로 빼자니 파라미터로 또 `name`, `address` 를 받아야한다.

```
    // 확장함수
    fun User.validateBeforeSave() {
            // 로컬함수
        fun validate(value: String, fieldName: String) {
            if (value.isEmpty()) {
                throw IllegalArgumentException(
                   "Can't save user $id: empty $fieldName")
            }
        }
    
            // 로컬함수 호출
        validate(name, "Name")
        validate(address, "Address")
    }
```