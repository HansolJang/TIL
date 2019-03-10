- 클래스와 인터페이스
- 뻔하지 않은 생성자와 프로퍼티
- 데이터 클래스
- 클래스 위임
- `object` 키워드 사용

## 4.1 클래스 계층 정의

### 인터페이스
```
    interface Clickable {
        fun click()
    		// 디폴트 구현이 있는 메소드
        fun showOff() = println("I'm clickable!")
    }
    
    interface Focusable {
        fun setFocus(b: Boolean) =
            println("I ${if (b) "got" else "lost"} focus.")
        fun showOff() = println("I'm focusable!")
    }
    
    class Button : Clickable, Focusable {
    		// override 키워드로 오버라이드 됨을 명시
        override fun click() = println("I was clicked")
    
    		// showOff를 오버라이드 하지 않으면 호출할 수 없음!
        override fun showOff() {
            super<Clickable>.showOff()   // 어떤 상위 메소드를 호출할 것인지 명시
            super<Focusable>.showOff()
        }
    }
```

### open, final, abstract 변경자: 기본적으로 final

- 상속하는 법에 대한 정확한 규칙이 없으면, 작성한 사람의 의도와는 다르게 오버라이드 될 수 있다.
- 모든 하위 클래스를 분석하는 것은 불가능하다.
- 특별히 상속하도록 만든 것이 아니라면 모두 `final`로!
    - 하위 메소드의 오버라이딩을 막기 위해 함수 앞에 `final`을 명시할 수 있다.
- `open` 변경자를 붙여야만 상속이 가능하다. 클래스, 메소드, 프로퍼티에도 붙일 수 있다.
- 이는 스마트 캐스트가 가능한 이유이기도 하다.
    - 타입 검사 뒤에 타입이 변경될 여지가 없다
    - `val` (final)이고 `커스텀 접근자` (set())가 없다

