# 🤖 Kotlin 가이드 : 개념 및 개발

Kotlin 언어의 문법, 함수형 프로그래밍, 코루틴 등 안드로이드 개발자에게 필요한 핵심 개념 등 모든 것을 다룬 문서입니다.

---

## 📘 주요 개념

### object 키워드
- 정의
    + 클래스를 정의함과 동시에 인스턴스를 생성하는 키워드
    + 코드 간결성 확보 및 Thread Safe
    + 최초 참조 시점에 단 한번만 인스턴스가 생성됨 (lazy + synchornized) & JVM class loader가 클래스 로딩을 Thread-safe하게 보장함
- 주요 사용 케이스
    + 싱글톤 (Singleton)
        * 하나만 존재하는 객체 정의
        ```kotlin
        object Logger {
            fun log(msg: String) {
                println("Log: $msg")
            }
        }
      
        Logger.log("message") // 호출
        ```
        * 생성자 호출 필요없고 Thread-safe 함
    + 동반객체(Companion Object)
        * 클래스 내부에서 정적 멤버처럼 사용
        ```kotlin
        class User(val name: String) {
            companion object {
                fun create(name: String): User = User(name)
                var country = "Korea" 
                @JvmStatic val BASE_ADDRESS = "서울특별시" // java static 처럼 
            }
        }
      
        User.create("Alice") // 클래스 이름으로 호출 가능
        User.coutnry // Seoul
        ```
        * @JvmField, @JvmStatic 붙이면 java 에서도 static 처럼 쓸 수 있음
    + 익명객체
        * 즉석에서 정의해 사용하는 임시 객체
        ```kotlin
        val buttonClickListener = object: View.OnClickListener {    
            override fun onClick(v: View?) {
                println("Clicked!")
            }
        }
        ```
        * 인터페이스나 추상 클래스를 즉석에서 구현
        * Android 리스너 구현에 자주 사용
    + Object Declaration
        * 상속 구조나 전략 객체로 사용
        * 동일한 인터페이스를 구현한 여러 객체 중 하나를 선택해 실행 전략을 바꾸는 패턴
        ```kotlin
        interface PaymentStrategy {
            fun pay(amount: Int)
        }

        object KakaoPay : PaymentStrategy {
            override fun pay(amount: Int) {
                println("카카오페이로 $amount 원 결제")
            }
        }

        object CreditCard : PaymentStrategy {
            override fun pay(amount: Int) {
                println("신용카드로 $amount 원 결제")
            }
        }
      
        class PaymentProcessor(var strategy: PaymentStrategy) {
            fun processPayment(amount: Int) {
                strategy.pay(amount)
            }
        }

        fun main() {
            val processor = PaymentProcessor(KakaoPay)
            processor.processPayment(10_000)  // 카카오페이로 결제

            processor.strategy = CreditCard
            processor.processPayment(20_000)  // 신용카드로 결제
        }
        ```
        * PaymentProcessor는 PaymentStrategy라는 상위 타입만 알고 있음
        * 실제 객체는 KakaoPay, CreditCard 등 다양하게 대체 가능 → 런타임 다형성
        * 조건에 따라 전략을 바꾸거나, 클라이언트 코드가 객체의 구체 타입을 몰라도 사용 가능


---


### lateinit & lazy
- 정의
  + 코틀린에서 non-null 타입 프로퍼티를 나중에 초기화할 수 있도록 지원하는 두 가지 메커니즘 입니다.

- lateinit
  + 선언 :
    ```kotlin
    lateinit var myProperty : String
    ```
  + 목적 : non-null 타입 프로퍼티를 선언 시점에 바로 초기화 할 수 없을 때 사용합니다.
  + 제약사항 :
    * var 프로퍼티에만 사용할 수 있습니다.
    * Nullable 타입에는 사용할 수 없습니다.
    * 기본 자료형을 사용하거나, 박싱된 타입을 사용해야 합니다.
    * 커스텀 getter/setter를 가질 수 없습니다.
  + 주의점 : lateinit 프로퍼티는 사용하기 전에 반드시 초기화되어야 합니다. 

- lazy
  + 선언 :
    ```kotlin
    val myLazyProperty : String by lazy { ... }
    ```
  + 목적 : 프로퍼티의 초기화 비용이 크거나, 실제로 사용될 때까지 초기화를 늦추고 싶을 때 사용합니다.
          lazy는 해당 프로퍼티에 처음 접근하는 순간 단 한번만 초기화 블록을 실행하고 그 결과를 저장합니다. 이후 저장된 값을 반환합니다.
  + 제약사항 : 
    * val 프로퍼티에만 사용할 수 있습니다. 
  + 특징 : 
    * 실제 사용 시점까지 초기화를 미룹니다.
    * 초기화 블록은 최초 접근 시에만 실행됩니다.
    * 기본적으로 LazyThreadSafetyMode.SYNCHRONIZED로 동작하여 여러 스레드에서 동시에 접근해도 안전하게 한 번만 초기화됩니다.

- 사용시점
  + lateinit 사용
    * 프로퍼티가 non-null 타입이어야 하고, 생성자에서 초기화할 수 없으며, 나중에 반드시 초기화될 것이라고 확실할 때
    * 초기화 시점을 개발자가 직접 제어해야 할 때

  + lazy 사용
    * 프로퍼티가 non-null 타입이어야 하고, 최초 접근 시점에 초기화되어야 하며, 그 값이 변경되지 않는 경우
    * 초기화 비용이 커서 실제 사용될 때까지 초기화를 늦추고 싶을 때
    * 싱글톤 패턴과 같이 특정 객체가 최초 사용 시점에 한번 만 생성되도록 하고 싶을 때


---

### Safe Call / Elvis 연산자
- 코틀린에서 Safe Call 연산자(?.)와 Elvis 연산자(?:)는 null-safety를 보장하기 위해서 자주 사용되는 두 가지 핵심 연산자 입니다.

- Safe Call 연산자 (?.)
  + 역할 : 객체가 null일 경우 메서드 호출이나 프로퍼티 접근을 건너뛰고 null을 반환합니다.
  + NullPointerException을 방지할 수 있습니다.

- 문법
  ```kotlin
  val result = someObject?.someMethod()
  ```

- 예제
  ```kotlin
    data class User(val name: String?)

    val user: User? = null
    val nameLength = user?.name?.length
    println(nameLength) // 출력: null
  ```
  
- Elvis 연산자 (?:)
  + 역할 : 좌측 값이 null일 경우 우측의 기본값을 사용합니다.
  
- 문법
  ```kotlin
  val value = nullableValue ?: defaultValue
  ```

- 예제
  ```kotlin
    val user: User? = null
    val nameLength = user?.name?.length ?: 0
    println(nameLength) // 출력: 0
  ```
  
- Safe Call 연산자 + Elvis 연산자 함께 쓰는 예
  ```kotlin
    val user: User? = null
    val nameLength = user?.name?.length ?: 0
    println(nameLength) // 출력: 0
  
    // Android 에서 자주 쓰는 패턴
    // null이면 return 처리
    val token = prefs?.getString("token", null) ?: return
  ```


---


## var vs val
- 개념
  + var 변경 가능한 변수 (mutable)
  + val 변경 불가능한 변수 (immutable)
- 공통점
  + | 항목                     | 내용                                    |
    | ---------------------- | ------------------------------------- |
    | **둘 다 변수 선언에 사용**      | Kotlin에서 변수를 선언할 때 `var` 또는 `val`을 사용 |
    | **타입 추론 가능**           | `var name = "John"`처럼 타입 명시 없이 선언 가능  |
    | **null 가능성, 범위 규칙 동일** | 둘 다 `?`로 nullable 여부 설정 가능, scope는 동일 |
- 차이점
  + | 항목             | `var`                              | `val`                                                              |
    | -------------- | ---------------------------------- | ------------------------------------------------------------------ |
    | **가변성**        | 값을 변경할 수 있음                        | 최초 할당 이후 값 변경 불가                                                   |
    | **재할당**        | 가능                                 | 불가능                                                                |
    | **사용 예시**      | 루프 인덱스, 상태 변화 변수                   | 상수, 변경되지 않는 참조값                                                    |
    | **컴파일러 처리**    | 매번 수정 가능성 고려                       | 불변성 보장 → 최적화 유리                                                    |
    | **컬렉션의 내용 변경** | `val`로 선언해도 **내부 요소는 변경 가능** (주의!) | `val list = mutableListOf(1, 2)`는 리스트 요소 변경 가능, 단 `list` 자체 재할당 불가 |
- 예시
  + ```kotlin
    // var: 값 변경 가능
    var name = "Alice"
    name = "Bob" // ✅ 가능

    // val: 값 변경 불가능
    val age = 30
    // age = 31 // ❌ 컴파일 오류

    // val로 선언된 불편 컬렉션의 내부는 변경 가능
    val list = listOf(1,2,3)
    // list 에는 객체를 변경할 수 있는 메소드가 제공되지 않음
    list = listOf(4,5,6) // ❌ 재할당 불가능
    
    // val로 선언된 가변 컬렉션의 내부는 변경 가능
    val numbers = mutableListOf(1, 2, 3)
    numbers.add(4) // ✅ 가능
    // numbers = mutableListOf(5, 6) // ❌ 재할당 불가능
    ```
  + val 의 의미는 재할당이 불가능하고 가변 컬렉션의 내부 값의 변경은 가능함
- 결론
  + 기본적으로 val 을 사용하고 변수를 만들고 필요한 경우에만 var 사용하는 것이 코틀린 가이드
  + val이 많을수록 코드 안정성, 스레드 안전성 증가
  + val 사용 시 **불변객체 지향(immutability)**로 인한 버그 예방 가능
  + 안드로이드 예
    ```kotlin
    class MyViewModel : ViewModel() {
       private val _uiState = MutableLiveData<UiState>()
       val uiState: LiveData<UiState> = _uiState  // 외부에는 읽기 전용으로 노출
    }
    ```
    * val을 쓰면 ViewModel의 구조적 명확성(읽기 전용 참조)이 보장됨
    * 실제 상태 변경은 내부 로직에서만 가능하게 설계 → 클린 아키텍처, MVVM 패턴에서 권장되는 방식
    * 의도하지 않은 재할당 방지
    * 상태 추적이 쉬움 → 참조가 고정돼 디버깅, 로깅, 추적이 수월


