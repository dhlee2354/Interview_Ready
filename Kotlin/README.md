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
    * > reified 는 ㅔ제릭 타입 정보를 런타임에도 유지할 수 있게 해주는 키워드 (단 inline 함수에서만 사용 가능)




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