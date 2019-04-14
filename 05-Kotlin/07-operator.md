- 연산자 오버로딩
- 관례: 여러 연산을 지원하기 위해 특별한 이름이 붙은 메소드
- 위임 프로퍼티

7장에서 계속 사용할 예제 클래스

- 2차원 상의 위치를 표현하는 `Point` 클래스

    data class Point(val x: Int, val y: Int)

## 7.1 산술 연산자 오버로딩

- 자바
    - 산술 연산에 `+`, `-`, `/`, `%` , `*` 등 사용 가능
    - String 에만 + 연산자 사용 가능

### 이항 산술 연산 오버로딩

- `fun` 앞에 `operator` 를 붙여야한다
- 확장함수로도 선언 가능 `operator fun Point.plus(other: Point)`
```kotlin
    data class Point(val x: Int, val y: Int) {
    	operator fun plus(other: Point): Point {
    		return Point(x + other.x, y + other.y)
    	}
    }
    
    val p1 = Point(10, 20)
    val p2 = Point(20, 30)
    println(p1 + p2)
    println(p1.plus(p2))
```
[오버로딩이 가능한 이항 산술 연산자](https://www.notion.so/d4d122fffe4242d9b2d6f82de909eea6)

- 오버로딩이 가능하다
    - 여러 파라미터를 받을 수 있다
- `<<` , `&` 와 같은 비트 연산자는 오버로딩할 수 없다.

### 복합 대입 연산자 오버로딩

`+=` , `-=` 와 같은 연산자
```kotlin
    operator fun <T> MutableCollection<T>.plusAssign(element: T) {
    		this.add(element)
    }
    
    val numbers = ArrayList<Int>()
    numbers += 42
```
### 단항 연산자 오버로딩

`-a` 와 같이 한 항에 대해 연산하는 연산자도 오버로딩할 수 있다
```kotlin
    operator fun Point.unaryMinus(): Point {
    	return Point(-x, -y)
    }
    
    val p = Point(10, 20)
    println(-p)
```
[Copy of 오버로딩이 가능한 이항 산술 연산자](https://www.notion.so/971843ae975744428bfcb3fb811ae705)