[클래스 내에서 상속 제어 변경자의 의미](https://www.notion.so/6fcc2d0a0ae544ccb3834ab6910bda31)

### 가시성 변경자: 기본적으로 public

- 외부 접근을 제한
- 자바의 기본 가시성: 패키지 전용 → 코틀린에선 `internal` 키워드 제공
- `private`클래스 선언 가능
- 자바에서는 `internal` → `public` 으로 컴파일된다.
    - 자바 코드로 `internal` 한 코틀린 멤버를 외부에서 접근 가능
    - 코틀린 컴파일러가 `mangling`

[코틀린의 가시성 변경자](https://www.notion.so/0b5fbe431b4f4b469cd43178c3e879ad)

```
    internal open class TalkativeButton: Focusable {
    		private fun yell() = println("hey!")
    		protected fun whisper() = println("let's talk!")
    }
    
    // public한 확장 함수가 internal한 클래스를 받을 수 없음
    fun TalkativeButton.giveSpeech() {
    		// private 함수 접근 불가
    		yell()
    		// protected 접근 불가
    		// 자바에선 같은 패키지라면 protected 접근 가능, 코틀린은 불가
    		whisper()
    }
```

### 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

- 클래스안에 다른 클래스 선언
- 내부 클래스에서 외부 클래스 접근 ***불가***
    - 접근 가능하도록 하려면 `inner` 키워드 명시
    - 외부 클래스 접근시 `this@Outer`

[자바와 코틀린의 중첩 클래스와 내부 클래스의 관계](https://www.notion.so/4a06f656022b424e9608a8cb817fb4b2)

### 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

`sealed`

- 상위 클래스를 상속한 하위 클래스 정의를 제한
- 클래스 외부에서 상속할 수 없음

```
    // 클래스 내부에서 중첩 클래스로 모든 상속 클래스 선언
    sealed class Expr {
        class Num(val value: Int) : Expr()
        class Sum(val left: Expr, val right: Expr) : Expr()
    }
    
    fun eval(e: Expr): Int =
        when (e) {
            is Expr.Num -> e.value
            is Expr.Sum -> eval(e.right) + eval(e.left)
    				// sealed 되어 있기 때문에 else 하지 않아도 됨
        }
```

## 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

### 클래스 생성자: 주 생성자와 초기화 블록

```
    // 기본 클래스 선언
    class User {
            val nickname: String
    
            constructor(_nickname: String) {
                    this.nickname = _nickname
            }
    }
    
    // init 블럭을 활용한 클래스 선언
    // init 블럭 안에서만 생성자에 있는 _nickname 변수에 접근 가능
    class User constructor(_nickname: String) {
            val nickname: String
            init {
                    nickname = _nickname
            }
    }
    
    // 디폴트 생성자를 바로 정의하는 클래스 선언
    class User(val nickname: String)
    
    // 생성자에 디폴트값 선언 = 생성자에 생략 가능
    class User(val nickname: String, 
                       val isSubscribed: Boolean = true)
```

```
    val hansol = User("한솔")            // isSubscribed 생략 가능
    val gye = User("계영", false)
```

- 생성자에 디폴트 값이 있으면 호출할 때 안 넣어줘도 된다.
- 컴파일러가 자동으로 파라미터가 없는 생성자를 만들어 준다.
- 이는 파라미터를 생략할 수 있게 함으로써 DI 라이브러리와의 통합도 쉽게 만들어 준다.
- 디폴트 생성자를 선언하지 않으면 `User()`을 자동으로 만들어 준다.
- **클래스를 상속할 땐 생성자를 반드시 호출해야 한다**.

```
    // 생성자가 있는건 클래스, 없는건 인터페이스
    class RadioButton: Button(), OnClickListener 
```

- `private constructor` 인스턴스해야할 필요가 없는 클래스
    - 유틸리티, 싱글턴패턴, 팩토리 패턴 등

    class Secretive private constructor() {}

### 부 생성자: 상위 클래스를 다른 방식으로 초기화

- 부생성자를 여럿 만들지 말것.
- 디폴트 값을 생성자에 명시할 것.
- `constructor` 키워드로 부생성자 명시
- `super`로 상위 클래스 생성자 호출
- 주생성자가 없다면, 부생성자는 반드시 상위 클래스를 초기화 하거나 다른 생성자를 호출해야한다.

### 부 생성자: 상위 클래스를 다른 방식으로 초기화

- 부생성자를 여럿 만들지 말것.
- 디폴트 값을 생성자에 명시할 것.
- `constructor` 키워드로 부생성자 명시
- `super`로 상위 클래스 생성자 호출
- 주생성자가 없다면, 부생성자는 반드시 상위 클래스를 초기화 하거나 다른 생성자를 호출해야한다.
- 

### 인터페이스에 선언된 프로퍼티 구현

```
    interface User {
            val nickname: String
    }
```

User의 프로퍼티를 구현하는 3가지 방법

```
    // 주생성자에 정의
    class PrivateUser(override val nickname: String) : User
    
    // 커스텀 게터로 정의
    // nickname 호출할 때 마다 게터 식 연산
    class SubscribingUser(val email: String) : User {
        override val nickname: String
            get() = email.substringBefore('@')
    }
    
    // 초기화
    // 클래스 생성할 때만 getFacebookName() 호출, 그 뒤부턴 단순 get
    class FacebookUser(val accountId: Int) : User {
        override val nickname = getFacebookName(accountId)
    }
```

### 게터와 세터에서 뒷받침하는 필드에 접근

- 뒷받침 필드 `field`
    - 커스텀 게터/세터에서 해당 필드에 접근하는 키워드

```
    class User(val name: String) {
        var address: String = "unspecified"
                    // 커스텀 세터
            set(value: String) {
                println("""
                    Address was changed for $name:
                    "$field" -> "$value".""".trimIndent())
                field = value         // 뒷받침필드 접근
            }
    }
```

### 접근자의 가시성 변경

- `get`은 `public` 이지만, `set`은 `private` 으로 하고 싶을 때

```
    class LengthCounter {
        var counter: Int = 0
            private set
    
        fun addWord(word: String) {
            counter += word.length
        }
    }
    
    fun main(args: Array<String>) {
        val lengthCounter = LengthCounter()
        lengthCounter.addWord("Hi!")
        println(lengthCounter.counter)
    }
```

## 4.3 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임

- 코틀린은 모든 클래스가 정의해야하는 toString, equals, hashCode 등을 컴파일러가 보이지 않는 곳에서 자동으로 만들어준다.

### 모든 클래스가 정의해야 하는 메소드

고객 이름과 우편번호를 저장하는 예제:
```
    class Client(val name: String, val postalCode: Int)
```

**문자열 표현: toString()**

- 클래스가 문자열로 표현될 내용 정의

```
    class Client(val name: String, val postalCode: Int) {
            override fun toString() = "Client(name=$name, postalCode=$postalCode)"
    }
```

**객체의 동등성: equals()**

- 내부에 동일한 데이터를 포함하는 경우 두 객체는 같다고 간주되어야한다.

```
    fun main(args: Array<String>) {
        val client1 = Client("Alice", 342562)
        val client2 = Client("Alice", 342562)
        println(client1 == client2)
    }
    
    // 두 객체가 같은 참조(주소)를 가지지 않기 때문에 같지 않다고 판단
    >>> false
```

- 참고: 동등성 연산에 == 연산자 사용

    primitive type(Int, Float, Long, Boolean 등): 값을 비교 

    reference type(String, Array, List, 커스텀 클래스 등): 참조된 주소값을 비교

    자바에서는 레퍼런스 타입에 == 을 사용하면 주소값을 비교하지만, ***코틀린에서는 내부에서 equals() 함수를 호출한다. 주소값을 비교하고 싶다면 ===을 사용할것!***

```
    class Client(val name: String, val postalCode: Int) {
            // 인자로 어떤 오브젝드든 받을 수 있다
        override fun equals(other: Any?): Boolean {
            if (other == null || other !is Client)     // Client 객체인지 검사
                return false
                    // 이름과 우편번호가 같으면 같은 고개으로 판단
            return name == other.name &&
                   postalCode == other.postalCode
        }
        override fun toString() = "Client(name=$name, postalCode=$postalCode)"
    }
```

**해시 컨테이너: hashCode()**

```
    val processed = hashSetOf(Client("Alice", 342562))
    println(processed.contains(Client("Alice", 342562)))
    
    >>> false
```

- JVM언어: `equals()`가 `true` 인 두 객체는 반드시 같은 `hashCode()`를 반환해야 한다
- `hashSet` 과 같은 해시 기반 객체는 두 객체의 비용을 줄이기 위해 해시코드 값이 같을 경우에만 데이터를 직접 비교한다.

```
    class Client(val name: String, val postalCode: Int) {
            ...
            override fun hashCode(): Int = name.hashCode() * 31 + postalCode
    }
```

- 이렇게 3개의 메소드를 오버라이드 해주어야 제대로 동작
- 코틀린은 이걸 어떻게 자동으로 해줄까?

### 데이터클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

```
    data class Client(val name: String, val postalCode: Int)
```

`data`키워드를 명시해주면 `toString()`, `equals()`, `hashCode()`를 자동 생성해준다!

- 주 생성자의 프로퍼티를 고려해 만들어진다.
- **주생성자가 아닌 프로퍼티**는 `equals()`, `hashCode()`의 ***고려 대상이 아니다***

**데이터 클래스와 불변성: copy() 메소드**

- 데이터 클래스의 프로퍼티는 `val`을 권장한다.
    - 해시맵의 키가 `var`이라면 키가 바뀌었을 때 컨테이너의 상태가 잘못될 수 있다.
    - 쉽게 추론 가능하다.
    - 스레드 동기화 필요성이 줄어든다.
- `val` 프로퍼티를 바꿔야만 한다면?
    - 객체 내부를 바꾸지 말고, 새로 객체를 복사해서 바꾸자!
    - 원본을 참조하는 곳에 영향을 끼치지 않음
    - 객체를 복사하면서 일부 프로퍼티만 바꿀 수 있는 `copy()`

```
        fun copy(name: String = this.name, postalCode: Int = this.postalCode) 
                    = Client(name, postalCode)
```

### 클래스 위임: by 키워드 사용

- 객체 지향 설계에서 무분별하게 상속하다보면, 하위 클래스가 상위 클래스의 가정을 무시하고 잘 못 구현, 세부 메소드가 추가되어 코드가 정상적으로 작동하지 않을 수 있다.

    → 클래스의 기본을 상속 불가능한 `final`로 정한다.

- 하지만 종종 상속이 안되는 클래스에 새로운 동작을 추가해야할 때가 있다.
- 데코레이터 패턴
    - 추가할 기능이 많은 경우 추가 기능들을 기능별로 데코레이터로 묶어 설계.
    - 여러 데코레이터들을 조합해 새로운 기능을 만들어냄
    - 데코레이터가 기존 클래스의 인터페이스를 모두 제공 (기능을 기존 클래스로 전달) 하면서 새로운 기능도 쓸 수 있게 해줌
    - 준비 코드가 많이 필요 (상속하고 오버라이드가 필요한 모든 메소드들을 재정의해야함)

```
        class DelegatingCollection<T> : Collection<T> {
                private val innerList = arrayListOf<T>()
        
                override val size: Int get() = innerList.size
                override fun isEmpty(): Boolean = innerList.isEmpty()
                ...
                // 오버라이드 다 한 후 기능 추가
        }
```

- 코틀린에서는 위임을 `by` 키워드로 지원한다.
- 원소를 추가한 횟수를 기록하는 컬렉션을 구현해보자.

```
    class CountingSet<T>(
            val innerSet: MutableCollection<T> = HashSet<T>()
    ) : MutableCollection<T> by innerSet {      // MutableCollection의 구현을 innserSet에 위임
    
        var objectsAdded = 0
    
        override fun add(element: T): Boolean {           // 오버라이드하지 않고 직접 재정의
            objectsAdded++
            return innerSet.add(element)
        }
    
        override fun addAll(c: Collection<T>): Boolean {  // 오버라이드하지 않고 직접 재정의
            objectsAdded += c.size
            return innerSet.addAll(c)
        }
    }
```

- MutableCollection에 문서화된 API를 사용하므로 잘 작동할 것임을 확신