---


### 스코프 함수(let, run with, apply, also)
- 객체의 컨텍스트 내에서 코드 블록을 실행할 수 있게 해주는 특별한 함수들 입니다.
- 스코프 함수들은 코드를 더 간결하고 읽기 쉽게 만들어주며, 주로 객체에 대한 연속적인 연산을 수행하거나, 객체의 프로퍼티에 접근하여 초기화 또는 설정 작업을 할 때 유용하게 사용 됩니다.
- 코틀린 표준 라이브러리에는 let, run, with, apply, also 다섯가지의 스코프 함수가 있습니다.

- 컨텍스트 객체 참조 방식
  + this(수신 객체) : 람다 내부에서 컨텍스트 객체를 this로 참조합니다. 객체의 멤버에 직접 접근할 수 있습니다.(run, with, apply)
  + it(람다 인자) : 람다 내부에서 컨텍스트 객체를 단일 인자 it으로 참조합니다.(let, also)

- 반환 값
  + 컨텍스트 객체 자신(apply, also) : 람다 블록을 실행한 후 컨텍스트 객체를 반환합니다. 객체에 대한 추가적인 연쇄 호출이 필요할 때 유용합니다.
  + 람다 결과(let, run, with) : 람다 블록의 마지막 표현식 결과를 반환합니다. 특정 연산의 결과를 변수에 할당하거나, 다른 연산으로 전달해야 할 때 유용합니다.

- let
  + 컨텍스트 객체 : it
  + 반환 값 : 람다 결과
  + 주요 사용 경우
    * Null이 아닌 객체에 대해서만 코드를 실행하는 경우
    * 특정 객체를 사용하여 연산한 결과를 다른 변수에 할당할 때

- run
  + 컨텍스트 객체 : this
  + 반환 값 : 람다 결과
  + 주요 사용 경우
    * 객체의 프로퍼티를 설정하고 동시에 어떤 연산의 결과를 반환해야 할 때
    * 특정 객체의 컨텍스트 내에서 여러 연산을 수행하고 그 결과를 얻고 싶을 때

- with
  + 컨텍스트 객체 : this
  + 반환 값 : 람다 결과
  + 주요 사용 경우
    * 특정 객체의 여러 함수나 프로퍼티에 접근해야 할 때
    * run과 유사하지만, with는 확장 함수가 아니므로 객체를 첫 번째 인자로 전달해야 합니다.

- apply
  + 컨택스트 객체 : this
  + 반환 값 : 컨텍스트 객체 자신
  + 주요 사용 경우
    * 객체의 프로퍼티를 설정하고 그 객체를 반환받고 싶을 때
    * 빌더 패턴과 유사하게 객체를 설정할 때 유용합니다.

- also
  + 컨텍스트 객체 : it
  + 반환 값 : 컨텍스트 객체 자신
  + 주요 사용 경우
    * 객체에 대한 부가적인 작업을 수행하고 싶을 때
    * 원래 객체를 변경하지 않고 추가적인 동작을 수행할 때
    * apply와 유사하지만, 컨텍스트 객체를 it으로 참조합니다.

- 어떤 스코프 함수를 선택해야 할까요?
  + null 검사와 함께 특정 연산을 수행하고 결과를 반환 : let
  + 객체 설정 후 객체 자체를 반환 : apply
  + 객체 설정 후 다른 결과를 반환 : run
  + 객체에 대한 부가적인 작업 후 객체 자체를 반환 : also
  + 특정 객체의 멤버에 반복적으로 접근하여 연산 후 결과를 반환 : with


---

### 타입 캐스팅 
- 정의
  + 하나의 타입을 다른 타입으로 변환하는 행위
  + 코틀린의 경우 타입 캐스팅이 **안전성 중심**으로 설계되어 있어 null or classCastException 줄이는 문법 강조

- 방식
  + | 구분          | 문법              | 설명                                    |
    | ----------- | --------------- | ------------------------------------- |
    | **강제 캐스팅**  | `as`            | 명시적 캐스팅. 실패 시 `ClassCastException` 발생 |
    | **안전한 캐스팅** | `as?`           | 실패 시 예외 대신 `null` 반환                  |
    | **스마트 캐스트** | `is` + if block | 조건문으로 타입 체크 후 자동 캐스팅                  |

- 사용법
  + as 강제 캐스팅
    * ```kotlin
      val any: Any = "Hello"
      val str: String = any as String  // ✅ 성공

      val num: Any = 123
      val str = num as String  // ❌ ClassCastException
      ```
  + as? 안전한 캐스팅
    * ```kotlin
      val any: Any = "Kotlin"
      val str: String? = any as? String  // ✅ "Kotlin"

      val num: Any = 123
      val str: String? = num as? String  // ✅ null 반환 (예외 없음)
      ```
  + is 타입 검사 (스마트 캐스트)
    * ```kotlin
      fun printLength(obj: Any) {
        if (obj is String) {
           println(obj.length)  // obj가 자동으로 String으로 캐스팅됨
        }
      }
      ```
    * is로 체크하면 블록 안에서 자동으로 스마트 캐스팅됨
    * !is는 반대 의미
    * ```kotlin
      fun handleInput(input: Any) {
        when (input) {
           is String -> println("String length: ${input.length}")
           is Int -> println("Double: ${input * 2}")
           else -> println("Unknown type")
        }
      }
      ```


---


### data class
- 데이터를 저장하기 위한 용도로 만들어진 클래스로, 데이터를 다룰 때 필요한 여러 기능을 자동으로 제공함으로써 **간결하고 효율적인 코드작성이 가능**
- 자동으로 생성되는 함수들
    - toString()
    - equals() / hashCode()
    - copy()
    - componentN()
```java
    val user1 = User("철수", 25)
    val user2 = User1.copy(age = 30)

    println(user1)  // User(name=철수, age=25)
    println(user2)  // User(name=철수, age=30)

    val (name, age) = user2
    println(name)   // 철수
    println(age)    // 30
```
- 제약사항
    - primary constructor에 최소 하나 이상의 파라미터가 있어야 함
    - open, abstract, sealed, inner 클래스로 선언할 수 없음
    - 상속은 불가능 (final로 선언 됨)


---


### enum 열거형 클래스
- 정의
  > 미리 정의된 고정된 상수 집합을 표현할 때 사용하는 열거형 클래스

- 기본 사용법
  + ```kotlin
    enum class Direction {
        NORTH, SOUTH, EAST, WEST
    }
    
    val dir = Direction.NORTH
    print(dir.name) // "NORTH"
    print(dir.ordinal) // 0
    ```
  + `name`: 문자열
  + `ordinal`: 순서(0부터 시작)
    
- 예제
  + 생성자와 프로퍼티 사용
    * ```kotlin
      enum class HttpsStatus(val code: Int, val desc: String) {
        OK(200, "Success"),
        NOT_FOUND(404, "Not Found"),
        SERVER_ERROR(500, "Internal Server Error")
      }
      
      val status = HttpsStatus.NOT_FOUND
      println(status.code) // 404
      println(status.desc) // Not Found
      ```
  + 함수 정의도 가능
    * ```kotlin
      enum class Operation {
        PLUS {
           override fun apply(a: Int, b: Int) = a + b
        },
        MINUS {
           override fun apply(a: Int, b: Int) = a - b
        };

        abstract fun apply(a: Int, b: Int): Int
      }
      
      Operation.PLUS.apply(5,3).also { println(it) } // 8
      Operation.MINUS.apply(5,3).also { println(it) } // 2
      ```
    * enum 상수마다 서로 달느 동작 구현 가능
    * abstract 함수 -> 각 상수 오버라이드
  + when 과 함께 쓰임
    * ```kotlin
      fun handleDirection(dir: Direction) = when(dir) {
        Direction.NORTH -> "Going up"
        Direction.SOUTH -> "Going down"
        Direction.EAST  -> "Going right"
        Direction.WEST  -> "Going left"
      }
      ```
    * 코틀린의 when 은 enum을 exhaustive(가능한 모든 경우를 빠짐없이 다룸) 하게 다룰 수 있어 모든 enum 을 다 처리하면 else 없어도 됨
  + values() / valueOf()
    * ```kotlin
      enum class Direction {
        NORTH, SOUTH, EAST, WEST
      }
      
      Direction.values().forEach { println(it) } // 전체 enum 출력
      Direction.valueOf("EAST").also { println(it) } // 문자열 -> enum
      ```
  
- enum vs sealed class
  + | 항목        | enum class           | sealed class                  |
    | --------- | -------------------- | ----------------------------- |
    | 목적        | 고정된 상수 집합            | 계층적인 타입 제한                    |
    | 런타임 확장    | ❌ 불가능 (상수 고정)        | ✅ 하위 클래스 자유롭게 정의 가능           |
    | 인스턴스 속성   | 제한적 (상수당 고정 생성자)     | 유연하게 정의 가능                    |
    | 상태 표현     | 간단한 상태 (예: 방향, 상태코드) | 복잡한 구조적 상태 표현에 유리             |
    | `when` 사용 | 자동 exhaustive 체크     | exhaustive 하려면 else 필요할 수도 있음 |

- 면접 관련 질문
  + enum 대신 sealed class 대신 사용하는 건 언제 적절한지?
    * 상태나 타입이 미리 정의된 값으로 고정되어 있고, 로직이 간단할 때 enum 이 적절
    * 복잡한 상태 표현은 sealed class 가 적절
    * enum 동종의 데이터나 상태 묶을 때 적합
    * sealed class 이종의 타입들을 하나의 계층으로 통합


---


### 확장 함수(Extension Function)
- 정의
  + 확장 함수는 기존 클래스를 상속하거나 Decorator와 같은 디자인 패턴을 사용하지 않고도 클래스에 새로운 기능을 추가할 수 있게 해주는 강력한 기능입니다.
  + 마치 원래 그 클래스에 있던 메서드처럼 호출할 수 있습니다.

