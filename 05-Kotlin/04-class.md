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