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

```
`joinToString()`을 모든 컬렉션의 확장 함수로 정의하자

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