- 기본 개념
  + 확장 함수를 선언하려면, 추가하고자 하는 함수의 이름 앞에 수신 객체 타입을 명시합니다.
  + 수신 객체 타입은 함수를 확장하고자 하는 클래스 또는 인터페이스의 이름 입니다.

- 주요 특징 및 장점
  + 코드 간결성 및 가독성 향상
    * 유틸리티 함수들을 특정 클래스와 관련된 것처럼 보이게 하여 코드의 가독성을 높입니다.
  + 기존 클래스 수정 없이 기능 확장
    * Java 표준 라이브러리 클래스나 서드파티 라이브러리의 클래스와 같이 직접 수정할 수 없는 클래스에도 새로운 기능을 추가할 수 있습니다.
  + 정잭 디스패치
    * 확장 함수는 정적으로 디스패치됩니다. 즉, 호출되는 함수는 변수의 컴파일 타임 타입에 의해 결정되며, 런타임 타입에 의해 결정되지 않습니다.
    * 이는 오버라이딩과 다르게 동작합니다. 만약 클래스에 멤버 함수와 동일한 시그니처를 가진 확장 함수가 있다면, 항상 멤버 함수가 우선적으로 호출 됩니다.
  + Null 수신 객체 처리 기능
    * 확장 함수는 수신 객체가 null일 가능성을 염두에 두고 정의할 수 있습니다.
  + 스코프 제한
    * 확장 함수는 일반 함수와 마찬가지로 특정 스코프 내에서만 유효하게 만들 수 있습니다.
    * 파일 최상단에 선언하거나, 클래스 또는 함수 내부에 선언할 수 있습니다.
    * 다른 패키지에서 사용하려면 import 해야 합니다.

- 확장 함수 vs 멤버 함수
  + 캡슐화 : 확장 함수는 클래스 외부에서 정의되므로, 클래스의 private 또는 protected 멤버에는 접근할 수 없습니다. 멤버함수는 클래스의 모든 멤버에 접근 가능 합니다.
  + 오버라이딩 : 멤버 함수는 서브클래스에서 오버라이드될 수 있지만, 확장 함수는 정적으로 디스패치 되므로 오버라이드 개념이 적용되지 않습니다.
  + 확장 함수 사용 시점
    * 기존 클래스에 유틸리티성 기능을 추가하고 싶을 때
    * 특정 타입에 대한 헬퍼 함수를 정의하여 코드의 가독성을 높이고 싶을 때
    * 클래스의 핵심 API의 일부는 아니지만, 해당 클래스의 객체를 다룰 때 자주 사용되는 부가적인 기능을 제공하고 싶을 때
  + 멤버 함수 사용 시점
    * 클래스의 핵심적인 기능을 구현할 때
    * 클래스의 내부 상태에 접근해야 할 때
    * 다형성을 활용하여 하위 클래스에서 동작을 변경할 수 있도록 하고 싶을 때

- 주의사항
  + 확장 함수는 실제로 클래스의 코드를 변경하는 것이 아닙니다. 단지 컴파일러가 해당 함수를 정적 메서드 호출로 변환해주는 syntactic sugar 입니다.
  + 너무 많은 확장 함수를 남용하면 코드를 이해하기 어려워질 수 있으므로, 적절한 상황에서 명확한 목적으로 사용하는 것이 좋습니다.
  + 확장 함수는 코틀린의 유연한 기능 중 하나로 API를 더 깔끔하고 사용하기 쉽게 만드는 데 큰 도움을 줍니다.

- 면접 관련 질문
  + 확장 함수와 유틸 클래스의 차이점은?
    * 전통적인 Java 스타일 유틸 클래스는 Utils.method() 형태이지만 코틀린의 확장함수는 obj.method() 형태로 사용 가능하며 가독성이 좋고, IDE 자동 완성도 더 자연스럽게 동작합니다.
  + 확장 함수 남용 시 단점은?
    * 정적 바인딩이기 때문에 다형성을 기대하면 안 됩니다.
    * 프로젝트 전체에 퍼지면 어디서 정의했는지 추적이 어려워질 수 있습니다.
    * 같은 이름의 확장 함수가 여러 모듈에 정의되면 충돌 가능성도 있습니다.


---


### sealed class
- 상속 가능한 클래스를 같은 **파일 내에서만 정의하도록 제한**한 클래스
- **컴파일러가 모든 하위 타입을 알고있게 되어**, when 문에서 **타입 분기 처리 안전하게 가능**
- 열거형(enum)보다 더 유연한 상태 표현
- when 문 사용할 때 else 없이도 모든 타입 처리 가능
- 불변 상태 모델링에 탁월 (ex) Success, Error, Loading 상태표현)
```kotlin
    sealed class Result
    
    data class Success (val data : String) : Result()
    data class Error (val message : String) : Result()
    object Loading : Result()
```
이 후 when 문에서 사용 방법 :
```kotlin
    fun handle (result : Result) {
        when (result) {
            is Success -> println("성공 : ${result.data}")
            is Error -> println("에러 : ${result.message}")
            is Loading -> println("로딩 중")
        }
    }
```

- 특징
    + | 항목                | 설명                                 |
                |-------------------|------------------------------------|
      | 상속 가능 클래스 제한      | 하위 클래스는 반드시 같은 파일 내에서 정의해야 함       |
      | 추상 클래스처럼 사용       | 직접 인스턴스화 불가능                       |
      | `when`구문 최적화    | 컴파일러가 하위 타입을 모두 인식하므로 `else` 생략 가능 |
      | `enum`보다 확장성 좋음 | 하위 클래스마다 **다른 속성, 로직** 가능          |

  + | 상황                                              | 추천 이유                     |
                    |-------------------------------------------------|---------------------------|
    | API 응답의 상태 표현 (`Success`, `Failure`, `Loading`) | 각 상태별 데이터가 다를 때 유용        |
    | UI 상태 표현 (ex) `LoggedIn`, `LoggedOut`)          | 명확한 타입 기반 상태 관리           |
    | 복잡한 조건 분기                                       | `when`과 함께 쓰면 안정정 + 가독성 향상 |


---


### 구조 분해 선언(Destructuring Declarations)
- 정의
  + 객체의 프로퍼티들을 여러 변수에 한 번에 할당할 수 있게 해주는 편리한 문법입니다.

- 기본 개념
  + ```kotlin
      val (변수1, 변수2, ...) = 객체
      ```
  + 객체의 특정 프로퍼티들이 순서대로 변수1, 변수2 등에 할당 됩니다.

- 동작 방식
  + 구조 분해 선언이 동작하기 위해서는 해당 객체가 특저 ㅇ규약을 따라야합니다.
  + 컴파일러는 componentN()이라는 이름의 연산자 함수를 찾아서 호출합니다.
    * 첫 번째 변수에는 component1() 함수의 반환 값이 할당됩니다.
    * 두 번째 변수에는 component2() 함수의 반환 값이 할당됩니다.
    * 이런 식으로 componentN() 함수가 순서대로 호출됩니다.

- 주요 사용 사례
  + Data Class : 데이터 클래스는 주 생성자에 선언된 프로퍼티에 대해 자동으로 componentN() 함수를 생성해줍니다.
  + ```kotlin
    data class User(val name: String, val age: Int)

    fun main() {
        val user = User("Alice", 30)
        val (name, age) = user // user.component1()은 name에, user.component2()는 age에 할당

        println("Name: $name, Age: $age") // 출력: Name: Alice, Age: 30
    }
    ```

  + Map.Entry : 맵을 반복할 때 키와 값을 한 번에 가져오는 데 유용합니다.
  + ```kotlin
    fun main() {
        val map = mapOf("A" to 1, "B" to 2, "C" to 3)

        for ((key, value) in map) { // map의 각 Entry에 대해 구조 분해 선언 사용
            println("Key: $key, Value: $value")
        }
        // 출력:
        // Key: A, Value: 1
        // Key: B, Value: 2
        // Key: C, Value: 3
    }
    ```

  + 컬렉션과 배열 : 리스트나 배열과 같은 컬렉션도 componentN() 함수를 통해 구조 분해를 지원합니다.
  + ```kotlin
    fun main() {
        val list = listOf("Apple", "Banana", "Cherry")
        val (first, second) = list // list[0]은 first에, list[1]은 second에 할당

        println("First: $first, Second: $second") // 출력: First: Apple, Second: Banana
        // val (a, b, c, d) = list // Error: Index out of bounds, list에는 3개의 요소만 있음
    }
    ```

  + 함수에서 여러 값 반환 : 함수가 여러 값을 반환해야 할 때, 데이터 클래스나 Pair, Triple과 같은 클래스를 반환 타입으로 사용하고, 호출부에서 구조 분해 선언을 통해 각 값을 쉽게 받을 수 있습니다.
  + ```kotlin
    data class Result(val value: Int, val status: String)

    fun process(): Result {
        // ... 어떤 처리 ...
        return Result(42, "Success")
    }

    fun main() {
        val (resultValue, resultStatus) = process()
        println("Value: $resultValue, Status: $resultStatus") // 출력: Value: 42, Status: Success
    }
    ```

  + 필요 없는 값 무시하기 : 구조 분해 선언시 특정 위치의 값이 필요 없다면 밑줄(_)을 사용하여 해당 변수를 무시할 수 있습니다.
  + ```kotlin
    data class Point(val x: Int, val y: Int, val z: Int)

    fun main() {
      val point = Point(10, 20, 30)
      val (x, _, z) = point // y 좌표는 무시
  
      println("X: $x, Z: $z") // 출력: X: 10, Z: 30
    }
    ```

  + 람다 파라메터에서의 구조 분해 : 람다 표현식의 파라메터에도 구조 분해 선언을 사용할 수 있습니다.
  + ```kotlin
    fun main() {
      val userList = listOf(User("Bob", 25), User("Charlie", 35))

      // 람다 파라미터 (name, age)에 User 객체를 구조 분해
      userList.forEach { (name, age) ->
          println("$name is $age years old")
      }
      // 출력:
      // Bob is 25 years old
      // Charlie is 35 years old

      val map = mapOf(1 to "one", 2 to "two")
      map.forEach { (key, value) -> // Map.Entry를 key와 value로 구조 분해
          println("$key -> $value")
      }
      // 출력:
      // 1 -> one
      // 2 -> two
    }
    ```
    
- 질문
  + 구조 분해 선언시 componentN() 함수는 어떤 역할을 하나요?
    * 구조 분해 선언시 내부적으로 호출되는 메서드 입니다. 
      예를 들어 val (x, y) = point는 point.component1()과 point.component2()를 호출합니다. 이는 컴파일러가 자동으로 변환해주는 형태입니다.
  + 구조 분해 선언이 성능에 영향을 줄 수 있나요?
    * 일반적으로 구조 분해는 간단한 메서드 호출로 이루어지기 때문에 큰 성능 부담은 없습니다.
      하지만 복잡한 클래스에서 많은 componentN()을 구현하거나 무분별하게 사용할 경우 가독성이나 성능에 영향을 줄 수 있으니 주의가 필요합니다. 


---


### inline
- 정의
  + 함수를 호출할 때 실제로 함수 호출 코드가 생성되는 것이 아니라, 함수 본문이 호출 지점에 "그대로 복사되는" 방식으로 동작하는 함수
  + 즉, "함수 호출 = 코드 복사 붙여넣기" → 성능 향상, 오버헤드 감소

- 필요한 이유?
  + 고차 함수 호출 시 함수 객체와 람다 객체가 생성되기 때문
  + inline은 이 객체 생성을 없애고 성능을 향상
  + 또한 람다 내부에서 return 사용 가능 등 코드 흐름 유연화

- 사용법
  + ```kotlin
    inline fun log(block: () -> Unit) {
      println("== Start ==")
      block()
      println("== End ==")
    }

    fun main() {
      log {
         println("Hello!")
      }
    }
    
    // 컴파일 시 (개념상)
    fun main() {
       println("== Start ==")
       println("Hello!")
       println("== End ==")
    }  
    ```
  + log() 함수는 호출이 아니라 코드로 펼쳐져 들어감 (인라인화)

- inline 효과
  + | 항목             | 설명                              |
    | -------------- | ------------------------------- |
    | ✅ 성능           | 함수 호출 비용 제거 (특히 람다 객체 생성 안 함)   |
    | ✅ return 흐름 제어 | 람다 내부에서 `return` 가능             |
    | ❌ 코드 크기        | 너무 많이 쓰면 바이너리 커짐 (주의)           |
    | ✅ 제네릭 람다       | `inline` 덕분에 람다에서 reified 사용 가능 |

- 보조 키워드 : noinline, crossinline
  + noinline : 인라인 안하고 객체로 넘기고 싶을 때
    * ```kotlin
      inline fun test(a: () -> Unit, noinline b: () -> Unit) {
        a() // 인라인됨
        b() // 인라인되지 않음
      }
      ```
    * 람다 여러 개 중 일부만 인라인하고 싶을 때 유용
  + crossinline : 람다 안에서 non-local return 못하게 막기
    * ```kotlin
      inline fun doSomething(crossinline action: () -> Unit) {
         Thread {
            action() // 여기서 return 하면 컴파일 에러
         }.start()
      }
      ```
    * 다른 스레드/콜백에서 실행될 람다에는 return 쓰면 위험하므로 금지

- 실무에서 자주 쓰이는 패턴
  + measureTimeMillis { ... }
  + runBlocking { ... } 
  + reified 타입 캐스팅 (ex. inline fun <reified T> ...)
    * ```kotlin
      inline fun <reified T> Gson.fromJson(json: String): T { 
         return this.fromJson(json, T::class.java)
      }
      ```
    * 일반 함수에서는 제네릭 타입 T는 런타임에 소멸되지만, inline + reified 덕분에 런타임에도 타입 정보를 유지 가능

- 면접 관련 질문
  + inline 함수 장점
    * 함수 호출 오버헤드를 줄이고, 람다 객체 생성을 방지해서 성능을 향상
  + noinline 이 필요한가?
    * 람다 중 일부만 인라인하고 싶거나, 해당 람다를 다른 곳에 전달해야 할 경우 사용
  + reified 는 왜 inline 함수에서만 사용할 수 있는가?
    * Kotlin의 제네릭은 기본적으로 타입 소거되지만, inline 함수는 컴파일 시 타입을 알고 있으므로 reified로 타입 유지가 가능
    * > reified 는 제네릭 타입 정보를 런타임에도 유지할 수 있게 해주는 키워드 (단 inline 함수에서만 사용 가능)




---



### Kotlin & Java
- 두 언어 모두 JVM에서 실행되는 언어, 안드로이드 개발이나 서버 개발에서 많이 쓰임 

- 공통점
  + | 항목                 | 설명                                     |
          |--------------------|----------------------------------------|
      | JVM 기반             | 둘 다 **Java Virtual Machine**에서 실행 됨    |
      | Java 라이브러리 사용 가능   | Kotlin은 Java의 모든 API, 라이브러리를 그대로 사용 가능 |
      | 클래스, 객체, 상속        | 객체지향 언어로서 기본 구조 동일                     |
      | 쓰레드, 네트워크, 파일 IO 등 | 기본 기능 거의 동일                            |
      | 안드로이드 개발 가능                   | 둘 다 안드로이드 공식 지원 언어               |

- 차이점
  + | 항목                 | Java                             | Kotlin                                |
              |--------------------|----------------------------------|---------------------------------------|
    | 문법 간결성             | 비교적 장황함                          | 매우 간결함 (`val`, `when`, `dataclass`)   |
    | Null 안정성           | `NullPointerException` 위험 존재     | 컴파일 단계에서 **null** 체크 지원 (`?`, `?:`)   |
    | 데이터 클래스            | 직접 `equals`, `hashCode` 등 작성해야 함 | `data class`로 자동 생성                   |
    | 함수형 프로그래밍          | 람다 지원은 있지만 불편                    | 고차 함수, 람다, `map/filter` 등 자연스러움       |
    | 확장 함수              | 없음                               | 기존 클래스에 함수 추가 가능 (`String.isEmail()`) |
    | 스마트 캐스팅            | 수동 형변환 필요                        | `is` 체크 후 자동 캐스팅                      |
    | 기본 자료형 (Primitive) | `int`, `double` 등 존재             | 모두 객체 타입으로 통합 (`Int`, `double`)       |
    | 코루틴 지원             | 없음 (외부 라이브러리 필요)                 | 코루틴으로 비동기 쉽게 처리 가능                    |
    | Null 처리            | 런타임에서 터짐                         | `?`, `!!`, `?:` 로 컴파일 타임에서 경구         |

- 예시
  1. 변수 선언
     - java
     ```java
        String name = "철수";
     ```
     - kotlin
     ```kotlin
      val name = "철수"   // 자동 타입 추론
     ```
  2. Null 처리
     - java
     ```java
      if (user != null) {
        System.out.println(user.getName());  
      }
     ```
     - kotlin
     ```kotlin
        println(user?.name ?: "이름 없음")
     ```
  3. 데이터 클래스
    - java
     ```java
      public class User {
        String name;
        int age;
      }
     ```
    - kotlin
     ```kotlin
        data class User (val name : String, val age : Int)
     ```


---


### 상속 제어 (open, final, abstract)
- 정의
  + 코틀린에서 상속 제어 키워드는 클래스나 함수, 프로퍼티가 상속/재정의 가능한지 명확하게 제어하는 기능
  + Java 와 기본값이 다르며 코틀린의 안정성과 설계 철학을 보여주는 부분

- 기본 개념 정리
  + | 키워드        | 의미                             | 기본값 여부            |
    | ---------- | ------------------------------ | ----------------- |
    | `final`    | **상속 불가**, 오버라이드 금지            | ✅ Kotlin의 기본값     |
    | `open`     | **상속 가능**, 오버라이드 허용            | ❌ 명시적으로 선언해야 함    |
    | `abstract` | 추상 멤버 또는 클래스, **무조건 오버라이드 필요** | ❌ 추상 클래스에서만 사용 가능 |

- 상세 설명
  + final
    * 클래스, 메서드, 프로퍼티 기본적으로 `final`
    * 오버라이드/상속 하려면 명시적으로 `open` or `abstract` 로 풀어야 함
    * ```kotlin
      class Animal {
         fun sound() { println("Animal sound") } // final이 기본
      }
      ```
  + open
    * 해당 클래스 or 멤버를 상속/재정의 가능하게 만듦
    * ```kotlin
      open class Animal {
         open fun sound() { println("Animal sound") }
      }

      class Dog : Animal() {
         override fun sound() { println("Bark!") }
      }
      ```
  + abstract
    * 클래스 또는 멤버가 추상적이며 하위 클래스에서 반드시 구현해야 함
    * abstract class 인스턴스화 불가

- 면접 관련 질문
  + 추상 클래스와 인터페이스의 차이는?
    * abstract class는 상태(필드)와 구현 메서드를 가질 수 있습니다.
    * interface는 다중 구현이 가능하며, 일부 구현만 제공할 수 있습니다. (Kotlin에서는 interface도 default method 가능)
  + data class 에 open or final 키워드 사용가능한가?
    * data class 기본 적으로 final 이며 open 붙여서 상속 가능하게 만들 수 없음. 붙일 경우 컴파일 에러 발생
    * 자동 생성되는 메서드 (equals(), hashCode(), copy(), toString() 등) 상속 허용 시 예상치 못한 동작이나 버그 발생 가능성 높아짐
    * 데이터 클래스는 단일 데이터 컨테이너로서의 목적에 맞춰 설계됨. 객체지향적 확장보다는 `값 기반 비교(value equality)`가 핵심.
    * 상속이 필요하다면 ? 일반 클래스 또는 composition 으로 써야 함


---


### Range & Progression
- 개요
    + 코틀린에서 Range(범위)와 Progression(수열)은 특정 타입의 값들이 순서대로 나열된 시퀀스를 표현하는 강력하고 유용한 개념입니다.
    + 주로 숫자 타입이나 문자 타입과 함께 사용되며, 반복문, 조건문 등 다양한 상황에서 코드를 간결하고 읽기 쉽게 만들어줍니다.

- Range
    + 개요
        * 닫힌 구간 또는 반 열린 구간으로 정의되는 값들의 집합입니다. 즉, 시작 값과 끝 값을 가지며, 이 두 값 사이의 모든 값을 포함합니다.
    + 생성 방법
        * 표준 라이브러리 kotlin.ranges 패키지에서 제공하는 함수들을 사용하여 Range를 생성할 수 있습니다.
        * .. 연산자 (rangeTo() 함수) : 닫힌 구간을 만듭니다. 끝 값을 포함합니다.
      ```kotlin
        val numbers = 1..4       // 1, 2, 3, 4를 포함하는 IntRange
        val chars = 'a'..'d'     // 'a', 'b', 'c', 'd'를 포함하는 CharRange
      ```
        * ..< 연산자 (rangeUntil() 함수) : 반 열린 구간을 만듭니다. 끝 값을 포함하지 않습니다.
      ```kotlin
        val numbersUntil = 1..<4  // 1, 2, 3을 포함하는 IntRange (4는 미포함)
        val charsUntil = 'a'..<'d'   // 'a', 'b', 'c'를 포함하는 CharRange ('d'는 미포함)
      ```
    + 주요 특징 및 사용법
        * 타입 : 생성되는 Range의 타입은 시작 값과 끝 값의 타입에 따라 결정됩니다.
        * in 연산자와 함께 사용 : 특정 값이 범위 내에 포함되는지 확인하는 데 매우 유용합니다.
      ```kotlin
        val numbers = 1..4       // 1, 2, 3, 4를 포함하는 IntRange
        val chars = 'a'..'d'     // 'a', 'b', 'c', 'd'를 포함하는 CharRange
      ```

- Progression
    + 개요
        * 시작 값, 끝 값, 그리고 증가분을 가집니다.
    + 생성 방법
        * step 변경자 : Range 뒤에 step을 사용하여 증가분을 지정합니다.
      ```kotlin
        val evenNumbers = 0..10 step 2 // 0, 2, 4, 6, 8, 10
        println(evenNumbers.toList())   // [0, 2, 4, 6, 8, 10]
  
        val countdown = 10..0 step 3 // 10, 7, 4, 1
        println(countdown.toList())  // [10, 7, 4, 1] (downTo와 함께 사용하는 것이 더 일반적)
      ```
        * downTo 함수 : 값을 감소시키는 Progression을 만듭니다.
      ```kotlin
        val decreasingNumbers = 5 downTo 1 // 5, 4, 3, 2, 1
        println(decreasingNumbers.toList()) // [5, 4, 3, 2, 1]
  
        val decreasingWithStep = 10 downTo 0 step 2 // 10, 8, 6, 4, 2, 0
        println(decreasingWithStep.toList())       // [10, 8, 6, 4, 2, 0]
      ```

    + 주요 특징 및 사용법
        * 구성 요소 : first, last, step
        * 반복문과 함께 사용 : Progression은 Iterable을 구현하므로 for 루프에서 직접 사용할 수 있습니다.
      ```kotlin
        for (i in 1..5) {
          print("$i ") // 1 2 3 4 5
        }
        println()
  
        for (i in 10 downTo 0 step 3) {
            print("$i ") // 10 7 4 1
        }
        println()
      ```
        * reversed() : Progression의 순서를 뒤집을 수 있습니다.
      ```kotlin
        val numbers = 1..5
        for (i in numbers.reversed()) {
            print("$i ") // 5 4 3 2 1
        }
        println()
      ```
      


---



### 접근제어자
1. public (기본값, 어디서나 접근 가능)
    ```kotlin
        // 파일 : Car.kt
        package com.example.car
   
        class Car {         // public 생략 가능
            fun drive() {
                println("Car is driving")
            }
        }    
   
        // 파일 : Main.kt
        import com.example.car.Car

        fun main() {
            val car = Car()
            car.drive()     // ✅ 접근가능 : public 이라 어디서나 사용 가능
        }
    ```
    - 기본 접근 제어자
    - 다른 패키지/모듈 에서도 접근 가능

2. internal (같은 모듈 내에서만 접근 가능)
    ```kotlin
        // 파일 : Engine.kt
        internal class Engine {
            fun start() {
                println("Engine started")
            }        
        }
        
        // 파일 : Main.kt (같은 모듈 내)
        fun main() {
            val engine = Engine()
            engine.start()      // ✅ 같은 모듈이므로 접근 가능
        }    
    ```
    ```kotlin
        // ❌ 다른 모듈에서
        import com.example.engine.Engine    // ❌ 컴파일 에러
   
        fun main() {
            val engine = Engine()   // ❌ 접근 불가 : internal은 다른 모듈에서 사용 불가
        }
    ```
    - **같은 Gradle 모듈 내**에서만 사용 가능 
    - 외부 SDK로 내보낼 필요 없는 클래스/함수에 적합

3. protected (상속 받은 클래스에서만 접근 가능)
    ```kotlin
        open class Animal {
            protected fun breathe() {
                println("Animal breathing")
            }
    
            fun walk() {
                println("Animal walking")
            }
        }
   
        class Dog : Animal() {
            fun sniff() {
                breathe()   // ✅ protected 이므로 하위 클래스에서 사용 가능
            }
        }    
    
        fun main() {
            val dog = Dog()     
            dog.walk()      // ✅ public method
            // dog.breathe()    // ❌ 외부에서는 접근 불가
        }    
    ```
    - 외부 접근 ❌
    - **상속받은 클래스 내부에서만 접근 가능**
    - Java의 `protected`와 거의 동일 

4. private (같은 클래스 또는 파일 내에서만 접근 가능)
    1. 클래스 내부 제한
        ```kotlin
            class SecretBox {
                private val code = "1234"
       
                private fun unlock() {
                    println("Box unlocked with $code")
                }
       
                fun tryUnlock() {
                    unlock()    // ✅ 클래스 내부이므로 접근 가능
                }
            }
       
            fun main() {
                val box = SerectBox()
                box.tryUnlock()         // ✅ 가능
                // box.unlock()         // ❌ 외부 접근 불가
                // println(box.code)    // ❌ 외부 접근 불가
            }
        ```
    2. 파일 내부 제한 (Top-level)
        ```kotlin
            // 파일 : Utils.kt
            private fun internalHelper() {
                println("Used only within this file")
            }
       
            fun publicApi() {
                internalHelper()    // ✅ 같은 파일 내
            }
        ```
        ```kotlin
            // 파일 : AnotherFile.kt
       
            // internal Helper()    ❌ 호출 불가 - 다른 파일에서는 private 함수 접근 안됨
        ```
        - 클래스의 구현 세부사항 숨길 때 사용
        - 파일 수준에서는 **다른 파일에서 접근 불가**
- 요약정리
    + | 접근 제어자      | 클래스 외부 접근  | 상속 관계 접근 | 같은 파일           | 같은 모듈 | 다른 모듈 |
                                |-------------|------------|----------|-----------------|-------|-------|
      | `public`    | ✅ |  ✅        | ✅               |   ✅    |    ✅   |
      | `internal`  | ✅ |     ✅     | ✅               |    ✅   |    ❌   |
      | `protected` | ❌ |    ✅      | ✅ (클래스 내)       |   ✅    |     ❌  |
      | `private`   | ❌ |  ❌        | ✅ (같은 클래스 / 파일) |   ❌    |    ❌   |

- 실전
    - `internal` : 앱 내부 전용 유틸, 헬퍼, Repository, ViewModel 등 노출 필요 없는 경우
    - `protected` : BaseViewModel, BaseAdapter 등에서 상속 구조 설계 시 유용
    - `private` : 캡슐화 (Encapsulation)를 위해 적극 사용 ()불필요한 노출 방지

- 질문
    + kotlin 접근제어자에는 어떤것들이 있나요?
        * public : 기본값 (어디서나 접근 가능)
        * internal : 같은 모듈 내에서만 접근 가능
        * protected : 상속받은 클래스 내에서만 접근 가능
        * private : 같은 클래스 또는 파일 내에서만 접근 가능
    + Java와 Kotlin의 접근제어자 차이점은?
        * kotlin에는 `default` 접근제어가 없음 -> 대신 internal 사용
        * kotlin의 `internal`은 모듈 기준 (Java에 없음)
        * kotlin에서 top-level 함수클래스도 `private`/`internal` 사용 가능
    + 접근제어자를 왜 명시적으로 설정해야 하나요?
        * 외부로 노출하지 않아야 할 정보를 보호 (캡슐화)
        * Api의 명확한 경계를 정의
        * 유지보수, 재사용성 향상


---


### reified & 제네릭
- 정의
  + reified
    * `구체화된` 컴파일된 코드 안에서 타입 T가 지워지지 않고 유지됨을 의미
    * 코틀린 제네릭 타입 파라미터를 런타임에 사용할 수 있게 해주는 키워드
    * `inline` 함수의 타입 파라미터에서만 사용 가능하며 
    * 일반적으로 불가능한 T::class, T::class.java, is T, as T 같은 표현 사용 가능
  + 제네릭
    * 클래스나 함수가 타입에 의존하지 않고 재사용성 있게 동작할 수 있게 해줌
    * 제네릭은 런타임 시점 타입 정보가 지워짐. 이를 타입 소거라고 부름 (T::class, T::class.java 같은 작업 불가)
    * ```kotlin
      fun <T> printItem(item: T) {
        println(item)
      }
      ```
    * 문제점은 일반 제네릭은 런타임 타입을 알 수 없음
    * ```kotlin
      fun <T> getTypeClass(): Class<T> {
        return T::class.java // compile error cannot use 'T' as reified type parameter
      }
      
      inline fun <reified T> getTypeClass(): Class<T> {
        return T::class.java
      }
      ```

- 활용 방안
  + JSON 파싱 with Gson
    * ```kotlin
      inline fun <reified T> parseJson(json: String): T {
        return Gson().fromJson(json, T::class.java)
      }
      
      val user: User = parseJson(jsonString)
      ```
    * reified 없으면 TypeToken<T>() 같이 우회하는 복잡한 코드 필요
  + 타입 검사
    * ```kotlin
      inline fun <reified T> isOfType(value: Any): Boolean {
        return value is T
      }
      
      val result = isOfType<String>("Hello") // true
      ```
  + ViewModel Factory in Android
    * ```kotlin
      inline fun <reified VM : ViewModel> createViewModel(): VM {
        return ViewModelProvider(this).get(VM::class.java)
      }
      ```
      
- 정리
  + | 상황                             | `reified` 필요 여부 |
    | ------------------------------ | --------------- |
    | 런타임에 타입 검사 (`is`, `as`)        | ✅ 필요            |
    | `T::class`, `T::class.java` 사용 | ✅ 필요            |
    | 단순 타입 재사용, 타입 전달 X             | ❌ 불필요           |
  + 제네릭은 컴파일 시점에만 타입 정보를 가짐 → 런타임엔 지워짐
  + reified는 이 타입 정보를 런타임까지 유지하게 함
  + inline 함수에서만 사용 가능
  + 안드로이드에서 Gson, ViewModel, Retrofit 등과 함께 자주 활용됨

- 면접 관련 질문
  + reified 키워드 언제 왜 사용?
    * reified는 제네릭 타입의 런타임 타입 정보를 유지하고 싶을 때 사용
    * 코틀린의 일반 제네릭은 타입 소거 때문에 런타임에 T::class, is T 같은 연산이 불가능
    * reified를 사용하면 이런 제약을 우회할 수 있음 (단, reified는 inline 함수에서만)
    * 예로 Gson 파싱, 타입 필터링, ViewModelFactory 로 뷰 모델 생성 시
  + 왜 reified는 inline 함수에서만 사용?
    * reified는 타입 정보를 런타임까지 보존해야 하므로, 컴파일 시점에 해당 타입이 코드에 구체적으로 삽입 
    * inline 함수는 컴파일 시점에 함수 본문이 호출된 위치에 직접 복사되기에 제네릭 타입도 실제 타입으로 치환
    * 이 구조 덕분에 T::class, is T 같은 코드가 가능해짐
    * 일반 함수는 이 치환이 일어나지 않기 때문에 reified를 사용 불가


---


### typealias
- 정의
  + typealias는 기준 타입에 새로운 이름을 부여하는 기능입니다.

- 기본 사용법
  + typealias 키워드를 사용하여 새로운 이름과 기존 타입을 연결합니다.
  + ```kotlin
    typealias Name = String
    typealias UserList = List<User>
    typealias ClickListener = (View) -> Unit
    ```
    
- typealias를 사용하는 이유 및 장점
  + 가독성향상 : 복잡하거나 긴 타입 이름을 의미 있는 이름으로 대체하여 코드를 더 쉽게 이해할 수 있도록 합니다.

- 복잡한 제네릭 타입 간소화
  + 제네릭을 사용하는 복잡한 타입을 더 짧고 관리하기 쉬운 이름으로 만들 수 있습니다.
  + ```kotlin
    // typealias 사용 전
    val complexFunction: (List<Map<String, Set<Int>>>, (String) -> Boolean) -> List<String> = { data, predicate ->
    // ...
    emptyList()
    }

    // typealias 사용 후
    typealias DataFilter = (String) -> Boolean
    typealias ComplexData = List<Map<String, Set<Int>>>
    typealias DataProcessor = (ComplexData, DataFilter) -> List<String>

    val simpleFunction: DataProcessor = { data, predicate ->
    // ...
    emptyList()
    }
    ```
    
- 함수 타입 명명
  + 특히 콜백 함수나 고차함수에서 함수 타입을 명확한 이름으로 정의하여 코드의 의도를 더 잘 전달할 수 있습니다. 

- typealias의 특징 및 주의사항
  + 새로운 타입을 만드는 것이 아님 : typealias는 단순히 기존 타입에 다른 이름을 붙이는 것입니다.
  + 생성자를 가질 수 없음 : typealias는 타입의 별칭일 뿐이므로, 자체적인 생성자를 가질 수 없습니다.
  + 상속 제어 불가 : typealias는 클래스가 아니므로, open, final 등의 상속 제어 키워드를 사용할 수 없습니다.
  + 제네릭 타입에도 사용 가능
    ```kotlin
    typealias StringList<T> = List<T> // T는 여전히 제네릭 파라미터
    val names: StringList<String> = listOf("Alice", "Bob")

    // 특정 타입으로 고정할 수도 있음
    typealias IntList = List<Int>
    val numbers: IntList = listOf(1, 2, 3)
    ```
  + 내부 클래스 및 객체에도 사용 가능
    ```kotlin
    class Outer {
        inner class Inner
        object NestedObject
    }

    typealias InnerClass = Outer.Inner
    typealias NestedObj = Outer.NestedObject
    ```
    
- 언제 typealias를 사용하면 좋을까?
  + 코드베이스 전체에서 반복적으로 사용되는 복잡한 타입 시그니처가 있을 때
  + 함수 타입을 매개변수나 반환 타입으로 자주 사용할 때
  + 특정 도메인에 맞는 의미 있는 타입 이름을 부여하고 싶을 때
  + 가독성을 높여 코드 ㅇ지보수를 용이하게 하고 싶을 때


---



### 고차 함수
- **"함수를 인자로 받거나, 함수를 반환하는 함수"**
  ```kotlin
        fun orderFunction (action : () -> Unit) {
            action()    // 전달받은 함수를 실행 
        }
  ```
  - 함수가 함수를 매개변수로 받거나 함수를 반환 = 고차함수

- 사용이유
  + | 장점                           | 설명                               |
              |------------------------------|----------------------------------|
    | ✅ 코드 재사용성 향상                 | 중복 제거, 로직 추상화                    |
    | ✅ 간결한 문법                     | 람다(lamda)와 함께 쓰면 코드가 짧아짐         |
    | ✅ 커스터마이징 용이                  | 특정 동작을 매개변수로 전달 가능 (e.g, 클릭 핸들러) |
    | ✅ Rx/Flow/Coroutine 에도 기반이 됨 | 고차함수 없이는 이들 API 불가능              |

- 고차함수 사용 시 이슈 및 해결책
  + | 주의점      | 설명                                          | 해결책                                       |
                  |----------|---------------------------------------------|-------------------------------------------|
    | 성능 이슈    | 람다나 함수 객체가 힙에 할당 됨                          | `inline` 키워드 사용                           |
    | 메모리 누수   | 람다가 외부 context를 캡처하면 누수 발생 가능 -> 특히 Android | `weak reference`, `lifecycle-aware 처리` 필요 |
    | 디버깅 어려움  | 람다 표현식 중첩 시 추적이 힘듦                          | 함수로 분리해서 명확하게 관리                          |

- 고차함수 구성 요소
  + | 요소      | 설명                                  | 예시                                              |
                      |---------|-------------------------------------|-------------------------------------------------|
    | 함수 타입   | `(Int, Int) -> Int`, `() -> Unit` 등 | `val sum : (Int, Int) -> Int = {a, b -> a + b}` |
    | 함수 파라미터 | 함수를 인자로 전달                          | `fun excute (action : () -> Unit)`              |
    | 함수 반환값  | 함수 자체를 리턴                           | `fun getOperator() : (Int. Int) -> Int`         |

- Kotlin에서 흔히 쓰는 고차함수 예시
  + | 함수                            | 역활      | 설명                              |
                            |-------------------------------|---------|---------------------------------|
    | `map`                         | 변환      | `list.map {it * 2}`               |
    | `filter`                      | 조건 걸러내기 | `list.filter {it > 0}`            |
    | `forEach`                     | 반복      | `list.forEach {println(it)}`      |
    | `run`, `let`, `apply`, `also` | 스코프 함수  | DSL 스타일 코드 구성                   |
    | `setOnClickListener`          | 이벤트 핸들러 | `button.setOnClickListener {...}` |

- 함수 타입 예시
    1. 함수를 인자로 받는 고차함수
       ```kotlin
            fun caculator (x : Int, y : Int, op : (Int, Int) -> Int) : Int {
                return op(x, y)
            }
       
            val result = calculator(5, 3) { a, b -> a + b }
            println(result) // 8
       ```
    2. 함수를 반환하는 고차함수
       ```kotlin
            fun getMulti(factor : Int) : (Int) -> Int {
                return { number -> number * factor }
            }
            
            val triple = getMulti(3)
            println(triple(10)) // 30
       ```
       
- 고차함수 + 람다
    ```kotlin
        fun doSomeThing (action : () -> Unit) {
            println("고차 함수 시작!")
            action()
            println("고차 함수 끝!")
        }
  
        doSomeThing {
            println("람다 실행 됨!")
        } 
  
        // 결과
        // 고차 함수 시작!
        // 람다 실행 됨!
        // 고차 함수 끝!
    ```
  
- 고차함수는 어떨때 사용?
    1. 콜백 함수 구현할 때
        ```kotlin
            fun requestPermission (onGranted : () -> Unit, onDenied : () -> Unit) {
                val granted = true  // 예시
                if (granted) onGranted() else onDenied
            }
        ```
        ```kotlin
            requestPermission(
                onGranted = { println("권한 허용됨") },
                onDenied = { println("권한 거부됨") }
            )    
        ```
       - 사용자가 뭘 할지 외부에서 정해서 전달

    2. 리스트 처리할 때
        ```kotlin
            val names = listOf ("Alice", "Bob", "Charlie")
            
            val upper = names.map { it.uppercase() }
            val filtered = names.filter { it.length <= 4 }
            
            println(upper)      // [ALICE, BOB, CHARLIE]
            println(filtered)   // [Bob]
        ```
        - `map`, `filter` 등은 모두 고차함수
        - 내부적으로 **"이 리스트에 어떤 작업을 해줄까?"** 를 전달받음

    3. 이벤트 처리할 때 (click 등)    
        ```kotlin
            button.setOnClickListener {
                println("버튼 클릭 됨!")     // 이게 바로 람다 -> 고차 함수에 전달되는 것
            }
        ```

- 면접 질문
  + 고차함수란 무엇인가요?
    * 고차함수는 함수를 매개변수로 받거나, 함수를 반환하는 함수입니다.
  + 고차함수를 사용하는 이유는 무엇인가요?
    * 중복제거 및 추상화 : 특정 작업을 외부에서 주입받아 유연하게 처리 가능
    * 콜백 처리 : 네트워크 응답, 클릭 이벤트, 권한 요청 등에 자주 사용
    * List 처리 (map/filter 등) : 컬렉션을 함수형 스타일로 처리 가능
    * 코드 가독성/재사용성 향상
  + inline 함수는 고차함수와 어떤 관계가 있나요?
    * 고차함수는 기본적으로 람다 객체를 생성하기 떄문에 성능 비용이 발생하는데, `inline` 키워드를 붙히면 컴파일시 코드를 직접 삽입 하므로, 성능 최적화 가능
  + 고차함수를 사용할 때 주의해야 할 점은?
    * 람다에서 **context (ex) Activity)**를 캡처하면 메모리 누수 발생이 가능
    * 복잡한 람다 중첩 -> 가독성이 저하
    * 인자가 많으면 오히려 유지보수 어려움



---




### mutable & immutable
- var / val
  1. var (mutable variable)
     + 값을 변경할 수 있음
     + **읽기/쓰기 모두 가능**
     ```kotlin
        var age = 30
        age = 31    // ✅ 가능
     ```
  
  2. val (immutable variable)
     + **한번 초기화되면 값을 변경할 수 없음**
     + 읽기 전용 변수(Read-only)
     + 불변 객체를 참조하거나, 가변 객체를 참조할 수 있지만 참조 자체는 바꿀수 없음
     ```kotlin
        val name = "ABC"
        // name = "New" ❌ 컴파일 오류 발생
     ```

- 컬렉션
  1. List, Set, Map (immutable)
     + 읽기 전용 / 변경 불가능
  2. MutableList, MutableSet, MutableMap (mutable)
     + 요소 추가, 삭제 등 기능
     ```kotlin
      val immutableList = listOf("A", "B", "C")
      // immutableList.add("D") ❌ 불가능
     
      val mutableList = mutableListOf("A", "B", "C")
      mutableList.add("D") // ✅ 가능
     ```
     + val mutableList = mutableListOf(...)에서 mutableList 자체는 변경 불가지만, 내부의 데이터는 변경 가능 (리스트는 mutable)
     + 컬렉션을 불변하게 유지하면 예측 가능하고 안전한 코드 작성이 쉬워짐

- 클래스의 불변성 (immutable Object)
  - kotlin에서는 **데이터 클래스를 불변**하게 설계하는 것을 권장
  ```kotlin
    data class User (val name: String, val age: Int)
    // 모든 프로퍼티가 val로 되어 있어, 인스턴스를 만든 후 값을 변경할 수 없음.
  
    val user = User("Heeju", 30)
    // user.name = "New" ❌ 불가능
  ```
  ```kotlin
    // 프로퍼티를 var로 하여 mutable 객체가 됨
    data class MutableUser (var name: String, var age: Int)
  ```
  
- immutable이 더 좋은 이유
  + | 장점                       | 설명                             |
                  |--------------------------|--------------------------------|
    | ❄️ 예측 가능성   | 상태가 변하지 않아 디버깅, 테스트가 쉬움        |
    | 🔐 스레드 안전성   | 멀티스레드 환경에서 동기화 문제 감소           |
    | 🧠 가독성    | 함수형 프로그래밍 스타일과 잘 어울림           |
    | ♻️ 참조 공유 가능    | 값이 변하지 않기 때문에 안전하게 여러곳에서 공유 가능 |

- 질문
  + val과 var의 차이점이 무엇인가요?
    * val은 읽기전용 변수이며 var은 값이 변경 가능한 변수입니다.
  + kotlin의 List와 MutableList의 차이점이 무엇인가요?
    * List는 읽기전용 컬렉션이며 MutableList는 읽기, 수정이 가능한 컬렉션 입니다.
  + val로 선언된 MutableList의 내용은 변경할 수 있을까요?
    * val로 선언된 리스트는 참조 자체를 바꿀 순 없지만, 내부 요소를 수정가능합니다
    * ex) val list = mutableListOf(1, 2, 3)에서 list.add(4)는 가능
   


---


### init 블록 & constructor()
- `init` 블록
  + 클래스 생성될 때 실행되는 초기화 블록
  + 클래스에 하나 이상 작성 가능하며, 생성자(constructor)보다 먼저 실행됨
  + 주로 검증, 초기 로그, 계산 작업에 사용
  + ```kotlin
    class User(val name: String) {        
        init {
            println("init name is $name")
        }
    }
    ```
    
- `constructor()`
  + 코틀린 클래스는 주 생성자(primary constructor) 와 보조 성성자(secondary constructor) 가질 수 있음
  + 주 생성자는 클래스 헤더에 정의되고, 필드 초기화는 `init 블록`에서 수행 가능
  + 보조 생성자는 `constructor` 키워드 명시적으로 사용
  + ```kotlin
    class User(val name: String) {
        constructor(name: String, age: Int) : this(name) {
            println("constructor name is $name age is $age")
        }
    }
    ```
    
- 차이점
  + | 항목    | `init` 블록       | `constructor()`         |
    | ----- | --------------- | ----------------------- |
    | 실행 시점 | 객체 생성 시 자동 실행   | 명시적으로 호출 필요             |
    | 목적    | 공통 초기화, 검증 로직   | 다양한 생성 방법 제공            |
    | 개수    | 여러 개 가능         | 여러 개 가능 (오버로드)          |
    | 호출 순서 | 생성자 → `init` 순서 | 보조 생성자 → 주 생성자 → `init` |

- 샘플 코드
  + ```kotlin
    class Book(val title: String) {
        init {
            println("책 제목: $title") // init 블록
        }

        constructor(title: String, author: String) : this(title) {
            println("저자: $author") // 보조 생성자
        }

        constructor(title: String, author: String, price: Int) : this(title) {
            println("저자: $author , 가격: $price") // 보조 생성자
        }
    }
    
    val book = Book("해리포터")
    val book1 = Book("해리포터2", "롤링2")
    val book2 = Book("해리포터3", "롤링3", 10000)    
    
    /* 출력 결과
    책 제목: 해리포터
    책 제목: 해리포터2
    저자: 롤링2
    책 제목: 해리포터3
    저자: 롤링3 , 가격: 10000
    */
    ```

- 결론
  + | 개념              | 핵심 역할                      |
    | --------------- | -------------------------- |
    | `init`          | 생성자 호출 후 **공통 초기화** 수행     |
    | `constructor()` | 다양한 생성 경로를 제공하는 **보조 생성자** |
  + init은 `공통 초기화 블록`, constructor()는 `오버로드용 생성자` 서로 보완하는 구조로 쓰인다.

- 면접 관련 질문
  + init 블록과 생성자(constructor)의 차이점은?
  + init 블록이 보조 생성자보다 먼저 실행되는가?
    * 보조 생성자 시작 -> : this(...) 주 생성자 호출 -> 주 생성자의 필드 초기화 + init 블록 실행 -> 보조 생성자의 나머지 블록 실행
    * 보조 생성자가 먼저 호출 되지만 코드 실행은 init 블록이 먼저 실행이 됨
  + 생성자 파라미터를 초기화 외에 검증하고 싶다면 어디?
    * 검증, 로깅, 조건 분기 같은 로직은 init 블록에 넣는 것이 적절
    * 생성자에는 기본값 할당만 하고, 로직은 init에서 분리하는 게 가독성과 유지보수 좋음


---


### Iterable, Sequence
- 코틀린에서 컬렉션 데이터를 다룰 때 Iterable과 Sequence는 중요한 역할을 합니다.
- 두 인터페이스 모두 요소의 시퀀스를 나타내지만, 데이터 처리 방식과 성능 특성에서 큰 차이를 보입니다.

- Iterable
  + 코틀린 표준 라이브러리에서 가장 기본적인 컬렉션 인터페이스 중 하나입니다.
  + 요소들을 반복(iterate) 할 수 있는 능력을 제공합니다. List, Set, Map 등 대부분 코틀린 컬렉션 타입이 Iterable을 구현합니다
  + 특징
    * 반복자(Iterator) 제공 : Iterable 인터페이스는 iterator() 메서드를 정의하며, 이 메서드는 Iterator<T> 객체를 반환합니다.
    * 이 반복자를 통해 컬렉션의 요소를 순차적으로 접근할 수 있습니다.
    ```kotlin
    interface Iterable<out T> {
        operator fun iterator(): Iterator<T>
    }

    interface Iterator<out T> {
        operator fun next(): T
        operator fun hasNext(): Boolean
    }
    ```
    * 다양한 확장 함수 : Iterable에는 map, filter, foreEach, find, groupBy 등과 같이 데이터를 변환하고 조작하는 수 많은 유용한 확장 함수가 정의되어 있습니다.
    * 즉시 계산 / 중간 컬렉션 생성 : Iterable에 대한 대부분의 연산은 즉시 계싼됩니다. 즉, 각 연산이 호출될 때마다 새로운 중간 컬렉션이 생성됩니다.
    ```kotlin
    val numbersList: List<Int> = listOf(1, 2, 3, 4, 5)

    // Iterable의 확장 함수 사용
    val doubledAndEven = numbersList
        .map { it * 2 }       // [2, 4, 6, 8, 10] - 중간 리스트 생성
        .filter { it % 2 == 0 } // [2, 4, 6, 8, 10] - 또 다른 중간 리스트 생성 (이 경우엔 동일)

    println(doubledAndEven) // 출력: [2, 4, 6, 8, 10]

    // for 루프를 통한 반복 (내부적으로 iterator 사용)
    for (number in numbersList) {
        println(number)
    }
    ```
  + 장점
    * 구현이 간단하고 직관적입니다.
    * 작은 크기의 컬렉션이나 연산의 수가 적을 때는 충분히 효율적입니다.
    * 결과 컬렉션이 즉시 필요할 때 유용합니다.
  + 단점
    * 큰 데이터셋이나 여러 단계의 연산 체인에서는 성능 저하가 발생할 수 있습니다.
    * 각 단계마다 새로운 중간 컬렉션을 생성하기 때문에 메모리 사용량과 처리 시간이 늘어날 수 있습니다.

- Sequence
  + Sequence<T>는 Iterable과 유사하게 요소의 시퀀스를 나타내지만, 지연 계산을 통해 데이터를 처리하도록 설계되었습니다.
  + 즉, 실제 연산은 최종 결과가 필요할 때까지 지연됩니다.
  + 특징
    * 지연 계산 : Sequence의 가장 큰 특징입니다. 중간 연산은 즉시 실행되지 않고, 최종 연산이 호출될 때까지 연산 계획만 세워둡니다.
    * 요소별 처리 : 최종 연산이 호출되면, Sequence는 각 요소를 개별적으로 전체 연산 파이프라인에 통과시킵니다. 첫 번째 요소가 map -> filter -> 과정을 거치고, 그 다음 두번째 요소가 같은 과정을 거칩니다.
    * 중간 컬렉션 생성 최소화 : 지연 계산과 요소별 처리 덕분에, Sequence는 중간 단계에서 불필요한 컬렉션을 생성하지 않습니다. 이는 특히 큰 데이터셋을 다룰 때 메모리 효율성과 성능 향상에 크게 기여합니다.
    ```kotlin
    val numbersSequence: Sequence<Int> = sequenceOf(1, 2, 3, 4, 5)
    // 또는 기존 컬렉션을 Sequence로 변환
    // val numbersSequence = listOf(1, 2, 3, 4, 5).asSequence()

    println("Sequence 연산 시작 전")

    val resultSequence = numbersSequence
        .map {
            println("map: $it")
            it * 2
        }
        .filter {
            println("filter: $it")
            it % 2 == 0
        }

    println("Sequence 연산 정의 완료 (아직 실행 안 됨)")

    // 최종 연산 호출 시 실제 연산 시작
    val firstResult = resultSequence.first() // 첫 번째 요소만 필요
    println("첫 번째 결과: $firstResult")

    println("\ntoList() 호출 시 전체 연산")
    val fullResultList = resultSequence.toList() // 모든 요소를 처리하여 리스트로 변환
    println("전체 결과 리스트: $fullResultList")
    ```
  + 장점
    * 큰 데이터셋 처리 시 성능 향상: 중간 컬렉션을 생성하지 않으므로 메모리 사용량이 적고, 불필요한 연산을 줄일 수 있습니다.
    * 무한 시퀀스(Infinite sequences) 처리 가능: generateSequence 등을 사용하여 무한한 요소를 가진 시퀀스를 정의하고, take와 같은 연산으로 필요한 만큼만 잘라내어 사용할 수 있습니다.
    * 효율적인 연산 순서: 각 요소가 전체 파이프라인을 통과하므로, 특정 조건에서 일찍 연산을 중단할 수 있습니다
  + 단점
    * 작은 컬렉션이나 간단한 연산에서는 Iterable보다 약간의 오버헤드가 있을 수 있습니다.
    * 지연 계산의 특성상 디버깅이 조금 더 복잡할 수 있습니다. 

- Iterable vs. Sequence: 언제 무엇을 사용할까?
  + | 특징         | Iterable (예: List, Set)                  | Sequence                                                | 
    |------------|------------------------------------------|---------------------------------------------------------|
    | 계산 방식      | 즉시 계산 (Eager)                            | 지연 계산 (Lazy)                                            | 
    | 중간 컬렉션 생성  | 각 연산마다 생성                                | 최종 연산 시까지 생성 안 함 (최소화)                                  |
    | 요소 처리 순서   | 컬렉션 전체에 대해 단계별로 처리 (map 전체 -> filter 전체) | 각 요소별로 전체 파이프라인 처리 (요소1: map->filter, 요소2: map->filter) |
    | 주요 사용 사례   | 작은 컬렉션, 간단한 연산, 결과가 즉시 필요할 때             | 큰 컬렉션, 여러 단계의 복잡한 연산, 무한 시퀀스, 성능 최적화 필요 시               |
    | 성능 (큰 데이터) | 상대적으로 낮음                                 | 상대적으로 높음                                                | 


---


### DSL (Domain-Specific Language)
- 정의
  + 특정 목적(domain)에 맞게 설계된 **맞춤형 언어 스타일 코드**를 Kotlin 문법 위에서 구현할 수 있는 기능
  + 대표적인 예로 `Gradle Kotlin DSL`, `Jetpack Compose`, `Anko`, `HTML DSL` 등이 있습니다.
  + Kotlin 문법을 활용해서 **내 도메인에 특화된 코드를 자연어처럼 작성**할 수 있게 해줌

- 구성 핵심 요소
  + | 요소                       | 설명                      |
    | ------------------------ | ----------------------- |
    | **Lambda with Receiver** | 람다 내부에서 객체(this)에 접근 가능 |
    | **Extension Function**   | 기존 타입에 새로운 함수 추가 가능     |
    | **Function Literals**    | 함수 자체를 값처럼 전달           |
    | **Named Arguments**      | 매개변수 이름을 코드 안에서 명확하게 표현 |

- 샘플
  + HTML DSL
    * ```kotlin
      fun html(block: HtmlBuilder.() -> Unit): String {
        val builder = HtmlBuilder()
        builder.block()
        return builder.build()
      }
      
      class HtmlBuilder {
        private val content = StringBuilder()

        fun body(block: BodyBuilder.() -> Unit) {
            content.append("<body>")
            val body = BodyBuilder()
            body.block()
            content.append(body.build())
            content.append("</body>")
        }
      
        fun build(): String = content.toString()
      }

      class BodyBuilder {
        private val content = StringBuilder()

        fun p(text: String) {
            content.append("<p>$text</p>")
        }
    
        fun build(): String = content.toString()
      }

        // 사용
        val result = html {
            body {
                p("Hello, Kotlin DSL!")
                p("This is a paragraph.")
            }
        }
        println(result) // <body><p>Hello, Kotlin DSL!</p><p>This is a paragraph.</p></body>
      ```
  + Gradle
    * ```kotlin
      plugins {
        kotlin("jvm") version "1.9.0"
        application
      }

      repositories {
        mavenCentral()
      }

      dependencies {
        implementation("com.squareup.retrofit2:retrofit:2.9.0")
        testImplementation("org.jetbrains.kotlin:kotlin-test")
      }

      application {
        mainClass.set("com.example.MainKt")
      }
      ```
      plugins {}, repositories {}, dependencies {} 전부 DSL
      * 중괄호 안은 this 수신 객체를 활용하는 람다 블록
  + SQL
    * JetBrains 의 Exposed 는 코틀린 DSL 로 SQL 작성할 수 있게 해주는 QRM 쿼리 빌더
    * ```kotlin
      object Users : Table() { 
        val id = integer("id").autoIncrement()
        val name = varchar("name", 50)
        override val primaryKey = PrimaryKey(id)
      }
      
      fun insertUser(name: String) {
            transaction {
                Users.insert {
                    it[Users.name] = name
                }
            }
        }

        fun getAllUsers(): List<String> {
            return transaction {
                Users.selectAll().map { it[Users.name] }
            }
        }
      ```
    * SQL 문법을 함수로 래핑한 DSL
    * insert {}, selectAll().map {} 등이 고차함수 + 람다 수신 기반
    * 타입 안정성 있는 쿼리 작성 가능
  + Jetpack Compose
    * ```kotlin
      fun GreetingScreen(name: String) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(text = "Hello, $name!")
            Button(onClick = { /* handle click */ }) {
            Text("Click Me")
            }
        }
      }
      ```
    * Column {}, Button {}, Text() 전부 DSL 스타일 컴포저블 함수
    * XML 없이 UI를 함수로 구성
    * 중첩 가능, 선언적, 가독성 높음

- 실전 DSL 활용 사례
  + | 사례                | 설명                    |
    | ----------------- | --------------------- |
    | Gradle Kotlin DSL | build.gradle.kts 파일   |
    | Jetpack Compose   | UI 선언을 함수로 구현         |
    | kotlinx.html      | HTML 빌더 DSL           |
    | SQL DSL           | Exposed 같은 라이브러리      |
    | Android UI DSL    | Anko (JetBrains, 구버전) |

- 면접 관련 질문 
  + Q1. Kotlin DSL은 어떤 상황에서 사용하는 게 적절하다고 생각하나요?
    * Kotlin DSL은 특정 도메인에 맞는 구조화된 코드를 더 선언적이고 읽기 쉽게 표현하고 싶을 때 사용하면 좋습니다.
      예를 들어 Gradle 빌드 스크립트, Jetpack Compose UI, SQL 쿼리 등과 같이 구조가 반복되거나 계층적인 데이터를 다룰 때 DSL이 효과적입니다.
      또한, DSL은 특정 API 사용 방식을 제한하거나 직관적으로 만들고 싶을 때도 유용합니다.
  + Q2. Jetpack Compose는 어떻게 DSL로 작동하나요?
    * Jetpack Compose는 Kotlin의 함수형 DSL 문법을 기반으로 설계된 UI 프레임워크입니다.
      각 UI 요소는 @Composable로 표시된 함수이며, 내부적으로는 수신 객체를 가진 람다(lambda with receiver)를 활용해 중첩된 UI 구조를 만들 수 있습니다.
      예를 들어 Column { Text(...) }는 ColumnScope.() -> Unit 형태의 DSL 블록을 받는 구조로, 코드의 가독성과 유지보수성이 좋아집니다.
  + Q3. build.gradle.kts vs build.gradle
    * | 항목       | `build.gradle` (Groovy DSL) | `build.gradle.kts` (Kotlin DSL) |
      | -------- | --------------------------- | ------------------------------- |
      | 언어       | Groovy                      | Kotlin                          |
      | 정적 타입 검사 | ❌ (런타임 에러 가능)               | ✅ (컴파일 타임 타입 체크 가능)             |
      | 코드 완성    | 제한적, IDE 자동완성 잘 안 됨         | 훨씬 우수함 (IntelliJ 완전 지원)         |
      | 문법 유연성   | 더 관대함 (익숙한 Gradle 스타일)      | 문법 더 엄격함, 타입 명확히 필요             |
      | 학습 곡선    | 초반 진입 쉬움                    | Kotlin 익숙하면 훨씬 강력하지만 초반 헷갈림     |
      | 유지보수     | 동적 타입으로 인해 오류 찾기 어려움        | 타입 안정성으로 유지보수 유리                |