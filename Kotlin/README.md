# 🤖 Kotlin 가이드 : 개념 및 개발

Kotlin 언어의 문법, 함수형 프로그래밍, 코루틴 등 안드로이드 개발자에게 필요한 핵심 개념 등 모든 것을 다룬 문서입니다.

---

## 📘 주요 개념

### object 키워드
- 정의
  + 클래스를 정의함과 동시에 인스턴스를 생성하는 키워드
  + 싱글톤 패턴을 간결하게 구현할 수 있게 해주며 코드 간결성 확보 및 Thread Safe
  + 최초 참조 시점에 단 한번만 인스턴스가 생성됨 (lazy + synchornized)
  + JVM class loader가 클래스 로딩을 Thread-safe하게 보장하기에 락 없이 안전
  
- 주요 사용 케이스
  + 싱글톤 (Singleton)
    * 하나만 존재하는 객체 정의
    * 생성자 호출 필요없고 Thread-safe 함
    * ```kotlin
      object Logger {
        fun log(msg: String) {
            println("Log: $msg")
        }
      }
      
      Logger.log("message") // 호출
      ```
  + 동반객체(Companion Object)
    * 클래스 내부에서 정적 멤버(static memeber)처럼 사용
    * @JvmField, @JvmStatic 붙이면 Java와의 상호운용성 을 더 쉽게 할 수 있다.
    * ```kotlin  
      class User(val name: String) {
          companion object {
              fun create(name: String): User = User(name)
              var country = "Korea" 

              @JvmStatic val BASE_ADDRESS = "서울특별시" // java static 처럼 
          }
      }
    
      User.create("Alice") // 클래스 이름으로 호출 가능
      User.BASE_ADDRESS // 서울특별시 java static 처럼 사용 가능
      ```
  + 익명객체
    * 일회성으로 인터페이스 또는 추상 클래스를 구현할 때 사용
    * Android 리스너 구현에 자주 사용
    ```kotlin
    val buttonClickListener = object: View.OnClickListener {    
        override fun onClick(v: View?) {
            println("Clicked!")
        }
    }
    ```
  + Object Declaration
    * 상속 구조나 전략 객체로 사용
    * 동일한 인터페이스를 구현한 여러 객체 중 하나를 선택해 실행 전략을 바꾸는 패턴
    * PaymentProcessor는 PaymentStrategy라는 상위 타입만 알고 있음
    * 실제 객체는 KakaoPay, CreditCard 등 다양하게 대체 가능 → 런타임 다형성
    * 조건에 따라 전략을 바꾸거나, 클라이언트 코드가 객체의 구체 타입을 몰라도 사용 가능
    * 전략 객체는 런타임에 교체 가능하며, 객체 간의 의존성 분리 + 다형성 확보가 가능
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

- 기타
  + `object`는 생성자가 없으므로 생성자 매개변수는 가질 수 없다.
  + 내부적으로 `final`, `static`, `thread-safe singleton`으로 컴파일 됨

- 면접 예상 질문 & 답변
  + Q1. `object`는 어떻게 thread-safe한 Singleton을 보장하나요?
    * `object`는 JVM의 class loading 단계에서 한 번만 초기화되며 이 과정은 JVM 자체가 thread-safe 보장하므로 별도의 락이나 synchronized 키워드 없이 안전함. (Lazy initialization + Classloader Locking)
  + Q2. `object`와 `companion object`의 차이는 무엇인가요?
    * `object`는 클래스 외부에서 싱글톤 객체를 정의할 때 사용하며, `companion object`는 클래스 내부에서 정적 멤버처럼 동작하게 하기 위해 사용. 둘 다 싱글톤이지만 클래스 내부인지 외부인지의 차이가 있음
  + Q3. `object`와 Java의 `static` 키워드는 어떤 차이가 있나요?
    * Java의 `static` 클래스 로딩 시점에 메모리에 올라가는 반면 코틀린의 `object`는 최초 참조 시점에 lazy 하게 초기화 됨. 또한, Kotlin은 함수, 필드, 클래스 등에서 정적 멤버를 만들기 위해 `companion object` + `@JvmStatic` 활용


---


### lateinit & lazy
- 정의
  + 코틀린에서 non-null 타입 프로퍼티를 나중에 초기화할 수 있도록 지원하는 두 가지 메커니즘
  + `lateinit` & `lazy` 2가지 방식 제공

- lateinit
  + 선언
    ```kotlin
    lateinit var myProperty : String
    ```
  + 목적
    * 선언 시 즉시 초기화할 수 없는 non-null var 프로퍼티를 나중에 명확히 초기화할 필요 있을 때 사용
  + 제약사항
    * var 프로퍼티에만 사용할 수 있습니다.
    * Nullable 타입에는 사용할 수 없습니다.
    * 기본 자료형(Int, Boolean 등) 사용할 수 없음 (박싱된 타입 가능)\
    * 커스텀 getter/setter를 불가
    * 클래스 내부에서만 사용 가능 (top-levl 사용 불가)
  + 주의점
    * lateinit 프로퍼티는 사용하기 전에 반드시 초기화 필요
    * ::myProperty.isInitialized 로 체크 가능

- lazy
  + 선언
    ```kotlin
    val myLazyProperty : String by lazy { 
      println("초기화 수행") 
      "Hello Lazy" 
    }
    ```
  + 목적
    * 초기화 비용이 큰 객체를 최초 사용 시점까지 늦추고 싶을 때
    * 객체가 한 번만 초기화되고 이후 절대 변경되지 않아야 할 때
  + 제약사항
    * val 프로퍼티에만 사용 가능
    * 초기화 블록은 단 한 번만 실행
  + 특징
    * 필요에 따라 `PUBLICATION`, `NON`E 등의 스레드 안전 모드로 변경 가능
    * 초기화 블록은 최초 접근 시에만 실행
    * 기본적으로 LazyThreadSafetyMode.SYNCHRONIZED로 동작하기에 멀티스레드 환경에서도 안전하게 한 번만 초기화 됨

- 비교
  + | 조건                            | 추천 방식      | 설명                          |
    | ----------------------------- | ---------- | --------------------------- |
    | 생성자에서 초기화 불가능, 나중에 수동으로 넣어야 함 | `lateinit` | 주로 의존성 주입(DI), 뷰 바인딩 등에서 사용 |
    | 초기화 비용이 크고, 최초 접근 시 자동 초기화    | `lazy`     | 싱글톤, 캐시, 설정값 로딩 등에서 적합      |
    | 값이 변경될 수 있음                   | `lateinit` | `var` 기반                    |
    | 값이 한 번만 설정되고, 불변              | `lazy`     | `val` 기반                    |

- 면접 관련 질문
  + `lateinit`과 `lazy`는 각각 어떤 상황에 쓰이나요?
    * lateinit은 초기화 시점을 개발자가 직접 제어해야 하는 경우 사용하며 주로 DI, Android view 바인딩 등에서 유용
    * lazy는 초기화가 자동으로 이루어지고 한 번만 실행되며, 초기화 비용이 큰 객체나 싱글톤 객체에 적합
  + `lateinit` 사용 시 주의할 점은?
    * 초기화 전에 접근하면 UninitializedPropertyAccessException 예외가 발생
    * val이나 기본 타입에는 사용할 수 없고, 반드시 non-null 참조 타입 + var 사용
  + `lazy`의 스레드 안전성은 어떻게 보장되나요?
    * 기본적으로 lazy는 LazyThreadSafetyMode.SYNCHRONIZED 모드로 작동함. 이는 여러 스레드에서 동시에 접근하더라도 초기화 블록이 한 번만 실행되도록 보장함. 필요에 따라 다른 모드로 변경할 수 있음 (PUBLICATION, NONE)


---


### Safe Call / Elvis 연산자
- 개념/정의
  + NPE(Null Pointer Exception) 방지하기 위해 nullable 타입과 null-safe 연산자 지원
  + `?.` safe call 과 `?:` Elvis 연산자 2가지를 주로 사용

- Safe Call 연산자 (?.)
  + 객체가 null 아닐 경우에만 메서드나 프로퍼티에 접근
  + 객체가 null 이면 연산을 건너뛰고 결과로 null 반환
  + NPE(Null Pointer Exception) 방지

- 사용법
  + ```kotlin
    val result = someObject?.someMethod()

    data class User(val name: String?)

    val user: User? = null
    val nameLength = user?.name?.length
    println(nameLength) // 출력: null
    ```
  
- Elvis 연산자 (?:)
  + 좌변이 null 일 경우 우변 값 사용
  + 기본값, 예외처리, 빠른 리턴 등에 활용
  
- 사용법
  + ```kotlin
    val value = nullableValue ?: defaultValue

    // Safe Call 연산자 + Elvis 연산자 함께 쓰는 예
    val user: User? = null
    val nameLength = user?.name?.length ?: 0
    println(nameLength) // 출력: 0
  
    // Android 에서 자주 쓰는 패턴
    // null이면 return 처리
    val token = prefs?.getString("token", null) ?: return

    // 예외 처리
    val user = findUser() ?: throw IllegalStateException("User not found")
    ```

- 요약
  + | 연산자  | 역할                 | 반환 결과               |
    | ---- | ------------------ | ------------------- |
    | `?.` | null이면 호출 or 접근 생략 | `null`              |
    | `?:` | null이면 우측 기본값 사용   | non-null 값 또는 throw |

- 면접 관련 질문
  + Safe Call 연산자와 Null 체크의 차이는?
    * Safe Call 연산자(?.)는 null 여부를 명시적 조건 없이 간결하게 처리할 수 있어 가독성이 높고, 체이닝에도 적합합니다. (예: a?.b?.c?.d) 기존의 if (a != null) a.b 구조보다 훨씬 직관적입니다.   
  + Elvis 연산자와 기본값 할당의 차이는?
    * Elvis 연산자(?:)는 null일 경우에만 우측 값을 사용하므로, 기본값이 조건적으로만 적용 된다. 단순한 기본값 할당이 아니라 null 체크와 동시에 fallback을 처리하는 연산자임
  + Elvis 연산자로 예외 처리가 가능한 이유는? 
    * ?: 우측은 표현식이므로, 값을 반환하거나 예외를 던지는 것도 허용되기에 이를 통해 null이 발생할 경우 한 줄로 throw 처리가 가능해지고, 코드가 더 간결하면서도 의도를 분명하게 표현할 수 있음


---


## var vs val
- 개념
  + var 변경 가능한 변수 가변(mutable)
  + val 변경 불가능한 변수 불변(immutable)

- 공통점
  + | 항목                     | 내용                                    |
    | ---------------------- | ------------------------------------- |
    | **둘 다 변수 선언에 사용**      | Kotlin에서 변수를 선언할 때 `var` 또는 `val`을 사용 |
    | **타입 추론 가능**           | `var name = "John"`처럼 타입 명시 없이 선언 가능  |
    | **nullable 허용 가능** | 둘 다 `?`로 nullable 여부 설정 가능 |
    | **스코프 규칙 동일** | 블록 스코프, 지역 변수, 전역 변수 등의 범위 규칙은 동일함 |

- 차이점
  + | 항목             | `var`          | `val`  |
    | -------------- | ---------------------------------- | ------------------------------------------------------------------ |
    | **가변성**        | 값을 변경할 수 있음                 | 최초 할당 이후 값 변경 불가                                                   |
    | **재할당**        | 가능                   | 불가능                          |
    | **사용 예시**      | 루프 인덱스, 상태 변화 변수 | 상수, 변경되지 않는 참조값        |
    | **컴파일러 처리**    | 매번 수정 가능성 고려  | 불변성 보장 → 최적화 유리 |
    
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

    class MyViewModel : ViewModel() {
      private val _uiState = MutableLiveData<UiState>()
      val uiState: LiveData<UiState> = _uiState
    }
    ```
  + `val`은 "참조를 바꿀 수 없다"는 의미이지, 참조 대상의 내부가 불변이라는 의미는 아님. (읽기 전용과 불변 객체는 다른 뜻)
  + `val`은 외부에 읽기 전용으로 상태를 노출하기 위해 사용함. 내부에서는 `_uiState`를 통해 실제 값 변경
  
- 결론
  + | 전략                | 이유                                |
    | ----------------- | --------------------------------- |
    | 기본은 `val` 사용      | 코드 안정성 향상, 예측 가능성 높음              |
    | `var`은 필요한 경우만 사용 | 상태 변화가 필요하고, 제어 가능한 범위 내에서만 사용할 것 |
    | `val` + 가변 컬렉션    | 구조는 고정하고 내부 상태만 바꾸는 경우에 적합        |
    | `val` 많을수록 장점     | 스레드 안정성, 버그 감소, 가독성 및 유지보수 유리     |

- 면접 관련 질문
  + val과 var의 실질적인 차이는 메모리나 성능에도 영향이 있나요?
    * val은 참조 변경이 불가능하므로 컴파일러가 더 많은 최적화 기회를 가질 수 있습니다. 
    * var는 값이 언제든 바뀔 수 있으므로 더 조심해서 처리해야 하고, 멀티스레드 환경에선 동기화도 고려됩니다.
  + val로 선언한 컬렉션도 내부 요소는 변경 가능한 이유는?
    * val은 변수가 참조하는 객체 자체를 바꿀 수 없다는 뜻이지, 그 객체의 내부 구조까지 불변이라는 의미는 아닙니다. 
    * 예: val list = mutableListOf(...)는 참조는 고정되지만 요소는 변경 가능합니다.
  + 어떤 상황에서 var를 꼭 써야 하나요?
    * 값이 계속 변하는 상태값을 관리할 때는 var가 필요합니다. 
    * 예: 루프 인덱스, 스크롤 위치 추적, 카운터 등 다만 이런 경우도 가능한 한 범위를 좁히고, 외부에서 변경 불가능하게 캡슐화해야 안전합니다.


---


### 스코프 함수(let, run with, apply, also)
- 개념 및 정의
  + 객체 컨텍스트 내부에서 블록 코드를 실행할 수 있도록 도와주는 함수
  + 간결하고, 읽기 쉽고, null-safe 만들어주며 객체 초기화, 설정, 연산, 로깅 등에 널리 사용
  + let, run, with, apply, also 등 주로 쓰임

- 컨텍스트 참조 방식 & 반환값 기준
  + | 함수   | 객체 참조 방식 | 반환 값          | 사용 목적 요약                           |
    |--------|----------------|------------------|------------------------------------------|
    | `let`  | `it`           | 람다 결과         | null-safe 연산, 결과 변환                |
    | `run`  | `this`         | 람다 결과         | 설정 + 결과 반환                         |
    | `with` | `this`         | 람다 결과         | 멤버 접근 중심 연산                      |
    | `apply`| `this`         | 객체 자체 반환     | 설정 후 객체 반환 (빌더 패턴처럼)        |
    | `also` | `it`           | 객체 자체 반환     | 부가 작업 (로깅, 디버깅, 체이닝 등)      |

- let
  + 객체 참조 `it`
  + 반환 값 : 람다 마짐가 표현식 결과
  + 사용하는 경우
    * Null 체크와 함께 코드 실행
    * 연산 결과를 변수에 할당
    * 메서드 체이닝
  + ```kotlin
    val result = nullableObj?.let {
      println("not null: $it")
      it.length
    }
    ``` 

- run
  + 컨텍스트 객체 : this
  + 반환 값 : 람다 결과
  + 사용하는 경우
    * 여러 프로퍼티/메서드를 한 블록에서 접근
    * 블록 실행 결과를 반환하고 싶을 때
    * 객체 초기화 + 연산
  + ```kotlin
    val result = person.run {
     println(name)
      age + 5
    }
    ``` 

- with
  + 컨텍스트 객체 : this
  + 반환 값 : 람다 결과
  + 사용하는 경우
    * 확장 함수가 아닌 일반 함수
    * 객체에 여러 번 접근할 때 가독성 향상
    * 주로 수동 DSL 문법에서 자주 사용
  + ```kotlin
    val result = with(person) {
      println(name)
      age + 10
    }
    ``` 

- apply
  + 컨택스트 객체 : this
  + 반환 값 : 객체 자체
  + 주요 사용 경우
    * 객체 초기화 후 그 객체를 그대로 리턴하고 싶을 때
    * 빌더 패턴 대체
  + ```kotlin
    val person = Person().apply {
      name = "Alice"
      age = 30
    }
    ``` 

- also
  + 컨텍스트 객체 : it
  + 반환 값 : 객체 자체
  + 주요 사용 경우
    * 로깅, 디버깅 등 부가 작업
    * 체이닝 내에서 원본 객체 유지하면서 추가 작업 수행
  + ```kotlin
    val person = Person("Alice", 30).also {
      println("Creating person: $it")
    }
    ``` 

- 선택 기준 요약
  + | 사용 상황                 | 추천 함수   |
    | --------------------- | ------- |
    | null 체크 + 결과 변환       | `let`   |
    | 객체 설정 후 다른 결과 반환      | `run`   |
    | 객체 설정 후 그 객체 반환       | `apply` |
    | 부가 작업 후 객체 반환 (예: 로깅) | `also`  |
    | 여러 멤버에 반복 접근 + 결과 반환  | `with`  |

- 추가 팁
  + with 는 확장 함수가 아님
    * `with(obj) { ... }` 형태로 사용 (run/apply 등은 obj.run {...} 식)
  + let + Elvis
    * null-safe 처리에 가장 많이 쓰이는 조합
    * val result = input?.let { it.trim() } ?: "default"
  + apply vs also
    * 결과는 같지만, `this` vs `it` 문맥 차이가 존재
    * `apply` 설정용, `also` 부가행위

- 면접 관련 질문
  + apply와 also의 차이는?
    * 둘 다 객체를 반환하지만, apply는 블록 안에서 this로 객체를 참조하며 설정 목적에 적합하고,also는 it으로 객체를 참조하여 로깅/검증/추가 작업 등에 적합합니다.
  + run과 with의 차이는?
    * run은 확장 함수로 객체에 바로 붙여 쓸 수 있고, with는 일반 함수이므로 객체를 인자로 전달해야 합니다.
    * 용도는 유사하지만 호출 형태에 차이가 있습니다.
  + 스코프 함수 선택 기준은 어떻게 되나요?
    * 연산 결과 반환이 중요하면 let, run, with
    * 객체 자체 반환이 필요하면 apply, also
    * null-safe 연산엔 let, 초기화에는 apply, 부가작업엔 also가 적합합니다.
 

---


### 타입 캐스팅 
- 정의
  + 하나의 객체를 다른 타입으로 변환하는 작업
  + 코틀린은 안전성과 명확성 중심으로 캐스팅을 설계하여 `ClassCastException`과 같은 런타입 에러 최소화 함

- 타입 캐스팅 방식 비교
  + | 분류            | 문법           | 설명                                         |
    |-----------------|----------------|----------------------------------------------|
    | **강제 캐스팅**   | `as`           | 타입이 맞지 않으면 **`ClassCastException` 발생**         |
    | **안전한 캐스팅** | `as?`          | 캐스팅 실패 시 `null` 반환 → **NPE 방지**            |
    | **스마트 캐스트** | `is` / `!is`   | 타입 체크 후 자동 캐스팅 (조건문 내에서만 유효)         |

- 사용 예시
  + `as` 강제 캐스팅
    * ```kotlin
      val any: Any = "Hello"
      val str: String = any as String  // ✅ 성공

      val num: Any = 123
      val str = num as String  // ❌ ClassCastException
      ```
    * 명시적으로 타입 변환하며 실패시 예외 발생
    * 반드시 타입이 정확히 일치하는지 확신이 있는 경우에만 사여ㅛㅇ  
  + `as?` 안전한 캐스팅
    * ```kotlin
      val any: Any = "Kotlin"
      val str: String? = any as? String  // ✅ "Kotlin"

      val num: Any = 123
      val str: String? = num as? String  // ✅ null 반환 (예외 없음)
      ```
    * 실패해도 예외 대신 null 반환
    * 코틀린의 null-safety 와 잘 어울리며, `?.`, `?:` 함께 스이면 더 안전  
  + `is` 타입 검사 (스마트 캐스트)
    * ```kotlin
      fun printLength(obj: Any) {
        if (obj is String) {
           println(obj.length)  // obj가 자동으로 String으로 캐스팅됨
        }
      }

      fun handleInput(input: Any) {
        when (input) {
            is String -> println("String length: ${input.length}")
            is Int -> println("Double: ${input * 2}")
            else -> println("Unknown type")
        }
      }
      ```
    * `is`로 체크하면 블록 안에서 자동으로 스마트 캐스팅됨
    * 해당 블록 안에서는 별도 캐스팅 없이 해당 타입처럼 사용 가능
    * `when`은 `is`를 내장적으로 사용하여 스마트 캐스트를 자동 처리함

- 면접 관련 질문
  + Kotlin의 as?가 Java의 캐스팅과 다른 점은?
    * Java에서는 잘못된 캐스팅 시 무조건 ClassCastException이 발생하지만, Kotlin의 as?는 실패할 경우 예외 대신 null을 반환하여 NPE나 런타임 크래시 위험을 줄여준다.
    * 즉, as?는 안전한 캐스팅 방식이다.
  + 스마트 캐스팅이 작동하지 않는 경우도 있나요?
    * 스마트 캐스트는 val이면서 non-null일 경우에만 안정적으로 작동한다.
    * 만약 var이거나, **nullable 타입(val str: String?)**이면 중간에 값이 바뀔 가능성이 있다고 판단되어 자동 캐스팅이 되지 않는다.
  + is와 as는 어떤 상황에서 어떻게 다르게 쓰이나요?
    * is는 타입을 체크하고, 체크된 블록 안에서 자동으로 스마트 캐스팅을 해준다.
    * as는 타입을 강제로 변환하며, 실패 시 예외가 발생한다.
    * 따라서 안정성이 중요할 땐 is + 조건문, 또는 as?를 활용하는 것이 좋다. 


---


### data class
- 개념/정의
  + 데이터를 담기 위한 클래스
  + `equals()`, `hashCode()`, `toString()`, `copy()` 등 데이터 처리에 필요한 함수들을 자동 생성
  + 코드를 더 간결하게 효율적으로 작성 가능
  
- 자동으로 생성되는 함수들
  + | 함수          | 역할                                               |
    |---------------|----------------------------------------------------|
    | `toString()`  | `"User(name=철수, age=25)"` 형태 문자열 반환           |
    | `equals()`    | 내용 기반 비교 (`==` 연산자 동작 정의)                  |
    | `hashCode()`  | `equals()` 기반 해시값 생성                            |
    | `copy()`      | 일부 속성만 바꾼 새 객체 생성                          |
    | `componentN()`| 구조 분해 선언 지원 (`val (a, b) = obj`)                |

- 사용 예시
  + ```kotlin
    data class User(val name: String, val age: Int)

    fun main() {
      val user1 = User("철수", 25)
      val user2 = user1.copy(age = 30)

      println(user1)  // User(name=철수, age=25)
      println(user2)  // User(name=철수, age=30)

      val (name, age) = user2
      println(name)   // 철수
      println(age)    // 30
    }
    ``` 

- 제약사항
  + primary constructor에 최소 하나 이상의 프로퍼티 필수
  + `open`, `abstract`, `sealed`, `inner` 클래스로 선언할 수 없음
  + 상속 불가 (`final`로 선언 됨)

- 활용 사례
  + API 응답 데이터 모델
  + 로컬 캐시 객체
  + ViewModel의 UI 상태 모델 (ex. data class UiState)
  + 구조 분해 기반 이벤트 전달 (val (type, message) = result)

- 면접 관련 질문
  + 일반 클래스와 data class의 차이점은?
    * 일반 클래스는 toString(), equals(), hashCode() 등을 직접 구현해야 하지만, data class는 주 생성자 프로퍼티 기반으로 이들 메서드를 자동 생성해준다.
    * 결과적으로 더 간결하고 실용적인 데이터 구조를 만들 수 있다.
  + copy() 함수의 장점은?
    * copy()는 기존 객체의 상태를 유지하면서, 일부 프로퍼티만 변경한 새 인스턴스를 쉽게 만들 수 있다.
    * 불변 객체 패턴에서 매우 유용하며, 상태 관리나 UI 변경 감지 로직 등에 자주 쓰인다.
  + data class를 상속할 수 없는 이유는?
    * Kotlin은 data class를 값 객체로 간주하므로, 동등성, 해시값, 복사 등에 대한 일관된 동작이 필요하다.
    * 상속을 허용하면 이런 계약이 깨질 수 있어 자동으로 final로 선언된다. 


---


### enum 열거형 클래스
- 개념 및 정의
  + 미리 정의된 고정된 상수 집합을 표현할 때 사용하는 열거형 클래스
  + 방향, 상태 코드, 명령어 등 한정된 선택지가 있을  때 적합
  + 각 열거 상수는 싱글턴 객체이며 필요 시 생성자와 메서드 정의 가능

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
  + enum 별로 다른 동작 구현 (함수 정의)
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
    * 각 enum 상수가 자신만의 메서드 구현을 가질 수 있음
    * 추상 메서드를 정의하고 override 가능
  + `when` 과 함께 쓰임
    * ```kotlin
      fun handleDirection(dir: Direction) = when(dir) {
        Direction.NORTH -> "Going up"
        Direction.SOUTH -> "Going down"
        Direction.EAST  -> "Going right"
        Direction.WEST  -> "Going left"
      }
      ```
    * `when`과 함께 사용 시 모든 enum을 다 다루면 `else` 생략 가능
    * enum 타입에 대해 `exhaustive`(가능한 모든 경우를 빠짐없이 다룸) 체크 수행
  + `values()` / `valueOf()`
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
  + enum class는 어떤 상황에서 적합한가?
    * enum은 가능한 값이 정해진 범위 내에 있고, 각 값이 하나의 고유한 상태나 명령을 의미할 때 적합
    * 예: 방향(Direction), HTTP 상태 코드, 정렬 기준 등
  + enum과 sealed class의 근본적인 차이?
    * enum은 고정된 상수 집합만 정의할 수 있고, 각 상수는 동일한 타입
    * 반면 sealed class는 서로 다른 구조나 속성을 가진 하위 타입을 정의할 수 있어 더 유연
    * 복잡한 상태 분기나 타입 분기에는 sealed class가 더 적합
  + enum 클래스에서 동작을 다르게 구현할 수 있는 이유는?
    * Kotlin의 enum 클래스는 각 상수를 객체처럼 취급하므로, 각 상수가 자체적으로 함수를 오버라이드할 수 있음
    * 이를 통해 Operation.PLUS, Operation.MINUS처럼 각 상수마다 다른 동작을 지정할 수 있음


---


### 확장 함수(Extension Function)
- 정의
  + 확장 함수는 기존 클래스의 소스를 변경하지 않고 외부에서 마치 그 클래스의 메서드처럼 새로운 기능을 추가할 수 있게 해주는 기능
  + 주로 Java 표준 클래스, 서드파티 라이브러리, 또는 개ㅏ 만든 클래스에 간편하게 기능을 부여할 때 를 사용

- 기본 구조 및 문법
  + ```kotlin
    fun String.addSuffix(suffix: String): String {
      return this + suffix
    }

    val name = "Chat"
    println(name.addSuffix("GPT"))  // ChatGPT
    ``` 
  + 수신 객체: `String` -> 확장 대상
  + `this` 키워드를 통해 수신 객체 내부처럼 접근 가능

- 주요 특징 및 장점
  + | 항목            | 설명                                                                |
    | ------------- | ----------------------------------------------------------------- |
    | 코드 간결성        | 클래스 외부에서 필요한 기능을 직접 추가할 수 있음 (유틸 함수의 메서드화)                        |
    | 클래스 수정 불필요    | Java 또는 타인이 만든 클래스에도 기능 확장 가능                                     |
    | 정적 디스패치       | 컴파일 타임 타입 기준으로 바인딩됨. → 다형성 불가                                     |
    | 멤버 함수 우선      | 클래스 내부에 동일 시그니처의 멤버 함수가 있다면 **확장 함수는 무시됨**                        |
    | null 수신 객체 가능 | `fun String?.safeLength() = this?.length ?: 0` 처럼 null-safe 처리 가능 |
    | 스코프 제한 가능     | 특정 클래스, 함수 내에서만 정의 가능 (`private`, `internal` 등 제한 가능)             |

- 주의사항
  + 확장 함수는 클래스 내부의 private, protected 멤버에는 접근할 수 없음
  + 실제로 클래스가 수정된 것이 아니라, 컴파일러가 정적 함수로 바꿔주는 문법적 설탕(Syntactic Sugar)
  + 동일한 확장 함수 이름이 여러 곳에서 정의되면 충돌 가능성 있음
  + 남용 시 코드 추적/관리 어려움 

- 확장 함수 vs 멤버 함수
  + | 항목       | 멤버 함수                          | 확장 함수                  |
    | -------- | ------------------------------ | ---------------------- |
    | 선언 위치    | 클래스 내부                         | 클래스 외부                 |
    | 내부 멤버 접근 | 가능 (`private`, `protected` 포함) | 불가능                    |
    | 다형성      | 가능 (`override`, `open` 등)      | 불가능 (정적 디스패치)          |
    | 용도       | 클래스의 핵심 기능 구현                  | 부가 기능, 유틸 함수, DSL 문법 등 |

- 사용 시점
  + 클래스 소스에 접근할 수 없는 경우
  + 보조 기능을 만들고 싶을 때 (ex. String.capitalizeWords())
  + 안드로이드에서 View/Context 등에 공통 유틸 제공할 때
  + DSL 스타일 코드 구성 시 

- 면접 관련 질문
  + 확장 함수와 Java 유틸 클래스의 차이점은?
    * Java의 유틸 클래스는 StringUtils.method(str) 형태로 사용하지만, Kotlin의 확장 함수는 str.method()처럼 사용할 수 있어 가독성이 좋고, IDE 자동완성도 더 자연스럽게 동작한다.
    * 기능은 동일할 수 있어도 문법적 표현력과 코드 흐름에서 차별성이 있다.
  + 확장 함수의 단점은?
    * 다형성 지원이 안 됨 (정적 디스패치) → 런타임 타입에 따른 함수 선택 불가
    * 클래스 외부에 정의되므로 내부 상태나 캡슐화된 정보를 사용할 수 없음
    * 함수 충돌 위험 및 남용 시 유지보수 어려움 (출처 추적 힘듦)
  + 확장 함수가 멤버 함수와 동일한 시그니처일 때 어떤 함수가 호출되나요?
    * 항상 멤버 함수가 우선 호출된다.
    * 확장 함수는 컴파일 타임에 정적으로 바인딩되므로, 클래스 내부에 정의된 메서드와 이름과 시그니처가 같다면 확장 함수는 무시된다.


---


### sealed class
- 개념 및 정의
  + 하위 클래스의 정의를 같은 파일로 제한한 제한적 상속 클래스
  + 컴파일러가 하위 타입을 모두 알 수 있기 때문에, `when` 문에서 안전하고 완전한 분기가 가능
  + `enum class`보다 더 유연하게 각 하위 타입에 **서로 다른 속성과 동작**을 부여할 수 있어 복잡한 상태 모델링에 적합

- 기본 문법
  + ```kotlin
    sealed class Result

    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
    object Loading : Result()

    // when 과 함께 사용
    fun handle(result: Result) {
      when (result) {
          is Success -> println("성공: ${result.data}")
          is Error   -> println("에러: ${result.message}")
          is Loading -> println("로딩 중")
      }
    }
    ```
  + `when` 블록에서 `else` 생략 가능 - 컴파일러가 모든 하위 타입을 알고 있어서 exhaustive check 가능

- 특징 요약
  + | 항목                | 설명                                                 |
    | ----------------- | -------------------------------------------------- |
    | **상속 제한**         | 하위 클래스는 **같은 파일 내에서만 정의 가능** (컴파일러가 전체 타입 인지)      |
    | **직접 인스턴스화 불가**   | 추상 클래스처럼 동작. `sealed` 자체는 객체 생성 불가                 |
    | **다양한 하위 클래스 가능** | `data class`, `object`, `class` 등 다양한 형태로 하위 정의 가능 |
    | **when 최적화**      | `when` 사용 시 `else` 생략 가능 (모든 타입 처리 확인됨)            |
    | **enum보다 유연함**    | enum은 상수만, sealed는 **속성/동작 커스터마이징** 가능             |

- enum class vs sealed class
  + | 비교 항목     | `enum class`            | `sealed class`                        |
    | --------- | ----------------------- | ------------------------------------- |
    | 사용 목적     | 제한된 **고정 상수 집합**        | **타입 계층 구조**, 다양한 상태 표현에 적합           |
    | 런타임 확장    | ❌ 불가능 (컴파일 시 고정)        | ✅ 가능 (같은 파일 내에서 자유롭게 정의 가능)           |
    | 속성 커스터마이징 | 제한적 (모든 상수는 동일 구조)      | 각 하위 클래스마다 **속성, 메서드 다르게 정의 가능**      |
    | when 처리   | 컴파일러가 자동 exhaustive 확인  | 동일 (else 생략 가능)                       |
    | 실무 활용     | 간단한 상수 분기 (정렬 기준, 방향 등) | 복잡한 상태 모델링 (API 응답, UI 상태, Command 등) |
 
- 실무 사용 예시
  + API 응답 모델
    * ```kotlin
      sealed class ApiResult<out T> {
        data class Success<T>(val data: T): ApiResult<T>()
        data class Failure(val error: Throwable): ApiResult<Nothing>()
        object Loading: ApiResult<Nothing>()
      }
      ``` 
  + UI 상태 표현 
    * ```kotlin
      sealed class LoginState {
        object LoggedOut : LoginState()
        data class LoggedIn(val user: User) : LoginState()
        object Loading : LoginState()
      }
      ``` 

- 면접 관련 질문
  + `sealed class`는 왜 `enum class`보다 더 유연한가요?
    * enum은 고정된 값만 표현할 수 있고, 각 상수가 동일한 구조를 가져야 합니다.
    * 반면, sealed class는 하위 클래스마다 서로 다른 속성과 동작을 정의할 수 있어 복잡한 상태나 데이터를 표현하는 데 더 적합합니다.
  + sealed class를 사용했을 때 when에서 else 없이도 안전한 이유는?
    * sealed class는 같은 파일 내에 정의된 모든 하위 타입을 컴파일러가 알 수 있어서, when 문이 exhaustive check를 수행할 수 있습니다.
    * 따라서 모든 하위 타입을 다 처리하면 else 없이도 컴파일 오류가 발생하지 않습니다.
  + sealed class와 abstract class의 차이는?
    * abstract class는 상속에 제한이 없고, 어디서든 하위 클래스 정의가 가능합니다.
    * 반면 sealed class는 하위 타입을 같은 파일에만 정의할 수 있어, 컴파일러가 전체 타입을 추론할 수 있다는 점이 다릅니다.
    * 이 덕분에 when 문에서 안전하고 명확한 분기 처리가 가능합니다.


---


### 구조 분해 선언(Destructuring Declarations)
- 정의
  + 객체의 프로퍼티들을 하ㄴ 번에 여러 변수로 분해해서 할당할 수 있게 해주는 문법
  + `data class`, `Map`, `List`, `Pair`, `Triple` 등 사용

- 기본 개념
  + ```kotlin
    val (a, b) = someObject  // someObject.component1(), component2() 호출
    ```
  + 구조 분해가 가능하려면 해당 클래스가 `componentN()` 연산자 함수 제공
  + 코틀린 컴파일러는 구조 분해 선언을 만나면 내부적으로 `component1()`, `component2()` 형태로 메소드 호출

- 주요 사용 사례
  + Data Class
    * ```kotlin
      data class User(val name: String, val age: Int)

      val user = User("Alice", 30)
      val (name, age) = user
      println("Name: $name, Age: $age")  // 출력: Name: Alice, Age: 30
      ```

  + Map.Entry
    * ```kotlin
      val map = mapOf("A" to 1, "B" to 2)

      for ((key, value) in map) {
        println("Key: $key, Value: $value")
      }

      // 출력:
      // Key: A, Value: 1
      // Key: B, Value: 2
      ```

  + 리스트 or 배열
    * ```kotlin
      val list = listOf("Apple", "Banana", "Cherry")
      val (first, second) = list

      println("First: $first, Second: $second")  // 출력: First: Apple, Second: Banana
      ```

  + 여러 값 반환
    * ```kotlin
      data class Result(val value: Int, val status: String)

      fun process(): Result = Result(42, "Success")

      val (value, status) = process()
      println("Value: $value, Status: $status")
      ```

  + 일부 값 무시
    * ```kotlin
      data class Point(val x: Int, val y: Int, val z: Int)

      val point = Point(10, 20, 30)
      val (x, _, z) = point

      println("X: $x, Z: $z")
      ```

  + 람다 파라메터
    * ```kotlin
      val users = listOf(User("Bob", 25), User("Charlie", 35))

      users.forEach { (name, age) ->
          println("$name is $age years old")
      }

      val map = mapOf(1 to "one", 2 to "two")
      map.forEach { (key, value) ->
          println("$key -> $value")
      }
      ```

- 성능 및 주의사항
  + 구조 분해는 컴파일 시 componentN 호출로 변환되므로 성능 부담은 거의 없음
  + 다만, 지나치게 많은 변수 사용은 가독성 저하나 실수를 유발할 수 있음
  + 컴파일 시점에 componentN() 함수가 정의되어 있어야 컴파일 가능 

- 질문
  + 구조 분해 선언시 componentN() 함수는 어떤 역할을 하나요?
    * 구조 분해 선언시 내부적으로 호출되는 메서드 입니다. 
      예를 들어 val (x, y) = point는 point.component1()과 point.component2()를 호출합니다. 이는 컴파일러가 자동으로 변환해주는 형태입니다.
  + 구조 분해 선언이 성능에 영향을 줄 수 있나요?
    * 아니요. 구조 분해는 컴파일 타임에 componentN() 메서드 호출로 변환되므로 오버헤드는 거의 없습니다. 
    * 다만, 너무 많은 분해 변수는 코드 가독성을 해칠 수 있어 주의가 필요합니다.
  + 구조 분해 선언이 가능하려면 어떤 조건이 필요한가?
    * 해당 클래스가 componentN() 함수를 제공해야 합니다.
    * data class는 이 함수를 자동으로 생성하며, Map.Entry, Pair, Triple 등 일부 클래스는 표준 라이브러리에서 제공됩니다.  


---


### inline 키워드
- 정의
  + `inline` 함수란 **호출되는 대신 그 본문이 호출 위치에 '직접 복사'되어 삽입**되는 함수
  + 주로 **고차 함수**에서 사용되며, **람다 객체 생성을 피하고 성능을 최적화**하는 목적

- 사용하는 이유?
  + | 목적           | 설명                                      |
    | ------------ | --------------------------------------- |
    | ✅ 성능 최적화     | 함수 호출 오버헤드 제거, 특히 람다 객체 생성을 생략          |
    | ✅ return 제어  | 람다 내부에서 return 사용 가능 (non-local return) |
    | ✅ reified 지원 | 제네릭 타입을 런타임에도 유지할 수 있음                  |
    | ❌ 코드 팽창 주의   | 반복 사용 시 바이너리 크기 증가 가능성 있음               |

- 기본 동작
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
  + `noinline` : 인라인 안하고 객체로 넘기고 싶을 때
    * ```kotlin
      inline fun test(a: () -> Unit, noinline b: () -> Unit) {
        a() // 인라인됨
        b() // 인라인되지 않음
      }
      ```
    * 람다 여러 개 중 일부만 인라인하고 싶을 때 유용
  + `crossinline` : 람다 안에서 non-local return 못하게 막기
    * ```kotlin
      inline fun doSomething(crossinline action: () -> Unit) {
         Thread {
            action() // 여기서 return 하면 컴파일 에러
         }.start()
      }
      ```
    * 다른 스레드/콜백에서 실행될 람다에는 return 쓰면 위험하므로 금지

- 실무 사용 예시
  + 성능 측정
    * ```kotlin
      inline fun measure(block: () -> Unit) {
        val start = System.currentTimeMillis()
        block()
        println("Elapsed: ${System.currentTimeMillis() - start}ms")
      }
      ```
  + 제네릭 타입 유지(`reified`)
    * ```kotlin
      inline fun <reified T> Gson.fromJson(json: String): T { 
         return this.fromJson(json, T::class.java)
      }
      ```
    * 일반 함수에서는 제네릭 타입 T는 런타임에 소멸되지만, inline + reified 덕분에 런타임에도 타입 정보를 유지 가능

- 면접 관련 질문
  + `inline` 함수 장점
    * 고차 함수 호출 시 발생하는 람다 객체 생성을 피해서 성능 향상
    * return 문법 유연화, reified 타입 활용 가능
  + `noinline` 이 필요한가?
    * 인라인 함수 내 람다 중 일부를 객체로 유지하고 싶을 때 사용
    * 예: 람다를 다른 함수에 전달하거나 저장하려는 경우 인라인하면 안 되기 때문
  + `reified` 는 왜 `inline` 함수에서만 사용할 수 있는가?
    * Kotlin의 일반 제네릭은 타입 소거되지만, inline 함수는 컴파일 시점에 T의 실제 타입 정보를 알 수 있으므로 이를 런타임에도 유지하게 하기 위해 reified와 함께 사용함


---


### Kotlin & Java
- 두 언어 모두 JVM에서 실행되는 언어, 안드로이드 개발이나 서버 개발에서 많이 쓰임 

- 공통점
  + | 항목            | 설명 |
    |-----------------|------|
    | JVM 기반        | 둘 다 Java Virtual Machine에서 실행 |
    | Java API 호환   | Kotlin은 Java API 100% 호출 가능 |
    | 객체지향 지원    | 클래스, 인터페이스, 상속, 다형성 등 동일 구조 |
    | Android 개발 가능 | Android 공식 지원 언어 (Java 7+ / Kotlin 1.3+) |
    | 스레드, 네트워크 등 | 동등 수준의 저수준 기능 지원 가능 |

- 차이점
  + | 항목                | Java                          | Kotlin                                  |
    |---------------------|-------------------------------|------------------------------------------|
    | 문법 간결성         | 장황한 편                      | `val`, `data class`, `when` 등 간결 |
    | Null 안정성         | `NullPointerException` 자주 발생 | 컴파일 타임에서 null 검사 (`?`, `!!`) |
    | 데이터 클래스        | getter/setter, equals 수동 작성 | `data class` 한 줄로 자동 생성         |
    | 함수형 스타일        | 람다/Stream 사용 불편              | 고차 함수, 람다, `map/filter` 자연스러움 |
    | 확장 함수           | 불가                             | 클래스 외부에서 메서드 확장 가능        |
    | 타입 캐스팅         | 강제 `instanceof + cast`         | `is` + 스마트 캐스팅 자동화            |
    | 비동기/코루틴 지원   | 외부 스레드/라이브러리 필요         | `suspend`, `launch`, `async` 기본 지원  |
    | 기본 타입 분리       | `int`, `double` 등 primitive 존재 | 전부 객체(`Int`, `Double`)로 통일        |


- 예시
  + ```java
    // 변수선언
    String name = "철수";

    // Null 안전 처리
    if (user != null) {
      System.out.println(user.getName());
    }

    // 데이터 클래스
    public class User {
      String name;
      int age;

      // 생성자, getter, equals, hashCode ...
    }
    ```
  + ```kotlin
    // 변수선언
    val name = "철수"   // 자동 타입 추론

    // Null 안전 처리
    println(user?.name ?: "이름 없음") // null-safe + default

    // 데이터 클래스
    data class User(val name: String, val age: Int)
    ```

- 결론
  + Kotlin은 간결함 + 안전성 + 현대적 문법에 중점을 둔 언어
  + Java는 보편성 + 방대한 생태계를 갖춘 기반 언어
  + Android 개발이나 현대 서버 개발에서는 Kotlin이 점점 우세
  + 하지만 상호운용성 완벽 → 둘을 함께 사용하는 프로젝트도 많음 

- 면접 관련 질문
  + Kotlin이 Java보다 유리한 점은?
    * Null 안정성, 코루틴, 간결한 문법, 고차 함수 등의 지원으로 코드 품질 및 생산성이 향상됩니다.
  + Kotlin 코드가 Java에서 호출 가능한가요?
    * 가능함. Kotlin은 JVM 바이트코드로 변환되므로 Java 클래스와 완전히 상호 운용됩니다. 
    * @JvmStatic, @JvmOverloads 등으로 더 자연스러운 호환도 가능.
  + Kotlin의 확장 함수는 Java의 어떤 대안인가요?
    * 기존 클래스에 새로운 메서드를 추가할 수 있는 방식으로, Java의 유틸 클래스보다 가독성과 사용성이 좋습니다.
    * 하지만 private 멤버에는 접근 불가한 제약도 있습니다.


---


### 상속 제어 (open, final, abstract)
- 정의
  + 코틀린에서 상속 제어 키워드는 클래스나 함수, 프로퍼티가 상속/재정의 가능한지 명확하게 제어하는 기능
  + Java 와 기본값이 다르며 코틀린의 안정성과 설계 철학을 보여주는 부분

- 핵심 개념
  + | 키워드        | 의미                             | 기본값 여부            |
    | ---------- | ------------------------------ | ----------------- |
    | `final`    | **상속 불가**, 오버라이드 금지            | ✅ Kotlin의 기본값     |
    | `open`     | **상속 가능**, 오버라이드 허용            | ❌ 명시적으로 선언해야 함    |
    | `abstract` | 추상 멤버 또는 클래스, **무조건 오버라이드 필요** | ❌ 추상 클래스에서만 사용 가능 |

- 상세 설명
  + final
    * 코틀린에서는 별도 지정이 없으면 `final`
    * 오버라이드/상속 하려면 명시적으로 `open` or `abstract` 로 풀어야 함
    * ```kotlin
      class Animal {
         fun sound() { println("Animal sound") } // final이 기본
      }

      // class Dog : Animal() { ... } // ❌ 컴파일 에러 (speak 오버라이드 불가)
      ```
  + open
    * 해당 클래스 or 멤버를 상속/재정의 가능하게 만듦
    * 용도: 다형성, 테스트용 mock 클래스 등에 사용
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
    * 인스턴스화 불가, 일부만 구현해도 가능 (abstract + concrete), 메서드에도 abstract 가능
    * ```kotlin
      abstract class Animal {
        abstract fun speak()
      }

      class Cat : Animal() {
        override fun speak() = println("Meow")
      }
      ```

- 실전 주의사항
  + `data class`는 `final` 고정 (→ 상속 불가)
  + `abstract`에는 반드시 open이 내포되어 있음 (굳이 `open abstract` 쓸 필요 없음)
  + `open` 없이 오버라이드 시도하면 컴파일 에러
  + 실무에서는 의도하지 않은 상속을 막기 위해 대부분 `final`을 유지하고 필요한 경우만 `open` 지정 

- 면접 관련 질문
  + Kotlin은 왜 모든 클래스가 기본적으로 final인가요?
    * 의도하지 않은 상속이나 오버라이드를 방지하고 안정성과 예측 가능성을 높이기 위해서입니다.
    * Java는 반대로 기본적으로 상속 가능하지만, Kotlin은 명시적 설계를 권장합니다.
  + abstract 클래스와 interface의 차이는?
    * abstract 클래스는 상태(필드) + 구현 가능 / 단일 상속
    * interface는 필드 없음(기본은), 구현 가능 / 다중 구현 가능
  + data class는 왜 상속이 불가능한가요?
    * data class는 equals(), hashCode(), copy() 등을 자동 생성하며, 상속 시 이 동작이 깨질 수 있기 때문입니다.
    * 명확한 데이터 표현용으로 final이 기본입니다.


---


### Range & Progression
- 개요
  + Range : 정해진 시작갑소가 끝값 사이의 연속된 값 집합 (ex: 1,2,3,4)
  + Progression(수열) : 일정한 간격(step)을 가진 수열 표현. 증/감소 모두 가능
  + 반복문, 조건문, 데이터 필터링 등 코드 가독성과 표현력 향상시킴

- Range
  + 닫힌 범위
    * ```kotlin
      val range = 1..5         // 1, 2, 3, 4를 포함하는 IntRange
      val charRange = 'a'..'d' // 'a', 'b', 'c', 'd'를 포함하는 CharRange
      ```
  + 반 열림 범위
    * ```kotlin
      val range = 1..<5 // 1 to 4 (5제외)
      ```
  + 포함 여부 체크
    * ```kotlin
      val x = 3
      if (x in 1..5) println("x는 1~5안에 있음")
      ```   
- Progression
  + step 키워드
    * ```kotlin
      val even = 0..10 step 2
      println(even.toList())  // [0, 2, 4, 6, 8, 10]
      ``` 
  + downTo
    * ```kotlin
      val countdown = 10 downTo 0
      println(countdown.toList())  // [10, 9, ..., 0]

      val customStep = 10 downTo 0 step 2
      println(customStep.toList())  // [10, 8, 6, 4, 2, 0]\
      ``` 
  + 반목문 활용
    * ```kotlin
      for (i in 1..5) print("$i ")         // 1 2 3 4 5
      for (i in 5 downTo 1 step 2) print("$i ") // 5 3 1
      ``` 
  + reversed()
    * ```kotlin
      val range = 1..5
      println(range.reversed().toList()) // [5, 4, 3, 2, 1]
      ```

- 구성요소 비교
  + | 구분      | Range (`..`, `..<`) | Progression (`step`, `downTo`) |
    | ------- | ------------------- | ------------------------------ |
    | 포함 여부   | 끝값 포함 / 미포함 선택 가능   | 자동 포함 (값 생성 규칙 따름)             |
    | step 설정 | 기본값 1 (증가)          | 자유롭게 `step` 지정 가능 (증가/감소 모두)   |
    | 방향      | 증가만 가능              | 증가/감소 모두 지원 (`downTo`, `step`) |
    | 활용도     | 범위 검사, 조건문, 반복문     | 커스텀 반복 조건, 특정 간격 수열 생성 등       |

- 면접 관련 질문
  + `..`와 `..<`의 차이점은?
    * `..`는 끝 값을 포함하는 닫힌 범위이고, `..<`는 끝 값을 포함하지 않는 반 열린 범위입니다.
    * 반복 조건이나 인덱스 조작에서 실수를 줄이기 위해 ..<를 자주 사용합니다.
  + Progression과 Range의 가장 큰 차이는?
    * Range는 기본적으로 1씩 증가하는 값의 집합이며 방향과 간격이 고정되어 있습니다.
    * 반면 Progression은 증가/감소 방향과 step 조절이 자유롭습니다.
  + Range와 함께 step을 사용하는 이유는?
    * 일정 간격의 값만 반복하고 싶을 때 사용합니다.
    * 예: 짝수만 반복, 역순 반복, 3단계 건너뛰기 등.       


---


### 접근제어자
- 개념 및 정의
  + 클래스, 함수, 변수 등이 어디에서 접근 가능한지 제한하는 키워드
  + 코틀린은 기본적으로 안정성과 명확성을 중시하기 때문에 접근 범위 명시하는 것이 중요함

- 접근 제어자 비교
  + | 접근 제어자   | 클래스 외부 접근 | 상속 클래스 접근 | 같은 파일 내 접근 | 같은 모듈 접근 | 다른 모듈 접근 |
    |---------------|------------------|------------------|-------------------|----------------|----------------|
    | `public`      | ✅               | ✅               | ✅                | ✅             | ✅             |
    | `internal`    | ✅               | ✅               | ✅                | ✅             | ❌             |
    | `protected`   | ❌               | ✅               | ✅ (클래스 내)     | ✅             | ❌             |
    | `private`     | ❌               | ❌               | ✅ (같은 클래스/파일) | ❌             | ❌             |

- public (기본값, 어디서나 접근 가능)
  + **어디서든 접근 가능**하며, 특별한 키워드 없이 선언하면 기본값으로 적용됨
    ```kotlin
    // 파일: Car.kt
    package com.example

    class Car {
        fun drive() = println("Car driving")
    }

    // 다른 패키지
    val car = Car()
    car.drive() // ✅ 정상 접근
    ```
- `internal` (같은 모듈 내에서만 접근 가능)
  + 모듈 단위로 접근 범위 제한
  + 모듈 기준이기 때문에 앱 내부용 코드, 라이브러리 구현 등에서 활용 높음
    ```kotlin
    internal class Engine {
      fun start() = println("Engine start")
    }

    // 같은 모듈
    val e = Engine() // ✅ 가능

    // 다른 모듈
    val e = Engine() // ❌ 컴파일 에러
    ``` 

- protected (상속 관계 내부에서만 접근 가능)
  + 외부에서는 보이지 않지만, 하위 클래스에서는 사용 가능
    ```kotlin
    open class Animal {
      protected fun breathe() = println("숨 쉬는 중")
    }

    class Dog : Animal() {
        fun bark() {
            breathe() // ✅ 가능
        }
    }

    val d = Dog()
    // d.breathe() // ❌ 외부에서는 접근 불가
    ```

- `private` (같은 클래스 or 파일 내에서만 접근 가능)
  + 캡슐화에 최적, 외부에 절대 노출되지 않도록 제한
    ```kotlin
    class Secret {
      private val code = "1234"
      private fun unlock() = println("Unlocked with $code")

      fun tryUnlock() = unlock() // ✅ 내부에서 호출 가능
    }

    // 파일 수준
    // File: Util.kt
    private fun helper() = println("File-local only")

    fun api() = helper() // ✅ 가능

    // 다른 파일에서 helper() 호출 시 컴파일 에러
    ```  

- 실무 활용 팁
  + | 상황                     | 권장 접근 제어자   | 이유           |
    | ---------------------- | ----------- | ------------ |
    | 유틸/헬퍼 함수, internal API | `internal`  | SDK 외부 노출 방지 |
    | 상속 구조 내 공통 로직          | `protected` | 하위 클래스 전용    |
    | 민감한 로직, 상태 캡슐화         | `private`   | 완전 은닉화       |
    | 외부에서 호출 가능한 API        | `public`    | API 명확히 노출   |

- 면접 관련 질문
  + Kotlin에만 있는 접근제어자는?
    * internal. 모듈 단위 접근 제어로 Java에는 존재하지 않음.
  + Java의 default 접근제어자와 Kotlin의 internal 차이?
    * Java의 default는 패키지 단위이고, Kotlin의 internal은 모듈 단위임.
    * internal은 multi-module 프로젝트에서 코드 은닉에 효과적
  + Kotlin에서는 top-level 함수에도 접근제어자를 붙일 수 있는가?
    * 가능. 함수, 클래스, 변수 모두 top-level일 경우에도 private, internal 등 설정 가능.
 

---


### reified & 제네릭
- 정의
  + 제네릭
    * 타입에 의존하지 않고 유연하고 재사용 가능한 코드를 작성할 수 있게 해주는 문법
    * ```kotlin
      fun <T> printItem(item: T) {
        println(item)
      }
      ```
    * ⚠️ 타입 소거(Type Erasure) 발생
      1. 컴파일 후 T의 실제 타입 정보가 런타임에 사라짐
      2. 예: T::class, T::class.java, is T, as T 사용 ❌불가
  + reified
    * reified: "구체화된"이라는 의미. 제네릭 타입 T를 런타임에도 타입 정보로 유지
    * 사용 조건:
      1. inline 함수의 타입 파라미터에만 사용 가능
      2. 컴파일 시 실제 타입으로 치환(inline + 타입 구체화)  
    * ```kotlin
      inline fun <reified T> getType(): Class<T> {
        return T::class.java // ✅ 가능!
      }
      ``` 

- 일반 제네릭과의 차이
  + | 기능                             | 일반 제네릭    | reified 제네릭      |
    | ------------------------------ | --------- | ---------------- |
    | `T::class`, `T::class.java` 사용 | ❌         | ✅ 가능             |
    | `is T`, `as T` 타입 검사           | ❌         | ✅ 가능             |
    | 런타임 타입 유지                      | ❌ (타입 소거) | ✅ (컴파일 시 타입 삽입)  |
    | 사용 가능 위치                       | 어디든       | `inline` 함수 안에서만 |
   
- 주요 활용 예시
  + Gson JSON 파싱
    * ```kotlin
      inline fun <reified T> parseJson(json: String): T {
        return Gson().fromJson(json, T::class.java)
      }

      val user: User = parseJson(jsonString)
      ``` 
    * 일반 제네릭이라면 TypeToken<T>().type을 써야 해서 복잡함
  + 타입 검사/필터링
    * ```kotlin
      inline fun <reified T> isOfType(value: Any): Boolean {
        return value is T
      }
      
      val result = isOfType<String>("Hello") // true
      ```
  + ViewModel Factory 
    * ```kotlin
      inline fun <reified VM : ViewModel> createViewModel(): VM {
        return ViewModelProvider(this).get(VM::class.java)
      }
      ```

- 상황별 사용 정리
  + | 상황                                   | `reified` 필요 여부 |
    | ------------------------------------ | --------------- |
    | 런타임 타입 검사 (`is T`, `as T`)           | ✅ 필요            |
    | `T::class`, `T::class.java` 사용       | ✅ 필요            |
    | 단순한 타입 재사용 / 전달                      | ❌ 불필요           |
    | Gson/Retrofit/뷰모델 등 런타임 타입 반영 필요한 경우 | ✅ 매우 유용         |
       
- 면접 관련 질문
  + `reified` 키워드 필요한 이유?
    * Kotlin의 제네릭은 타입 소거 때문에 T::class, is T 같은 연산이 불가
    * reified는 실제 타입 정보를 코드에 삽입해서 런타임에도 활용 가능하게 해줌
  + `inline` 함수에서만 `reified`를 쓸 수 있는가?
    * inline 함수는 컴파일 시 함수 본문이 호출 위치에 복사됨
    * 이때 T가 실제 타입으로 치환되므로 타입 정보 유지 가능
    * 일반 함수는 이 치환이 안 되기 때문에 타입 소거 발생 → reified 사용 불가


---


### typealias
- 정의
  + `typealias`는 기준 타입에 새로운 이름(별칭) 부여하는 문법
  + 새 타입을 만드는 것이 아니라 기존 타입을 가리키는 별명(alias) 역할

- 기본 사용법
  + typealias 키워드를 사용하여 새로운 이름과 기존 타입을 연결합니다.
  + ```kotlin
    typealias Name = String
    typealias UserList = List<User>
    typealias ClickListener = (View) -> Unit
    ```
    
- 사용하는 이유 및 장점
  + 가독성향상
    * 복잡하거나 긴 타입 이름을 의미 있는 이름으로 대체하여 코드를 더 쉽게 이해할 수 있도록 합니다.
    * ```kotlin
      val handler: (Int, String, Boolean) -> Unit // ❌ 직관성 낮음
      typealias ErrorHandler = (Int, String, Boolean) -> Unit
      val handler: ErrorHandler                    // ✅ 의미 명확
      ```
  + 복잡한 제네릭 타입 간소화
    * 제네릭을 사용하는 복잡한 타입을 더 짧고 관리하기 쉬운 이름으로 만들 수 있습니다.
    + ```kotlin
      // BEFORE
      val processor: (List<Map<String, Set<Int>>>, (String) -> Boolean) -> List<String>

      // AFTER
      typealias DataFilter = (String) -> Boolean
      typealias ComplexData = List<Map<String, Set<Int>>>
      typealias DataProcessor = (ComplexData, DataFilter) -> List<String>

      val processor: DataProcessor = { data, filter -> emptyList() }\
      ```
  + 콜백, 함수 타입 정리
    * ```kotlin
      typealias LoginCallback = (Boolean, String?) -> Unit

      fun login(id: String, pw: String, callback: LoginCallback) { ... }
      ```
  + 제네릭 타입
    * ```kotlin
      typealias StringList<T> = List<T>
      val users: StringList<String> = listOf("A", "B")

      typealias IntList = List<Int>
      val numbers: IntList = listOf(1, 2, 3)
      ```
  + 내부 클래스, 객체에도 사용 가능
    * ```kotlin
      class Outer {
        inner class Inner
        object NestedObject
      }

      typealias InnerClass = Outer.Inner
      typealias NestedObj = Outer.NestedObject
      ``` 
    
- 주의사항
  + | 항목        | 설명                                         |
    | --------- | ------------------------------------------ |
    | 새로운 타입 아님 | `typealias`는 진짜 새로운 타입이 아니며, 컴파일 타임에 치환될 뿐 |
    | 생성자 없음    | `typealias` 자체에 생성자, 프로퍼티를 추가할 수 없음        |
    | 상속 불가     | `open`, `abstract`, `sealed` 등 사용 불가       |
    | 충돌 주의     | 동일한 타입에 너무 많은 별칭을 만들면 오히려 혼란을 초래할 수 있음     |

- 언제 typealias를 사용하면 좋을까?
  + | 상황               | 이유                                             |
    | ---------------- | ---------------------------------------------- |
    | 복잡한 타입 반복 사용     | 의미 있는 이름으로 정의하여 재사용 및 간결화                      |
    | 함수 타입 콜백 명확화     | 매개변수/반환값 구조를 명확하게 표현 가능                        |
    | 도메인에 맞는 타입 의미 부여 | String → `UserName`, Int → `UserId` 등으로 표현력 향상 |

- 면접 관련 질문
  + typealias는 어떤 역할을 하나요?
    * 기존 타입의 별칭을 제공해 복잡한 타입을 간결하고 명확하게 표현할 수 있게 해줍니다.
    * 실제 타입은 변하지 않고, 컴파일 타임에 단순히 치환됩니다.
  + typealias는 새 타입을 만드는 건가요?
    * 아니요. typealias는 새 타입이 아닙니다. 동일한 타입에 다른 이름을 부여하는 것뿐입니다.
    * 예: typealias Email = String → 여전히 String으로 동작합니다.
  + 언제 typealias를 적극 사용하는 것이 좋을까요?
    * 복잡한 함수 타입, 제네릭 타입을 반복해서 사용할 때
    * 도메인 개념에 맞춰 코드의 표현력과 명확성을 높이고 싶을 때
    * 공통 콜백 타입이나 인터페이스 구조를 읽기 쉬운 코드로 만들고 싶을 때 


---


### 고차 함수
- 개념 및 정의
  + 함수를 인자로 받거나 함수를 반환하는 함수
  + 함수는 일급 객체이므로 변수처럼 전달 가능 -> 함수도 데이터처럼 다룰 수 있음
  + ```kotlin
    fun orderFunction (action : () -> Unit) {
        action()    // 전달받은 함수를 실행 
    }
    ```

- 고차함수의 장점
  + | 항목                | 설명                                           |
    | ----------------- | -------------------------------------------- |
    | 🎯 코드 재사용         | 반복되는 로직을 추상화하고, 로직을 외부에서 주입 받음               |
    | 🎯 가독성 향상         | 람다와 함께 사용 시 코드가 간결하고 의도 파악 쉬움                |
    | 🎯 유연한 동작 주입      | 동작을 파라미터로 받아서 다르게 정의 가능 (e.g., 클릭 리스너, 콜백 등) |
    | 🎯 코루틴/Flow/Rx 기반 | Kotlin 비동기 API 대부분이 고차 함수로 구성됨               |

- 고차함수 주의사항
  + | 이슈      | 설명                                    | 해결법                            |
    | ------- | ------------------------------------- | ------------------------------ |
    | 성능      | 함수/람다는 객체 생성 발생 → 힙 메모리 사용            | `inline` 키워드 사용 (컴파일 시 코드로 치환) |
    | 메모리 누수  | 람다가 외부 컨텍스트(Activity 등)를 캡처할 경우 GC 불가 | Lifecycle-aware 또는 WeakRef 사용  |
    | 디버깅 어려움 | 람다 중첩 시 콜스택 추적 어려움                    | 람다를 이름 있는 함수로 분리               |

- 사용 예제
  + 콜백 패턴
    * ```kotlin
      fun requestPermission(onGranted: () -> Unit, onDenied: () -> Unit) {
        val granted = true // 예시
        if (granted) onGranted() else onDenied()
      }

      requestPermission(
          onGranted = { println("허용됨") },
          onDenied = { println("거부됨") }
      )
      ```
  + 리스트 처리
    * ```kotlin
      val names = listOf("Alice", "Bob", "Charlie")

      val upper = names.map { it.uppercase() }
      val filtered = names.filter { it.length <= 4 }

      println(upper)      // [ALICE, BOB, CHARLIE]
      println(filtered)   // [Bob]
      ``` 
  + inline 과 고차 함수
    * inline 키워드를 쓰면 람다 함수 객체가 생성되지 않고, 호출 위치에 코드가 직접 삽입됨 → 성능 최적화
    * ```kotlin
      inline fun log(action: () -> Unit) {
        println("시작")
        action()
        println("끝")
      }
      ```  

- 면접 질문
  + 고차 함수를 사용하는 이유는?
    * 콜백 처리나 UI 이벤트 처리에 유연하게 대응 가능
    * 로직 추상화를 통해 중복 제거
    * 컬렉션 연산 (map/filter 등)을 간결하게 표현 가능
  + 고차 함수에서 inline을 쓰는 이유는?
    * 함수 객체 생성을 피하고, 성능 향상을 위해서입니다. 
    * 람다가 인라인 되면 호출 오버헤드와 힙 할당을 줄일 수 있습니다.
  + 고차 함수 사용 시 주의할 점?
    * 람다에서 외부 컨텍스트를 참조하면 GC 대상에서 제외될 수 있어 메모리 누수 발생 가능
    * 람다 중첩이 심해지면 가독성 저하 → 별도 함수로 추출 권장


---


### mutable & immutable
- 변수 (var / val)
  + | 키워드 | 의미             | 값 변경 가능 여부 | 예시 코드                     |
    |--------|------------------|------------------|------------------------------|
    | `val` | 읽기 전용 (immutable) | ❌ 불가능           | `val name = "Jane"`         |
    | `var` | 읽기/쓰기 가능 (mutable) | ✅ 가능              | `var age = 30` → `age = 31` |
  + ```koltin
    val nickname = "Sky" // ❌ nickname = "Blue" → 컴파일 에러
    var score = 100      // ✅ score = 150 → 정상 작동
    ```

- 컬렉션 (List/Set/Map)
  + 불변 컬렉션 (Immutable) 
    * 수정 불가, 읽기 전용
    * ```kotlin
      val names = listOf("A", "B", "C")
      // names.add("D") ❌ 컴파일 에러
      ``` 
  + 가변 컬렉션 (Mutable) 
    * 요소 추가/삭제 가능
    * ```kotlin
      val names = mutableListOf("A", "B")
      names.add("C") // ✅ 가능
      ``` 
  + | 컬렉션 타입                                    | 수정 가능 여부 |
    | ----------------------------------------- | -------- |
    | `List`, `Set`, `Map`                      | ❌ 불가     |
    | `MutableList`, `MutableSet`, `MutableMap` | ✅ 가능     |
  
- 클래스의 불변성 (Immutable Object)
  + ```kotlin
    // 불변 객체
    data class User(val name: String, val age: Int)

    val user = User("Alex", 20)
    // user.name = "John" ❌ 컴파일 에러
    ``` 
  + ```kotlin
    // 가변 객체
    data class MutableUser(var name: String, var age: Int)

    val mUser = MutableUser("Alex", 20)
    mUser.name = "John" // ✅ 변경 가능
    ```

- 왜 Immutable 이 더 좋은가?
  + | 장점           | 설명                           |
    | ------------ | ---------------------------- |
    | ❄️ 예측 가능성    | 값이 바뀌지 않으므로 디버깅, 테스트가 쉬움     |
    | 🔐 스레드 안전성   | 멀티스레드 환경에서 동기화 없이 안전하게 사용 가능 |
    | ♻️ 참조 공유     | 여러 곳에서 안전하게 공유 가능            |
    | 🧠 함수형 프로그래밍 | 불변 상태는 함수형 패러다임과 자연스럽게 통합됨   |

- 면접 관련 질문
  + val과 var의 차이는?
    * val: 읽기 전용. 초기화 후 값 변경 불가
    * var: 읽기/쓰기 모두 가능
  + val로 선언한 MutableList의 요소는 변경 가능할까?
    * 가능합니다.
    * val은 참조를 바꿀 수 없는 것이지, 참조된 컬렉션의 내용까지 불변이라는 의미는 아님 
    * ```kotlin
      val list = mutableListOf(1, 2, 3)
      list.add(4) // ✅ 가능
      // list = mutableListOf(9, 9, 9) ❌ 불가능 (참조 변경 금지)
      ```
  + 코틀린 `List` 와 `MutableList` 차이점?
    * | 항목      | `List` | `MutableList` |
      | ------- | ------ | ------------- |
      | 읽기      | ✅ 가능   | ✅ 가능          |
      | 쓰기 (수정) | ❌ 불가   | ✅ 가능          |
  + 언제 immutable 을 쓰고 언제 mutable 을 써야 하나요?
    * 기본은 불변으로 설계하고, 정말 필요할 때만 mutable 사용
    * 상태 변화가 불가피한 경우만 var, MutableList 사용
    * 데이터 전달, API 응답, UI 상태 표현 등은 대부분 immutable로 구성하는 것이 안전


---


### init 블록 & constructor()
- `init` 블록
  + | 특징             | 설명                                                                 |
    |------------------|----------------------------------------------------------------------|
    | 실행 시점         | 객체가 생성될 때 **자동으로 실행됨**                                           |
    | 위치             | 클래스 본문 내부, 여러 개 작성 가능                                            |
    | 용도             | 초기값 검증, 로그 출력, 계산 등 **공통 초기화 로직 처리**                           |
    | 실행 순서         | 생성자 호출 후 → `init` 블록 실행                                         |
  + ```kotlin
    class User(val name: String) {        
        init {
            println("init name is $name")
        }
    }
    ```
    
- `constructor()`
  + | 구분     | 설명                                                           |
    | ------ | ------------------------------------------------------------ |
    | 주 생성자  | 클래스 헤더에 정의, **가장 기본적인 생성자**                                  |
    | 보조 생성자 | `constructor` 키워드로 클래스 본문 안에 정의, **다양한 인자 조합을 지원**           |
    | 호출 순서  | 보조 생성자 → 주 생성자 호출 (`: this(...)`) → `init` 블록 → 보조 생성자 블록 실행 |
  + ```kotlin
    class User(val name: String) {
        constructor(name: String, age: Int) : this(name) {
            println("constructor name is $name age is $age")
        }
    }
    ```
    
- 실행 흐름 예시
  + ```kotlin
    class Book(val title: String) {
      init {
          println("📘 Book title: $title") // init 블록
      }

      constructor(title: String, author: String) : this(title) {
          println("✍️ Author: $author") // 보조 생성자
      }

      constructor(title: String, author: String, price: Int) : this(title) {
          println("💰 Author: $author, Price: $price")
      }
    }

    fun main() {
      Book("해리포터")
      Book("해리포터2", "롤링")
      Book("해리포터3", "롤링", 10000)
    }
    /*
    📘 Book title: 해리포터
    📘 Book title: 해리포터2
    ✍️ Author: 롤링
    📘 Book title: 해리포터3
    💰 Author: 롤링, Price: 10000
    */
    ``` 

- 요약
  + | 항목    | init 블록             | constructor() (보조 생성자)             |
    | ----- | ------------------- | ---------------------------------- |
    | 실행 시점 | 생성 시 자동 실행          | 명시적으로 호출해야 실행됨                     |
    | 주 용도  | **공통 초기화** 로직 처리    | **다양한 생성 방식**을 허용                  |
    | 개수 제한 | 여러 개 가능             | 여러 개 가능 (`오버로드` 형태)                |
    | 실행 순서 | 주 생성자 → `init` 실행 순 | 보조 생성자 → 주 생성자 → `init` → 보조 블록 실행 |

- 면접 관련 질문
  + init 블록과 constructor의 차이는?
    * `init`은 공통 초기화 코드
    * `constructor`는 인자 조합을 달리해 다양한 방식으로 인스턴스를 생성할 수 있도록 함
  + init이 constructor보다 먼저 실행되나요?
    * 보조 생성자가 먼저 호출되지만, 실행은 주 생성자의 필드 초기화 → init → 보조 생성자 본문 순서로 진행됨
  + 생성자 파라미터에 검증 로직을 넣고 싶다면?
    * init 블록에서 검증하는 것이 좋음
    * 생성자에선 값만 받고, 검증/로깅/비즈니스 로직은 분리하는 게 유지보수에 유리


---


### Iterable, Sequence
- Iterable
  + `List`, `Set`, `Map` 등 대부분의 컬렉션이 `Iterable<T>` 구현
  + 즉시 계산 방식(Eager evaluation)
  + 각 연산 시 중간 컬렉션 생성
  + 특징
    * `map`, `filter`, `forEach` 등 다양한 확장 함수 제공
    * 컬렉션 전체를 한 연산씩 처리 (ex: `map` 전체 -> `filter` 전체)
    * ```kotlin
      interface Iterable<out T> {
          operator fun iterator(): Iterator<T>
      }

      interface Iterator<out T> {
          operator fun next(): T
          operator fun hasNext(): Boolean
      }

      val numbersList: List<Int> = listOf(1, 2, 3, 4, 5)

      // Iterable의 확장 함수 사용
      val doubledAndEven = numbersList
        .map { it * 2 }       // [2, 4, 6, 8, 10] - 중간 리스트 생성
        .filter { it % 2 == 0 } // [2, 4, 6, 8, 10] - 또 다른 중간 리스트 생성 (이 경우엔 동일)

      // for 루프를 통한 반복 (내부적으로 iterator 사용)
      for (number in numbersList) {
          println(number)
      }  
      ```
  + 장단점
    * | 장점               | 단점                         |
      | ---------------- | -------------------------- |
      | 간단하고 직관적인 구조     | 큰 데이터셋에서 성능 저하 가능          |
      | 컬렉션 전체를 쉽게 반복 가능 | 연산마다 새로운 리스트(중간 컬렉션) 생성됨   |
      | 결과를 즉시 확인 가능     | 연산 체인이 길어질수록 메모리/CPU 부담 증가 |

- Sequence
  + `Sequence<T>`는 지연 계산 방식(Lazy evaluation)
  + `map`, `filter` 등 중간 연산은 실행하지 않고 계획만 수립
    * ```kotlin
      val sequence = sequenceOf(1, 2, 3, 4, 5)

      val result = sequence
          .map {
              println("map: $it")
              it * 2
          }
          .filter {
              println("filter: $it")
              it % 2 == 0
          }

      println("정의만 되었고 아직 실행 안됨")

      val first = result.first() // 연산 시작! 1개만 처리
      ``` 
  + 특징
    * 요소 단위로 전체 파이프라인을 순차 처리
    * 중간 컬렉션을 만들지 않음
    * 최종 연산 (first(), toList(), count() 등) 호출 시 연산이 시작됨
    ```kotlin
    val numbersSequence: Sequence<Int> = sequenceOf(1, 2, 3, 4, 5)
    // 또는 기존 컬렉션을 Sequence로 변환
    // val numbersSequence = listOf(1, 2, 3, 4, 5).asSequence()
    ```
  + 장단점
    * | 장점                 | 단점                                  |
      | ------------------ | ----------------------------------- |
      | 성능 효율적 (중간 컬렉션 없음) | 소규모 연산에서는 약간의 오버헤드 존재               |
      | 큰 데이터, 복잡 연산에서 유리  | 디버깅이 어려울 수 있음                       |
      | **무한 시퀀스 처리 가능**   | 모든 API에서 지원되지 않음 (`randomAccess` 등) |
\
- Iterable vs. Sequence 정리
  + | 항목        | `Iterable`                          | `Sequence`                     |
    | --------- | ----------------------------------- | ------------------------------ |
    | 계산 방식     | 즉시 계산 (Eager)                       | 지연 계산 (Lazy)                   |
    | 중간 컬렉션 생성 | 매 연산마다 생성됨                          | 최종 연산까지 생성되지 않음                |
    | 연산 처리 순서  | 전체 컬렉션 → 각 연산별 처리                   | 각 요소 → 전체 파이프라인 순차 처리          |
    | 예시 메서드    | `.map()`, `.filter()`, `.forEach()` | `.asSequence().map().filter()` |
    | 사용 추천 상황  | 적은 데이터, 결과 즉시 필요할 때                 | 대용량, 복잡한 체인, 무한 처리 필요할 때       |
    | 성능        | 상대적으로 낮음                            | 성능 최적화 가능                      |

- 언제 무엇을 써야 할까?
  + | 상황                             | 추천 타입        |
    | ------------------------------ | ------------ |
    | 단순 변환, 소규모 컬렉션                 | ✅ `Iterable` |
    | 수십만 건 이상, 다단계 연산               | ✅ `Sequence` |
    | 무한 시퀀스 (`generateSequence {}`) | ✅ `Sequence` |
    | 연산 결과를 바로 써야 함                 | ✅ `Iterable` |
    | `first()`, `find()`로 빠르게 추출    | ✅ `Sequence` |

- 면접 관련 질문
  + Iterable과 Sequence의 차이점은?
    * Iterable은 즉시 계산, Sequence는 지연 계산
    * Iterable은 중간 컬렉션 생성, Sequence는 요소 단위 처리로 메모리 효율적
  + Sequence는 언제 쓰는 게 좋은가요?
    * 큰 데이터셋이나 무한 스트림 처리, 또는 중간 리스트 생성을 피하고 싶을 때
    * 예: asSequence().map().filter().first() 


---


### DSL (Domain-Specific Language)
- 정의
  + **도메인 특화 언어** (Domain-Specific Language)
  + Kotlin 문법을 활용해서 **내 도메인에 특화된 코드를 자연어처럼 작성**
  + 일반적인 코드보다 **선언적(declarative)**이고 **가독성 높음** 
  + 대표적인 예로 `Gradle Kotlin DSL`, `Jetpack Compose`, `Anko`, `HTML DSL` 

- 구성 핵심 요소
  + | 요소                  | 설명                                            |
    |---------------------|-----------------------------------------------|
    | `Lambda with receiver` | 람다 블록 내에서 `this`로 대상 객체의 함수/프로퍼티 사용 가능 |
    | `Extension function`   | 기존 타입에 함수를 추가하여 DSL 문법처럼 사용 가능         |
    | `Named arguments`      | 인자명을 명확하게 지정하여 의도를 코드에 드러낼 수 있음     |
    | `Function literals`    | 함수를 값처럼 전달 가능 → 고차함수 기반 DSL 구현 가능       |

- 대표 예제
  + HTML DSL
    * 각 블록은 수신 객체(this)를 가진 람다로 동작
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
  + Gradle Kotlin DSL (build.gradle.kts)
    * plugins {}, repositories {}, dependencies {} 전부 DSL
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
  + Kotlin DSL은 어떤 상황에서 사용하는 게 적절하다고 생각하나요?
    * 특정 도메인 작업을 명확하고 선언적으로 표현하고 싶을 때
    * 반복적 구조, 트리 형태, 구성형 API 등에서 효과적 (예: Gradle, UI, SQL, HTML, JSON 등)
  + Jetpack Compose는 어떻게 DSL로 작동하나요?
    * Compose 함수들은 대부분 수신 객체를 가진 고차 함수
    * 예: Column { Text(...) }는 ColumnScope.() -> Unit 형태
    * 이로써 UI 구조를 중첩함수 + 선언형 스타일로 자연스럽게 표현 가능
  + build.gradle.kts vs build.gradle
    * | 항목     | Groovy DSL (`.gradle`) | Kotlin DSL (`.gradle.kts`) |
      | ------ | ---------------------- | -------------------------- |
      | 언어     | Groovy (동적)            | Kotlin (정적)                |
      | 타입 안정성 | ❌ 없음 (런타임 오류 가능)       | ✅ 컴파일 타임 검사 가능             |
      | IDE 지원 | 제한적 자동완성               | 강력한 자동완성 및 타입 추론           |
      | 유연성    | 문법 유연함 (다소 혼란스러울 수 있음) | 명확한 문법 필요 (컴파일 엄격)         |
      | 학습 난이도 | 진입 장벽 낮음               | Kotlin 익숙해야 쉽게 사용 가능       |


---


### 예외 처리
- 예외란
  + 실행 중 발생하는 오류 상황을 **객체로 표현**
  + 모든 예외는 `Throwable`을 상속
  + ```text
    Throwable
    ├── Error               // 시스템 레벨 심각 오류 (복구 불가)
    └── Exception           // 애플리케이션 수준 예외
        ├── RuntimeException // 주로 프로그래밍 오류
        └── IOException      // 입출력 중 발생
    ```

- 코틀린 예외 처리 특징
  + | 특징                  | 설명                                          |
    | ------------------- | ------------------------------------------- |
    | Unchecked Exception | Java처럼 강제 예외 선언 없음                          |
    | throw는 표현식          | `throw`는 값처럼 동작 가능 (`Nothing` 반환)           |
    | try도 표현식            | `val result = try { ... } catch { ... }` 가능 |

- 예외 발생
  + ```kotlin
    fun checkAge(age: Int) {
      if (age < 0) throw IllegalArgumentException("Age cannot be negative")
    }
    ``` 

- 전제 조건 함수
  + ```kotlin
    fun process(input: String?) {
      require(!input.isNullOrEmpty()) { "Input must not be null or empty" } // IllegalArgumentException
    }

    class User(val id: Int, var name: String) {
      fun rename(newName: String) {
          check(newName.length > 1) { "Name too short" } // IllegalStateException
      }
    }

    fun fail(message: String): Nothing = throw IllegalArgumentException(message)
    val user = name ?: fail("No name!")
    ```
  + | 함수        | 예외 타입                      | 사용 용도                 |
    | --------- | -------------------------- | --------------------- |
    | `require` | `IllegalArgumentException` | 함수 인자 검증              |
    | `check`   | `IllegalStateException`    | 객체 상태 검증              |
    | `error`   | `IllegalStateException`    | 무조건 예외 발생 (불가능한 분기 등) |

- 예외 잡기 (try-catch-finally)
  + ```kotlin
    fun readFile(path: String) {
      var reader: BufferedReader? = null
      
      try {
          reader = File(path).bufferedReader()
          println(reader.readLine())
      } catch (e: FileNotFoundException) {
          println("File not found: $path")
      } catch (e: IOException) {
          println("I/O error: ${e.message}")
      } finally {
          reader?.close()
          println("Always runs!")
      }
    }
    ``` 

- try 표현식
  + ```kotlin
    val message = try {
      "Success"
    } catch (e: Exception) {
      "Failure: ${e.message}"
    }
    ``` 

- 핵심 요약
  + | 개념              | 요약 설명                                        |
    | --------------- | -------------------------------------------- |
    | 예외 구조           | `Throwable` → `Exception` / `Error`          |
    | `throw`         | 표현식 (리턴 값처럼 사용됨) → 보통 `Nothing` 리턴 함수와 함께 사용 |
    | `require/check` | 조건 위반 시 예외 → 명확한 계약(Contract) 문서화 가능         |
    | try-catch       | 예외 발생 가능 코드 감싸기 + 다양한 예외 분기 + 자원 정리까지        |
 
- 면접 관련 질문
  + 코틀린은 Checked Exception을 지원하나요?
    * 아니요. Kotlin은 모든 예외가 Unchecked입니다.
    * Java의 throws와 같은 선언이 필요 없습니다.
  + `require`, `check` 차이점
    * | 함수      | 목적               | 예외 타입                    |
      | ------- | ---------------- | ------------------------ |
      | require | 함수에 넘긴 **인자 검증** | IllegalArgumentException |
      | check   | 객체의 **상태 검증**    | IllegalStateException    |
  + throw는 왜 표현식인가요?
    * throw는 Nothing 타입을 반환하므로, 삼항 연산자나 Elvis (?:) 등과 함께 사용할 수 있어 제어 흐름을 깔끔하게 구성할 수 있습니다.
  + 예외가 발생한 이후 finally는 반드시 실행되나요?
    * 네, finally는 예외 발생 여부와 관계없이 항상 실행됩니다. 
    * 단, System.exit()나 JVM 자체 종료 같은 상황은 예외입니다.


---


### suspend 함수
- 개념 및 정의
  + `suspend` 함수 = **일시 중단 가능한 함수**
  + 코루틴 내에서만 호출 가능
  + **Thread는 차단하지 않고**, 함수 실행만 잠깐 멈췄다가 재개 가능
  + **네트워크 요청, DB 쿼리, delay** 등에서 주로 사용 

- 특징
  + | 항목                | 설명                                                            |
    |-------------------|-----------------------------------------------------------------|
    | 💤 중단 가능           | `delay()`, 네트워크/IO 작업 중 일시 정지 후 재개 가능                         |
    | 🧵 Non-blocking      | 스레드를 멈추지 않음 → **UI 스레드에서 안전**                                 |
    | 🔄 Resume 가능       | 중단된 지점부터 이어서 실행됨                                         |
    | 🚫 일반 함수에서 직접 호출 불가 | `launch`, `async` 같은 **코루틴 블록 내에서만 호출 가능**                 |

- 사용 예제
  + 기본
    * ```kotlin
      suspend fun fetchData(): String {
        delay(1000)  // 1초 대기 (Thread.sleep 아님!)
        return "데이터 완료"
      }

      lifecycleScope.launch {
        val result = fetchData()  // ✅ suspend 함수는 코루틴 안에서 호출
        println(result)
      }
      ```
  + Dispatcher 지정 (withContext)
    * ```kotlin
      suspend fun loadData(): String {
        return withContext(Dispatchers.IO) {
          // IO 작업 수행
          val data = File("file.txt").readText()
          data
        }
      }
      ```
    * withContext() 사용 이유:
      1. 작업에 적합한 스레드 풀에서 실행
      2. Main, IO, Default 등 명확히 분리 가능
  + 병렬 실행 (async)
    * ```kotlin
      suspend fun parallelWork() = coroutineScope {
        val task1 = async { fetchData1() }
        val task2 = async { fetchData2() }
        println("${task1.await()} + ${task2.await()}")
      }
      ```
    * async는 병렬 작업, await()는 결과 대기
    * launch는 결과 없는 단순 실행용

- 일반 함수와 suspend 함수 비교
  + | 항목        | 일반 함수                 | suspend 함수                            |
    | --------- | --------------------- | ------------------------------------- |
    | 중단 가능     | ❌                     | ✅ `delay()`, `IO`, `network` 등에서 일시정지 |
    | 스레드 차단 여부 | 차단 (ex. Thread.sleep) | 비차단 (non-blocking)                    |
    | 실행 환경     | 어디서든 호출 가능            | 코루틴 안에서만 호출 가능                        |
    | 비동기 처리 방식 | Callback, Thread 등 필요 | `async/await` 등으로 간단하게 구현 가능          |

- suspend 호출 위치 예시
  + | 컨텍스트              | 사용 방법                           |
    | ----------------- | ------------------------------- |
    | ViewModel         | `viewModelScope.launch { ... }` |
    | Activity/Fragment | `lifecycleScope.launch { ... }` |
    | 단순 진입점            | `runBlocking { ... }`           |
    | 테스트               | `runTest { ... }` (코루틴 테스트용)    |
 

- 면접 질문
  + suspend 함수란?
    * 일시 중단 가능한 함수입니다.
    * 코루틴 안에서 실행되어야 하며, 스레드는 차단하지 않습니다.
    * 비동기 처리에서 콜백 없이 순차 코드처럼 사용 가능합니다.
  + 일반 함수와 suspend 함수의 차이는 무엇인가요?
    * | 일반 함수      | suspend 함수     |
      | ---------- | -------------- |
      | 블로킹        | 논블로킹           |
      | 중단 불가      | 중단 가능          |
      | 어디서든 호출 가능 | 코루틴 내에서만 호출 가능 |
  + 왜 UI 스레드에서 suspend 함수가 유용한가요? 
    * `Thread.sleep()`과 달리 UI thread를 멈추지 않고 작업을 일시 중단하므로, 앱이 멈추지 않고 부드럽게 동작할 수 있습니다.
  + suspend 함수를 병렬로 실행하려면?
    * async {} 와 await() 사용
    * coroutineScope 내에서 실행


---


### 코루틴 빌더 (runBlocking, launch, async)
- 정의
  + **코루틴 빌더** = 코루틴을 시작하는 진입점 함수
  + 대표: `runBlocking`, `launch`, `async`
  + 모두 내부적으로 새로운 `Job`을 생성하여 코루틴 실행

- runBlocking 빌더
  + | 항목         | 내용                                 |
    |------------|------------------------------------|
    | 목적         | **블로킹 방식**으로 코루틴 실행             |
    | 사용 위치      | `main()`, 테스트 코드 등 진입점               |
    | UI 스레드 사용 | ❌ 금지 (ANR 발생 가능)                   |
    | 리턴 타입      | T (마지막 표현식)                        |
    | 특징         | 블로킹 스코프, 내부 코루틴이 끝날 때까지 현재 스레드 대기 |
  + ```kotlin
    fun main() = runBlocking {
      println("runBlocking start")
          launch {
              delay(1000)
              println("done inside launch")
          }
      println("runBlocking end")
    }
    ```

- launch 빌더
  + | 항목    | 내용                                |
    | ----- | --------------------------------- |
    | 목적    | 결과 없이 **작업 실행** (fire-and-forget) |
    | 리턴 타입 | `Job` (취소, 완료 추적용)                |
    | 사용 예  | UI 업데이트, 로그, 애니메이션 등              |
    | 실행 방식 | 비동기, 블로킹 없음                       |
  + ```kotlin
    GlobalScope.launch {
      delay(1000)
      println("World!")
    }
    Thread.sleep(2000) // main이 먼저 끝나지 않게 대기
    ```

- async 빌더
  + | 항목    | 내용                          |
    | ----- | --------------------------- |
    | 목적    | **결과 반환이 필요한 비동기 작업** 수행    |
    | 리턴 타입 | `Deferred<T>`               |
    | 결과 받기 | `await()` 사용                |
    | 사용 예  | 계산 작업, API 호출 등 병렬 결과 필요할 때 |
  + ```kotlin
    runBlocking {
      val a = async { delay(1000); "A" }
      val b = async { delay(1500); "B" }
      println("${a.await()}, ${b.await()}")
    }
    ```

- async + awaitAll()
  + ```kotlin
    val result = listOf(
      async { delay(300); "One" },
      async { delay(200); "Two" },
      async { delay(100); "Three" }
    ).awaitAll()

    println(result) // ["One", "Two", "Three"]
    ```  
  + `awaitAll()`은 여러 결과를 동시에 기다리고, 첫 예외 발생 시 나머지를 자동 취소해줌.

- 비교 정리
  + | 항목           | launch           | async                       | runBlocking        |
    | ------------ | ---------------- | --------------------------- | ------------------ |
    | 결과 반환        | ❌ 없음             | ✅ `Deferred<T>` → `await()` | ❌ (내부에서 변수로 반환 가능) |
    | 블로킹 여부       | ❌ (비차단)          | ❌ (비차단)                     | ✅ 현재 스레드 블로킹       |
    | 대표 용도        | UI 작업, 로그 등      | API, 계산 결과 등                | 테스트, main 진입점      |
    | 코루틴 범위 설정 가능 | ✅ CoroutineScope | ✅ CoroutineScope            | 자체가 Scope          |

- 면접 관련 질문
  + 코루틴 빌더 중 launch와 async의 차이점은?
    * `launch`는 결과 없이 작업 실행 (`Job`)
    * `async`는 결과 반환 (`Deferred`) → `await()`로 결과 수신
  + `runBlocking`을 언제 쓰나요?
    * `main()` 함수, 테스트 진입점에서만 사용
    * 스레드를 블로킹하므로 UI에서는 절대 사용 금지
  + `async` 병렬 사용하려면?
    * ```kotlin
      val a = async { workA() }
      val b = async { workB() }
      val result = a.await() + b.await()
      ```
  + `awaitAll()`과 `joinAll()`의 차이는?
    * awaitAll()은 여러 Deferred를 한꺼번에 기다리면서, 코드가 깔끔하고 직관적으로 작성되도록 해줍니다.
    * 또한 예외가 발생할 경우, 첫 번째 예외를 즉시 전달하고 나머지 Deferred는 자동으로 취소되기 때문에 안정적인 병렬 처리를 보장할 수 있습니다.


---




### Dispatchers & Thread Context & withContext
1. Dispatchers
   - 코루틴이 실행되는 스레드 또는 스레드풀을 지정하는 역할
   - (코루틴을 어디서 실행시킬지 지정하는 스케줄)
   + | Dispatcher             | 특징                                                         |
     |------------------------|------------------------------------------------------------|
     | Dispatchers.Default    | CPU 집중 작업에 적합 (ex) 정렬, 계산). 기본적으로 core 수 만큼의 스레드로 구성된 풀 사용 |
     | Dispatchers.IO         | 디스크, 네트워크 I/O 등에 적합. 더 많은 스레드를 사용하는 풀                      |
     | Dispatchers.Main       | 안드로이드 UI 스레드. UI 조작은 반드시 이 디스패처에서 수행해야 함                   |
     | Dispatchers.Unconfined | 처음에는 호출한 스레드에서 실행되지만, 중단 후 재개 시점에서는 호출한 스레드에서 계속되지 않을 수 있음 |
   - 예시
     ```kotlin
        launch (Dispatchers.Default) {      // CPU 작업
            doCpuIntensiveTask()
        }
     
        launch (Dispatchers.IO) {           // 네트워크, DB 등 I/O
            val result = fetchDataFromNetwork()     
        }
     ```
     
2. Thread Context (CoroutineContext)
   - 코루틴이 실행될때 사용되는 환경정보의 집합체
   - ("코루틴이 어떻게 실행될 것인가"를 설정하는 모든 요소의 묶음)
   - 구성요소 예시
     + Dispatcher -> 어떤 스레드에서 실행할지
     + Job -> 부모-자식 관계를 만들기 위한 단위
     + CoroutineName -> 디버깅용 이름
     + CoroutineExceptionHandler -> 예외 처리용 핸들러
   - 예시
     ```kotlin
        val context = Dispatchers.IO + CoroutineName("FetchJob")
        // context는 IO Dispatcher에서 실행되며 이름이 "FetchJob"인 코루틴을 생성
     ```
     
3. withContext
   - suspend fun withContext(context: CoroutineContext, block: suspend () -> T): T
   - 현재 코루틴을 일시적으로 다른 컨텍스트 (= Dispatcher 등)로 전환해서 실행
   - 작업이 끝난 후에는 원래 컨텍스트로 자동 복귀
   - (주로 비동기 작업을 명확한 Dispatcher로 전환하고 싶을때 사용)
   - 예시
     ```kotlin
        // I/O 작업을 I/O Dispatcher로 이동
     
        suspend fun loadFile() : String {
            return withContext (Dispatchers.IO) {
                File("data.txt").readText()
            }
        }
     ```
     ```kotlin
        // UI 작업
     
        viewModelScope.launch {
            val result = withContext (Dispatchers.IO) { networkCall() }
            withContext (Dispatchers.Main) { textView.text = result }
        }
     ```
   - withContext 와 launch 의 차이
     * | 항목       | launch                | withContext     |
       |----------|-----------------------|-----------------|
       | 반환값      | 없음 (Job)              | 결과를 반환          |
       | 용도       | 병렬 실행 fire-and-forget | 컨텍스트 전환 및 결과 반환 |
       | 중단 가능 여부 | suspend 아님            | suspend 함수      |
   - 예제
     ```kotlin
        fun main() = runBlocking {
            println("🟢 main start: ${Thread.currentThread().name}")

            val result = withContext (Dispatchers.IO) {
                println("🔵 IO 작업 실행 중: ${Thread.currentThread().name}")
                delay(500)
                "데이터"
            }
     
            println("🟢 다시 메인으로: ${Thread.currentThread().name}")
            println("📦 결과: $result")
        }
     ```
   - 요약
     + | 개념               | 설명                                                       |
            |------------------|----------------------------------------------------------|
       | Dispatcher       | 어떤 스레드(풀)에서 코루틴을 실행할지 지정                                 |
       | CoroutineContext | Dispatcher 외에도 Job, Name, ExceptionHandler 등을 포함하는 실행 환경 |
       | withContext      | 특정 컨텍스트를 일시적으로 전환하여 suspend 함수 실행, 결과 반환                 |

- 질문
  + withContext 와 launch의 차이는 무엇인가요?
    * launch는 Job을 반환하고 결과값이 없으며, 병렬처리를 위해 사용됩니다.
    * withContext는 suspend 함수로 결과값을 반환하고, 새로운 context에서 코드를 순차적으로 실행합니다
    * withContext는 주로 전환이 필요할 때, launch는 병렬 작업이나 fire-and-forget 용도로 사용됩니다.
  + UI 스레드에서 무거운 연산을 하면 어떤 문제가 발생하나요? 해결방안은 무엇인가요?
    * UI 스레드에서 무거운 연산을 하면 ANR(Application Not Responding)이 발생할 수 있습니다.
    * Dispatchers.Default 또는 Dispatchers.IO로 context를 전환하여 백그라운드에서 작업하고, UI 업데이트 시에는 다시 Dispatchers.Main으로 돌아와야 합니다.
    ```kotlin
        viewModelScope.launch {
            val result = withContext (Dispatchers.Default) { doHeavyWork() }
    
            withContext (Dispatchers.Main) { updateUI(result) }
        }
    ```
  + Coroutine에서 Dispatcher란 무엇인가요?
    * Dispatcher는 코루틴이 어떤 스레드나 스레드 풀에서 실행될지를 결정하는 요소입니다. 
    * Dispatchers.Main은 UI 스레드, Dispatchers.IO는 네트워크나 디스크 I/O 작업, Dispatchers.Default는 CPU 연산에 적합한 스레드 풀에서 실행됩니다.


---


### CoroutineScope & Scope & Job 관리
- CoroutineScope 정의
  + 코틀린 코루틴에서 구조화된 동시성을 구현하는 핵심요소 입니다.
  + CoroutineScope는 새로운 코루틴을 시작하고 이들의 생명주기를 관리하는 범위를 정의합니다.

- CoroutineScope 개념
  + CoroutineScope는 CoroutineContext의 한 종류로, 특히 Job을 포함하여 코루틴의 생명주기를 제어합니다. 
  + CoroutineScope 내에서 시작된 모든 코루틴은 해당 스코프의 Job의 자식으로 관리됩니다.

- CoroutineScope 주요역할
  + 코루틴의 생명주기 관리
    * CoroutineScope는 자신이 관리하는 모든 자식 코루틴의 생명주기를 추적합니다.
    * 스코프가 취소되면(scope.cancel()), 해당 스코프 내에서 실행 중이던 모든 자식 코루틴도 함께 취소됩니다. 
      이는 리소스 누수를 방지하고 코루틴을 깔끔하게 정리하는 데 매우 중요합니다. 
    * Android의 ViewModel에서 viewModelScope를 사용하면, ViewModel이 소멸될 때 viewModelScope도 취소되어 관련된 모든 코루틴이 자동으로 정리됩니다.
  + 구조화된 동시성 제공
    * "구조화된 동시성"은 코루틴이 명확한 부모-자식 관계를 가지며, 부모 코루틴(또는 스코프)이 모든 자식 코루틴의 완료를 기다리거나 함께 취소될 수 있도록 하는 프로그래밍 모델입니다.
    * CoroutineScope는 이러한 구조를 만들어, 코루틴이 "어디선가 백그라운드에서 떠도는" 것을 방지하고, 예측 가능하며 관리하기 쉬운 동시성 코드를 작성할 수 있게 합니다.
    * 부모 스코프의 Job이 완료되기를 기다리면(scope.coroutineContext[Job]?.join()), 모든 자식 코루틴이 완료될 때까지 기다리게 됩니다.
  + 코루틴 컨텍스트 상속 및 재정의
    * CoroutineScope는 자신만의 CoroutineContext를 가집니다. 이 컨텍스트에는 Job, CoroutineDispatcher, CoroutineName, CoroutineExceptionHandler 등이 포함될 수 있습니다.
    * 스코프 내에서 새로운 코루틴을 시작할 때 (launch, async 등), 자식 코루틴은 기본적으로 부모 스코프의 컨텍스트를 상속받습니다.
    * 필요에 따라 코루틴 빌더에 특정 컨텍스트 요소를 전달하여 부모의 컨텍스트를 재정의하거나 추가할 수 있습니다.
  + 코루틴 빌더의 수신 객체
    * launch, async와 같은 코루틴 빌더는 CoroutineScope의 확장 함수로 정의되어 있습니다. 따라서 이러한 빌더는 CoroutineScope 인스턴스 내에서 호출되어야 합니다.

- CoroutineScope 생성 방법
  + CoroutineScope() 팩토리 함수
    * CoroutineScope(coroutineContext: CoroutineContext): 주어진 CoroutineContext를 사용하여 새로운 스코프를 만듭니다. 일반적으로 Job()과 Dispatcher를 결합하여 컨텍스트를 구성합니다.
    ```kotlin
        val scope1 = CoroutineScope(Job() + Dispatchers.Main)
    ```
  + MainScope() 팩토리 함수
    * UI 관련 작업에 적합한 스코프를 만듭니다. 내부적으로 SupervisorJob()과 Dispatchers.Main을 사용합니다.
    ```kotlin
        val mainUiScope = MainScope() // SupervisorJob() + Dispatchers.Main
    ```
  + Android Jetpack의 생명주기 스코프
    * viewModelScope: ViewModel 클래스의 확장 프로퍼티로, ViewModel이 소멸될 때 자동으로 취소됩니다. 
    * lifecycleScope: LifecycleOwner (예: Activity, Fragment)의 확장 프로퍼티로, 해당 생명주기 객체가 소멸될 때 자동으로 취소됩니다.
  + GlobalScope
    * 애플리케이션 전체 생명주기를 가지는 전역 스코프입니다. 
    * GlobalScope에서 시작된 코루틴은 애플리케이션이 종료될 때까지 계속 실행될 수 있으며, 구조화된 동시성의 이점을 제공하지 않아 리소스 누수나 관리의 어려움을 초래할 수 있습니다. 
    * 특별한 경우가 아니면 사용을 지양해야 합니다.
  + coroutineScope { ... } 빌더
    * 이것은 새로운 CoroutineScope 인스턴스를 직접 만드는 것과는 약간 다릅니다. coroutineScope 빌더는 새로운 자식 스코프를 생성하고, 그 안의 모든 자식 코루틴이 완료될 때까지 현재 코루틴을 일시 중단합니다.
    * 주로 여러 코루틴을 병렬로 실행하고 모두 완료되기를 기다리거나, 특정 작업 그룹에 대한 예외 처리를 통합하려는 경우에 사용됩니다. 
    * 부모 코루틴의 컨텍스트를 상속받지만, Job은 새로 생성하여 자식 코루틴들을 관리합니다. 
    * 만약 이 내부 스코프에서 예외가 발생하면 해당 스코프와 그 자식들만 취소되고, 외부 스코프에는 영향을 미치지 않거나 예외를 전파할 수 있습니다.

- CoroutineScope 사용 시 주의사항
  + **적절한 생명주기에 맞는 스코프 사용**: Android에서는 viewModelScope나 lifecycleScope처럼 컴포넌트의 생명주기와 연동된 스코프를 사용하는 것이 중요합니다.
  + **스코프 취소**: 더 이상 필요하지 않은 스코프는 반드시 cancel()을 호출하여 관련된 모든 코루틴을 정리해야 합니다. (Jetpack 스코프는 자동으로 처리됨)
  + **Job과 SupervisorJob**: 일반 Job은 자식 코루틴 중 하나라도 실패하면 다른 모든 자식과 부모 Job까지 취소시킵니다. 
    반면 SupervisorJob은 자식 코루틴의 실패가 다른 자식이나 부모에게 영향을 주지 않도록 합니다. 
    UI 관련 작업이나 독립적인 여러 작업을 관리할 때 SupervisorJob이 유용할 수 있습니다. 
    MainScope()나 viewModelScope는 내부적으로 SupervisorJob을 사용합니다.

- Scope 정의
  + 다양한 컨텍스트에서 사용될 수 있지만, 일반적으로 변수, 함수, 클래스 등 식별자(identifier)가 유효하게 참조될 수 있는 코드상의 범위 또는 영역을 의미합니다.

- Scope의 주요역할
  + 이름 충돌 방지
    * 서로 다른 스코프에서는 동일한 이름의 변수나 함수를 선언할 수 있게 해줍니다.
    * 각 스코프는 독립적인 이름 공간을 가지므로, 한 스코프의 변수가 다른 스코프의 동일한 이름의 변수와 충돌하지 않습니다.
  + 가시성 및 접근 제어
    * 스코프는 특정 식별자가 코드의 어느 부분에서 보이고 접근 가능한지를 결정합니다.
    * 일반적으로 내부 스코프에서는 외부 스코프의 식별자에 접근할 수 있지만, 외부 스코프에서는 내부 스코프의 식별자에 직접 접근할 수 없습니다.
    * 이는 정보 은닉의 기초가 되며, 코드의 모듈성과 안정성을 높입니다.
  + 메모리 관리
    * 많은 프로그래밍 언어에서 변수의 생명주기는 해당 변수가 선언된 스코프와 밀접하게 관련됩니다.
    * 변수는 스코프에 진입할 때 메모리에 할당되고, 스코프를 벗어날 때 메모리에서 해제될 수 있습니다.
  + 코드의 구조화 및 가독성 향상
    * 스코프를 통해 코드를 논리적인 블록으로 나눌 수 있으며, 이는 코드의 구조를 명확하게 하고 이해하기 쉽게 만듭니다.
    * 변수나 함수의 영향 범위를 제한함으로써 코드의 특정 부분에 집중하여 분석하고 수정하기 용이해집니다.

- Job 정의
  + 실행 중인 코루틴의 핸들 또는 생명주기를 나타내는 객체입니다.
  + 코루틴이 시작될 때마다 해당 코루틴을 대표하는 Job인스턴스가 생성되며, 이를 통해 코루틴의 상태를 추적하고 제어할 수 있습니다.

- Job의 주요역할
  + 생명주기 관리
    * New (생성됨): Job이 생성되었지만 아직 활성화되지 않은 상태 (예: CoroutineStart.LAZY로 시작된 경우). 
    * Active (활성): 코루틴이 현재 실행 중이거나 일시 중단된 상태.
    * Completing (완료 중): Job의 모든 자식 코루틴이 완료되었지만, Job 자체의 완료 작업이 아직 진행 중인 상태.
    * Completed (완료됨): Job이 성공적으로 완료된 상태.
    * Cancelling (취소 중): Job에 취소 요청이 들어왔고, 취소 작업이 진행 중인 상태.
    * Cancelled (취소됨): Job이 취소된 상태.
  + 취소
    * cancel(cause: CancellationException? = null): 해당 Job과 이 Job에 속한 모든 자식 Job들에게 취소 요청을 보냅니다.
  + 대기
    * join(): Job이 완료될 때까지 (성공, 실패, 취소 모두 포함) 현재 코루틴을 일시 중단합니다. 이는 특정 코루틴의 작업이 끝날 때까지 기다려야 하는 경우에 유용합니다.
  + 부모-자식 관계 및 구조화된 동시성
    * Job은 계층 구조를 가질 수 있습니다. 한 코루틴(부모 Job) 내에서 다른 코루틴(자식 Job)을 시작하면, 자식 Job은 부모 Job의 하위 항목이 됩니다.
    * 부모 Job이 취소되면 모든 자식 Job도 재귀적으로 취소됩니다. 이는 구조화된 동시성의 핵심 원칙 중 하나로, 리소스 누수를 방지하고 코루틴의 생명주기를 예측 가능하게 만듭니다.
    * 부모 Job은 기본적으로 모든 자식 Job이 완료될 때까지 완료되지 않습니다.
  + 예외 처리
    * 자식 코루틴에서 처리되지 않은 예외가 발생하면, 해당 예외는 부모 Job으로 전파됩니다.
    * 기본적으로 부모 Job은 예외가 발생하면 자신과 다른 모든 자식 코루틴을 취소시킵니다.
    * CoroutineExceptionHandler를 CoroutineContext에 등록하여 이러한 예외를 처리할 수 있습니다.
  + 자식 Job 접근 및 관리
    * children: Sequence<Job>: 해당 Job의 직접적인 자식 Job들의 시퀀스를 반환합니다. 이를 통해 자식들의 상태를 확인하거나 개별적으로 제어할 수 있습니다. 
      
- | 특징       | 일반 스코프 (Scope)                          | Job (Kotlin Coroutines)                 | CoroutineScope (Kotlin Coroutin   | 
  |----------|-----------------------------------------|-----------------------------------------|-----------------------------------| 
  | 핵심 정의    | 식별자(변수, 함수 등)가 유효하게 참조될 수 있는 코드상의 영역/범위 | 개별 코루틴의 핸들 또는 생명주기를 나타내는 객체             | 새로운 코루틴을 시작하고 이들의 생명주기를 관리하는 범위   | 
  | 주요 관심사   | 식별자의 가시성, 생명주기, 이름 충돌 방지                | 단일 코루틴의 상태 추적, 실행 제어(취소, 대기), 부모-자식 관계  | 여러 코루틴의 그룹화, 일괄적인 생명주기 관리, 컨텍스트 제공 | 
  | 목적       | 코드 구조화, 메모리 관리, 접근 제어                   | 개별 코루틴의 제어 및 상태 확인                      | 구조화된 동시성, 리소스 누수 방지, 코루틴 실행 환경 제공 | 
  | 생성/형성 방식 | 언어의 문법적 구조 (블록 {} , 함수, 클래스 등)          | launch, async 등 코루틴 빌더 호출 시 반환          | CoroutineScope() 팩토리 함수, viewModelScope, lifecycleScope, coroutineScope { } 빌더 등 |
  | "취소" 개념  | 직접적인 "취소" 개념 없음 (스코프를 벗어나면 식별자 해제)      | cancel(): 해당 Job 및 그 자식 Job들에게 취소 요청    | cancel(): 스코프 내 모든 코루틴(Job) 일괄 취소 | 
  | 생명주기     | 식별자가 선언된 영역의 실행 흐름에 따름                  | New, Active, Completing, Completed, Cancelling, Cancelled | 스코프 자체의 생명주기 (예: ViewModel 소멸 시 viewModelScope 취소) | 
  | 주요 기능/연산 | - (언어 규칙에 의해 정의됨)                       | join(), cancel(), isActive, isCompleted, isCancelled | launch(), async(), cancel() (스코프의 Job을 통해) | 
  | 비유       | 도구나 재료를 사용할 수 있는 "작업 공간" (방, 책상)        | 특정 작업을 수행하는 "개별 작업자" 또는 "작업 지시서" | 여러 작업자(코루틴)를 관리하는 "프로젝트 관리 사무실"   |
   

---


### 플랫폼 타입
- 개념
  + 플랫폼 타입(platform type)은 코틀린이 자바 코드를 호출할 때
    * Java 의 Nullable 여부를 정확히 알 수 없기 때문
    * Kotlin 대신 `nullable` / `non-null`을 추론하지 않고 보류하는 타입
  + 이 값이 null 이 될 수도 있고 아닐 수도 있은 개발자가 책임지고 처리하라 라는 의도를 가지는 타입
    * Kotlin 문법상으로는 T! 라고 표현되며 실제 코드에서는 T!가 표기되지는 않지만 컴파일러가 내부적으로 구분해ㅓ서 다룸

- 왜 플랫폼 타입이 생겼는가?
  + Kotlin & Java 는 100% 상호운용을 목표로 하고 있지만 Java 는 @Nullable / @NotNull 같은 어노테이션이 없으면 타입에 null 가능 여부 정보가 없다
  + Kotlin 은 null 안정성을 엄격히 체크하지만 Java 에서 넘어오는 타입의 null 안정성 보장할 수 없음
  + 개발자에게 책임을 맡긴 플랫폼 타입을 도입

- 샘플코드
  + ````java
    public class User {
        public String getName() {
            return null; // 실수로 null 리턴해도 java 는 컴파일 오류 없음
        }
    }
    ````
  + ```kotlin
    val user = User()
    val name = user.name // platform type: String! -> null 체크 강제 안됨
    println(name.length) // NPE 발생 가능 (코틀린이 강제 체크 못함)
    ```
  + user.name 의 타입은 String! 으로 플랫폼 타입으로 처리되며 null이 올 수도 있다는 사실을 코틀린이 보류하고 있음

- 결론
  + 플랫폼 타입은 코틀린과 자바를 연결하기 위한 타협
  + 코틀린이 null 안정성을 유지하면서도 Java 코드를 그대로 쓸 수 있도록 
    * Nullable 여부가 불확실할 때는 플랫폼 타입으로 처리
    * 개발자가 명확하게 null 체크 직접 해줘야 함
    * Java 코드에서 annotation 으로 명확한 정보 표시

- 면접 관련 질문
  + 플랫폼 타입이 Kotlin에 왜 필요한가요?
    * Java 타입 시스템에는 null 안정성 정보가 없기 때문에 코틀린이 자바의 타입을 바로 단정할 수 없음
    그래서 T! 형태의 플랫폼 타입으로 두고 개발자가 책임지고 null 처리 하도록 한 것
  + 플랫폼 타입을 방치하면 어떤 문제가 발생할 수 있나요?
    * 플랫폼 타입은 컴파일러가 null 검사 강제를 하기 않기 때문에 Java 코드에서 null 넘어와도 코틀린
    쪽에서 NPE(NullPointerException) 터질 수 있음. 따라서 플랫폼 타입을 사용할 때는 반드시 null 검증 습관 필요
  + 플랫폼 타입을 null-safe 하게 다루는 방법이 있을까요?
    * safe call(?.), elvis 연산자(?:), 명시적인 null 체크(if value != null)
    * 코틀린 측에서 직접 안전하게 다루거나 자바 코드에 어노테이션(@Nullable, @NotNull) 추가해서
    코틀린이 플랫폼 타입을 올바르게 추론하게 해야합니다.


---


### 가변인자(vararg)
- 정의
  + 코틀린에서 가변인자는 함수를 호출할 때 동일한 타입의 인자를 여러 개 전달하거나, 아예 전달하지 않을 수도 있도록 하는 기능입니다.
  + 함수를 정의할 때 파라미터 이름 앞에 vararg 키워드를 붙여서 사용합니다.
- 기본사용법
  ```kotlin
    fun printNumbers(vararg numbers: Int) {
        for (number in numbers) {
            print("$number ")
        }
        println()
    }

    fun main() {
        printNumbers(1, 2, 3)       // 출력: 1 2 3
        printNumbers(4, 5, 6, 7, 8) // 출력: 4 5 6 7 8
        printNumbers()              // 출력: (아무것도 출력 안 함, 빈 줄)
    }
  ```
- 주요 특징 및 설명
  + **vararg 키워드:** 함수 파라미터를 선언할 때 타입 앞에 vararg를 붙입니다.
  + **내부적으로 배열(Array)로 처리:** vararg 파라미터는 함수 내부에서 해당 타입의 배열로 취급됩니다. 위 printNumbers 예제에서 numbers는 Array<Int> 타입입니다. 따라서 배열과 관련된 모든 연산(반복, 인덱스 접근 등)을 사용할 수 있습니다. 
  + **인자 전달 방식:**
    * 쉼표로 구분하여 여러 개 전달: myFunction("a", "b", "c")
    * 아무것도 전달하지 않음: myFunction() (이 경우 함수 내부에서는 빈 배열로 처리됨)
    * 이미 배열이 있는 경우 (스프레드 연산자 *): 이미 생성된 배열의 요소들을 가변인자로 전달하고 싶을 때는 배열 이름 앞에 스프레드(spread) 연산자 * 를 사용합니다.
    ```kotlin
    fun printValues(vararg values: String) {
            values.forEach { println(it) }
        }

        fun main() {
            val messages = arrayOf("Hello", "World")
            printValues(*messages) // 배열의 각 요소를 개별 인자로 전달
            // 위 코드는 printValues("Hello", "World")와 동일하게 동작합니다.

            // 스프레드 연산자 없이 배열을 직접 전달하면 컴파일 오류 발생
            // printValues(messages) // 오류!
        }
    ```
  + **위치 제약:**
    * 하나의 함수에는 하나의 vararg 파라미터만 허용됩니다.
    * vararg 파라미터는 일반적으로 파라미터 목록의 가장 마지막에 위치하는 것이 좋습니다. 만약 vararg 파라미터 뒤에 다른 파라미터가 온다면, 해당 파라미터에 값을 전달할 때 이름 있는 인자(named argument)를 사용해야 합니다.
    ```kotlin
    fun processData(vararg values: Int, multiplier: Int) {
            for (value in values) {
                println(value * multiplier)
            }
        }

        fun main() {
            // vararg 뒤의 파라미터는 이름 있는 인자로 전달해야 함
            processData(1, 2, 3, multiplier = 10)
            // processData(multiplier = 10, 1, 2, 3) // 이렇게는 안됨 (vararg가 먼저 와야 함)
            // processData(1, 2, 3, 10) // 오류! 10이 values에 포함될지 multiplier에 갈지 모호함
        }
    ```
  + **제네릭과 함께 사용:** vararg는 제네릭 타입과 함께 사용될 수 있습니다.
    ```kotlin
    fun <T> printAll(vararg items: T) {
        for (item in items) {
            println(item)
        }
    }

    fun main() {
        printAll("apple", "banana", "cherry")
        printAll(10, 20, 30)
        printAll(true, false, true)
    }
    ```
    
- vararg를 사용하는 경우:
  + 함수가 가변적인 수의 인자를 받아야 할 때 (예: String.format, 로깅 함수, 컬렉션 빌더 함수 등)
  + 인자의 개수가 명확하지 않거나, 0개부터 여러 개까지 유연하게 처리해야 할 때 

- 장점:
  + 유연성: 함수 호출 시 전달하는 인자의 개수를 자유롭게 조절할 수 있습니다. 
  + 간결성: 여러 개의 오버로딩된 함수를 만들 필요 없이 하나의 함수로 다양한 인자 개수를 처리할 수 있습니다. 

- 주의사항:
  + vararg 파라미터는 내부적으로 배열을 생성하므로, 매우 빈번하게 호출되거나 성능에 민감한 코드에서는 약간의 오버헤드가 발생할 수 있습니다. 하지만 대부분의 경우 이는 무시할 만한 수준입니다. 
  + Java와의 상호운용성: 코틀린의 vararg는 Java의 가변인자(...)와 호환됩니다. 가변인자는 코틀린에서 함수를 더 유연하고 편리하게 작성할 수 있도록 도와주는 강력한 기능입니다.
    
    



___




### Operator Overloading
- operator 키워드를 붙히면 Kotlin이 해당 메서드를 **특수한 연산자로 해석**
- ex) plus() 함수에 operator 붙히면 -> + 연산자로 사용 가능

1. 기본 문법
   ```kotlin
        operator fun plus(other : T) : T
   ```
   + ex) + 연산자 오버로딩
   ```kotlin
        data class Point (val x : Int, val y : Int) {
            operator fun plus (other : Point) : Point {
                return Point (x + other.x, y + other.y)
            }
        }
   
        val p1 = Point(1, 2)
        val p2 = Point(3, 4)
        val p3 = p1 + p2    // plus()로 호출됨
        println(p3)     // Point(x=4, y=6)
   ```
   
2. 오버로딩 가능한 연산자
   * | 연산자 | 함수 이름     | 예시 함수 시그니처                             |
            |-----|-----------|----------------------------------------|
     | +   | plus      | operator fun plus(other: T): T         |
     | -   | minus     | operator fun minus(other: T): T        |
     | *   | times     | operator fun times(other: T): T        |
     | /   | div       | operator fun div(other: T): T          |
     | %   | rem       | operator fun rem(other: T): T          |
     | []  | get, set  | operator fun get(index: Int): T        |
     | ==  | equals    | operator fun equals(other: Any?): Boolean |
     | !=  | equals 사용 | 위와 동일                                  |
     | ++  | inc       |   operator fun inc(): T     |
     | --  | dec       |       operator fun dec(): T    |
     | ()  | invoke    |   operator fun invoke(): T     |
     | in  | contains  |     operator fun contains(value: T): Boolean     |
     | ..  | rangeTo   |   operator fun rangeTo(other: T): ClosedRange<T>     |

3. 주요예시
   - [] 오버로딩 : get, set
   ```kotlin
        class MyList {
             private val data = mutableListOf(1, 2, 3)

             operator fun get(index: Int): Int = data[index]
             operator fun set(index: Int, value: Int) {
                  data[index] = value
             }
        }

        val list = MyList()
        println(list[0])     // get 호출 → 1
        list[0] = 10         // set 호출
   ```
   - == 오버로딩 : equals
   ```kotlin
        data class User(val name: String) {
             override operator fun equals(other: Any?): Boolean {
                    return (other is User) && other.name == name
             }
        }

        val u1 = User("Tom")
        val u2 = User("Tom")
        println(u1 == u2)  // true → equals 호출됨
   ```
   - () 오버로딩 : invoke
   ```kotlin
        class Greeter(val message: String) {
             operator fun invoke(name: String) {
                    println("$message, $name!")
             }
        }

        val g = Greeter("Hello")
        g("Bae")  // invoke 호출 → Hello, Bae!
   ```
   
4. 주의사항
   - operator 키워드는 **정해진 함수 이름**에서만 사용 가능
   - equals를 오버로딩 하면 반드시 hashCode()를 재정의 해야함


---


### infix 함수
- 개념/정의
  + 점(.) or 괄호 없이 중위 포기법으로 호출할 수 있는 함수
  + 마치 연산저처럼 자연스러운 문장형 코드 스타일 (ex a.plus(b) -> a plus b)

- 사용법
  + 선언 시 `infix` 키워드
  + 수신 객체에 대해 하나의 파라미터만 받는 멤버 함수나 확장 함수여야 함
  + vararg or 디폴트 값 매개변수는 불가
  + ```kotlin
    infix fun Int.multiply(x:Int): Int {
        return this * x
    }
    println(5 multiply 3) // 15
    ```

- 특징
  + 점(.) 연산자를 쓰지 않기에 가독성 높임
  + DSL 스타일 코드에 잘 어울림

- 면접 관련 질문
  + infix 함수 왜 필요한가?
    * 점 표기나 괄호를 줄여서 자연스럽게 표현할 수 있게 하여 코드 가독성을 높임
  + infix 함수 사용 조건?
    * 클래스의 멤버 함수 or 확장함수
    * 하나의 파라미터만 받아야 함
    * vararg, 디폴트 파라미터 사용 불가
  + infix 함수와 operator 함수 차이
    * infix 는 중위 표기범을 위한 키워드이고 operator 는 +, -, * 연산자 오버로딩 위한 키워드


---


### Any 클래스
- 정의
  + 코틀린의 Any 클래스는 자바의 Object 클래스와 유사하게 모든 코틀린 클래스의 최상위 타입입니다.
  + 어떤 클래스를 정의하든, 명시적으로 다른 클래스를 상속하지 않으면 자동으로 Any 클래스를 상속받게 됩니다.

- 주요 특징 및 역할
  + 모든 타입의 루트
    * Any는 코틀린 타입 계층 구조의 정점에 위치합니다. 모든 코틀린 타입은 Any 타입으로 간주될 수 있습니다.
    * 따라서 어떤 타입의 값이든 저장할 수 있는 변수를 선언하고 싶을 때 Any 타입을 사용할 수 있습니다.
    * ```kotlin
      val myVariable: Any = "Hello" // String은 Any 타입으로 취급 가능
      val anotherVariable: Any = 123    // Int도 Any 타입으로 취급 가능
      val yetAnotherVariable: Any = true // Boolean도 Any 타입으로 취급 가능
      ```
  + 기본 메서드 제공
    * equals(other: Any?): Boolean
    * hashCode(): Int
    * toString(): String

  + Nullable 타입 (Any?)
    * 코틀린은 Null 안전성을 중요하게 다루므로, null 값을 가질 수 있는 모든 타입의 참조를 나타내기 위해 Any? 타입을 사용합니다.
    * 반면, Any 타입은 null이 아닌 값만 가질 수 있음을 의미합니다.
    * ```kotlin
      var nonNullable: Any = "Cannot be null" // nonNullable = null // 컴파일 오류!
      var nullable: Any? = "Can be null"
      nullable = null // 허용됨
      ```
      
  + Java Object와의 관계
    * 코틀린이 JVM에서 실행될 때, 코틀린의 Any 타입은 Java의 java.lang.Object 타입으로 컴파일됩니다.
    * 따라서 Java 코드와 상호 운용할 때, Java의 Object를 코틀린에서는 Any (또는 Any?)로 다룰 수 있으며, 그 반대도 마찬가지입니다.
  
- 요약
  + kotlin.Any는 코틀린의 모든 클래스의 최상위 부모 클래스로, 
    모든 객체가 가져야 할 기본적인 메서드(equals, hashCode, toString)를 제공하며, 타입 계층 구조의 루트 역할을 합니다. 
    이를 통해 코틀린은 강력한 타입 시스템과 Null 안전성을 유지하면서도 유연한 프로그래밍을 지원합니다.
    

---


### 반공변성
- 개념 및 정의
  + 타입 계층 구조에서 하위 타입 대신 상위 타입을 받도록 허용하는 제네릭 타입 제약 방식
  + 제네릭 타입 매개변수 앞에 `in` 키워드를 붙여 선언
    * 해당 타입이 입력(소비) 전용 명시
    * T의 부모 타입까지 허용하게 만들 수 있음
  + Int 타입이 Number 하위 자료형일 때 Class Box<in T> 선언 시
    * Class Box<Number>는 Class Box<Int> 의 하위 자료형이 됨
  + ![out_in](../_assets/variance.png)

- 예시
  + ```kotlin
    open class Animal
    class Dog : Animal()
    
    class AnimalConsumer<in T> {
        fun consume(animal: T) {
            println("Consumed: $animal")
        }
    }
    
    val dogConsumer: AnimalConsumer<Dog> = AnimalConsumer<Animal>()  // OK (in 사용 시)
    ```
  + `AnimalConsumer<in T>` T의 부모 타입도 안전하게 대입 가능하도록 만듬
  + 'AnimalConsumer<Animal>' 은 `AnimalConsumer<Dog>`로 대입될 수 있음 -> 반공변성

- 필요한 이유?
  + 함수를 인자로 받는 경우, 그 함수가 더 일반적인(상위) 타입을 소비할 수 있어야 유연한 설계가 가능
  + 반공변성은 이런 상황에서 타입 안정성을 유지하면서도 다형성을 허용하기 위해 필요
  + ```kotlin
    fun feedDogs(consumer:AnimalConsumer<Dog>) {
        consumer.consume(Dog())
    }
    ```
    * 만약 `AnimalConsumer<Dog>` 대신 `AnimalConsumer<Animal>` 을 넘길수 없다면 유연한 구조 불가

- 공변성과의 비교
  * | 개념       | 키워드   | 의미                 | 예시 타입 방향                                   |
    | -------- | ----- | ------------------ | ------------------------------------------ |
    | 공변성      | `out` | 생산 전용 (출력만)        | `List<out Animal>` → `List<Dog>` 허용        |
    | **반공변성** | `in`  | 소비 전용 (입력만)        | `Consumer<in Dog>` → `Consumer<Animal>` 허용 |
    | 무공변      | 없음    | 기본값, in/out 둘 다 불가 | `MutableList<T>` 등                         |

- 면접 관련 질문
  + Kotlin에서 in 키워드는 어떤 의미인가요?
    * 제네릭 타입이 입력(소비) 전용으로 상위 타입으로 대체 가능
  + Kotlin의 in과 Java의 ? super T는 어떤 관계인가요?
    * 동일한 의미로 사용
  + 반공변성과 공변성은 서로 반대 방향이라고 보아도 되나요?
    * 공변성(out) 은 생산(출력) 하기에 하위 타입 허용
    * 반공변성(in) 은 소비(입력) 하기에 상위 타입 허용




---




### companion object
- 클래스 내부에서 인스턴스(객체)를 만들지 않고도 사용할 수 있는 정적 영역
- Java의 static과 비슷하지만, Kotlin은 static 키워드가 없고 **객체 기반으로 대신 구현**

1. 사용 방법
   - 문법
      ```kotlin
         class MyClass {
             companion object {
                 const val PI = 3.14
                
                 fun create() : MyClass {
                     return MyClass()
                 }
             }
         }
      ```
   - 사용 방법
      ```kotlin
         val obj = MyClass.create()
         println(MyClass.PI)
      ```
   | MyClass를 생성하지 않아도 **create나 PI에 접근 가능**

2. companion object의 특징
   + | 항목          | 설명                                         |
                 |-------------|--------------------------------------------|
     | 정적 접근       | 클래스명으로 직접 접근 가능 (MyClass.x)                |
     | 싱글턴         | companion object는 클래스 당 **하나**만 존재         |
     | 이름 지정 가능    | companion object Nane { ... } 처럼 이름 줄 수 있음 |
     | 인터페이스 구현 가능 | 다른 객체처럼 인터페이스 구현 가능                        |
     | 자바와의 호환     | 자바에서는 MyClass.Companion.method()로 접근       |

3. 이름 있는 companion object 예시
    ```kotlin
        class Logger {
            companion object Factory {
                fun create() : Logger = Logger() 
            }
        }
   
        // 사용
        val logger = Logger.create()
    ```
   | 이름 지정하면 Logger.Factory.create()로도 접근 가능

4. 사용
   + | 상황                   | 예시                      |
                      |----------------------|-------------------------|
     | 팩토리 메서드 만들기          | MyClass.create()        |
     | 정적 상수 선언             | const val VERSION = 1.0 |
     | 자바 static과 호환        | @JvmStatic              |
     | 클래스 내부에서 공유해야 할 util | companion object 활용     |

5. 예시 (팩토리 + 상수)
   ```kotlin
        class User private constructor (val name : String) {
            companion object {
                const val DEFAULT_NAME = "Guest"
   
                fun create (name : String?) : User {
                    return User (name ?: DEFAULT_NAME)
                }        
            }    
        }
   
        val name1 = User.create("Bae")
        val name2 = User.create(null)
   ```
   
6. 주의사항
   + | 항목                          | 주의할 점                                        |
                           |-----------------------------|----------------------------------------------|
     | static이 없음                  | 반드시 companion object 사용해야 함                  |
     | const는 최상위 or companion only | const val은 반드시 companion object 또는 top-level |
     | companion 내부는 싱글턴           | 상태(state)를 저장할 경우 thread-safe 고려             |

7. 면접 질문 
   + companion object는 어떤 특징을 가지고 있나요?
     * 클래스 내부에 1개만 선언 가능합니다.(싱글턴 객체)
     * 클래스 명으로 직접 접근 가능합니다. (ex) MyClass.name())
     * 일반 객체처럼 인터페이스 구현이 가능합니다.
   + companion object 내부에서 생성자(private constructor)를 사용할수 있는 이유는 무엇인가요?
     * companion object는 클래스 내부에 있으므로 private constructor에도 접근 가능 하여, 외부에서는 생성자를 숨기고 
       create() 같은 팩토리 메서드만 제공하는 패턴을 구현할 수 있습니다.
   + companion object에 상태(state)를 저장해도 되나요?
     * 가능하지만 companion object는 기본적으로 싱글턴이기 때문에 **여러 스레드에서 동시에 접근하면 race condition이 발생합니다.**
     * 따라서 공유상태가 있다면 적절한 동기화 (ex) @Synchronized, volatile)가 필요합니다.


---


### 싱글턴 패턴 및 동기화
1. 기본 개념
   - object 키워드를 사용하면 **자동으로 싱글턴 객체**가 생성
   - 예시
   ```kotlin
        object Logger {
            fun log (msg: String) {
                println("Log : $msg")
            }
        }
   ```
   ```kotlin
        Logger.log("Hello")
   ```
   - object는 클래스처럼 정의되지만 단 **1개의 인스턴스만 생성됨**
   - 별도 생성자 필요 없음 (new 사용 불가)
   - Java의 enum Singleton 처럼 안전하게 단일 객체 보장

2. Kotli이 object만으로 싱글턴이 가능한 이유
   - Kotlin은 내부적으로 **JVM 클래스 로더를 통해 싱글턴 보장을 수행합니다.**
   - 즉, object는 클래스가 처음 사용될 때 **JVM이 한 번만 초기화**해주기 때문에 **Thread-safe(스레드 안전)** 합니다.

3. Thread-safe 한가?
   - 기본적으로 object는 Thread-safe 함.
   - Kotlin의 object는 JVM 클래스 로딩 규칙에 따라 안전하게 초기화 되므로 **멀티스레드 환경에서 절대 2번 생성되지 않는다.**
   - 모두 안전
   ```kotlin
        object DatabaseManager {
            fun connect() = println("DB 연결")
        }    
   ```
   
4. 상태를 가질 경우 주의할 점
   - object는 싱글턴이기 때문에 내부에 var 같은 **mutable state (변경 가능한 값)**를 두면 
     여러 스레드에서 동시 접근할 경우 **경쟁 상태 (Race condition)**가 발생할 수 있음
   - ex)
   ```kotlin
        object Counter {
            var count = 0
   
            fun increment() {
                count++
            }
        }
   ```
   - 멀티스레드 환경에서는 count++가 Thread-safe 하지 않음 -> **동기화 필요**

5. 동기화 (synchronized) 방법
   - @Sychronized 사용 (JVM 에서 제공)
     ```kotlin
          object SafeCounter {
              var count = 0
   
              @Sychronized
              fun increament() {
                  count++
              }            
          }
     ```
        - Jvm의 Sychronized 블록과 동일하게 작동
        - 단점 : 성능 저하 가능성
     
   - AtomicInteger 등 동시성 클래스 사용
     ```kotlin
          import java.util.concurrent.atomic.AtomicInteger
     
          object AtomicCounter {
              private val count = AtomicInteger(0)
     
              fun increment() {
                  count.incrementAndGet()
              }
     
              fun get(): Int = count.get()
          }
     ```
        - AtomicInteger는 **CAS(Compare-And-Set) 기반**으로 동기화를 처리
        - 빠르고 가벼운 방식 -> **멀티스레드 환경에 권장**

6. Lazy Singleton 만드는 방법 (지연 초기화 된 싱글턴)
   ```kotlin
        class MyService private constructor() {
            companion object {
                val instance : MyService by lazy {
                    MyService()
                }
            }
        }
   ```
    - by lazy는 **첫 호출 시 1번만 초기화**
    - 내부적으로 **Thread-safe**

7. Singleton 구현방식 정리
   * | 방식      | 코드          | Thread-safe   | 특징         |
         |---------|-------------|---------------|------------|
     | object  | object A {} |   ✅     | 가장 간단한 싱글턴 |
     | by lazy |  val x by lazy {}   |    ✅    | 지연 초기화     |
     |  @Synchronized     |   메서드에 어노테이션    |   ✅     |   간단하지만 느림         |
     |  AtomicInteger     |    동시성 클래스     |      ✅     |  빠르고 안정적     |

8. 요약
   + | 항목             | 설명                           |
                                |----------------|------------------------------|
     | 기본 싱글턴         | object                       |
     | 상태 있음 + 동기화 필요 | @Synchronized, AtomicInteger |
     | 늦은 초기화     |    by lazy 사용     |
     |  Thread-safe 여부     |     Kotlin object는 기본적으로 Thread-safe     |

9. 면접 질문
   * kotlin object는 왜 thread-safe 하나요?
     + object는 JVM의 class loader에 의해 최초 1회만 초기화되므로, 여러 스레드가 동시에 접근해도 인스턴스가 중복 생성되지 않습니다.
     + Thread-safe가 기본으로 보장됩니다.
   * Kotlin에서 상태(state)를 가지는 싱글턴은 문제가 없나요?
     + 싱글턴 자체는 Thread-safe지만, 내부에 var처럼 mutable state가 있을 경우 Race Condition이 발생할 수 있습니다.
     + 따라서 동기화가 필요하거나, AtomicInteger 등의 동시성 도구를 사용해야 안전합니다.
   * @Synchronized 키워드는 어떤 역할을 하나요?
     + Kotlin에서 @Synchronized는 Java의 synchronized와 동일하게 작동하며, 해당 메서드를 동시에 하나의 스레드만 실행할 수 있게 합니다.
     + 단점은 성능 저하가 발생할 수 있습니다.


---


### 공변성
- 정의
  + Kotlin에서 **공변성(Covariance)**은 제네릭 타입 시스템의 중요한 개념으로,
    타입 계층 구조에서 상위 타입이 필요한 곳에 하위 타입을 안전하게 사용할 수 있도록 허용하는 것을 의미합니다.
    쉽게 말해, List<Dog>를 List<Animal> 타입의 변수에 할당할 수 있게 하는 것입니다 (단, Dog는 Animal의 하위 타입이라고 가정).
    Kotlin에서는 out 키워드를 사용하여 공변성을 선언합니다. out 키워드와 공변성 제네릭 클래스나 인터페이스의 타입 파라미터 앞에 out 키워드를 붙이면,
    해당 타입 파라미터는 **공변적(covariant)**이라고 선언됩니다.
  + ```kotlin
    interface Source<out T> {
        fun nextT(): T // T를 생산(반환)만 함
    }

    fun useSource(strs: Source<String>) {
        val objects: Source<Any> = strs // 이것이 가능! (String은 Any의 하위 타입)
        val obj: Any = objects.nextT() // 반환된 String을 Any로 받을 수 있음
    }
  ```

- out의 의미와 제약 조건
  + 생산자(Producer) 역할
    * out으로 선언된 타입 파라미터 T는 해당 클래스/인터페이스의 멤버 함수에서 오직 반환 타입(out-position)으로만 사용될 수 있습니다.
    * T 타입의 값을 "생산"만 할 수 있습니다.
  + 소비자(Consumer) 역할 불가
      * T 타입의 값을 함수의 매개변수(in-position)로는 사용할 수 없습니다.
      * T 타입의 값을 "소비"할 수 없습니다.

- 공변성이 필요한 이유와 장점
  + 유연성 향상
    * 더 일반적인 타입(상위 타입)을 기대하는 함수나 컬렉션에 특정 타입(하위 타입)의 객체를 안전하게 전달할 수 있게 되어 코드의 유연성과 재사용성이 높아집니다.
  + 타입 안전성 유지
    * out 키워드의 제약 조건 덕분에, 공변성을 사용하더라도 런타임에 타입 오류가 발생하는 것을 컴파일 시점에 방지할 수 있습니다.
  + Java의 ? extends T 와 유사
    * Kotlin의 Source<out T>는 Java의 Source<? extends T>와 유사한 개념입니다.
    * Java에서는 이를 사용처 분산이라고 하며, 와일드카드를 사용하는 곳에서 공변성을 지정합니다.
    * 반면 Kotlin은 선언처 분산을 지원하여 클래스나 인터페이스를 선언할 때 out (또는 in)을 명시합니다.

- 요약
  + 공변성 (out T): "생산자" 역할. T 타입의 객체를 반환만 할 수 있고, 매개변수로 받을 수는 없습니다.
  + C<하위타입>을 C<상위타입>으로 안전하게 취급할 수 있게 합니다 (예: List<String>을 List<Any>로).
  + 타입 안전성을 유지하면서 코드의 유연성을 높입니다.
  + Kotlin의 List<out E>가 대표적인 예입니다. 공변성은 반공변성(in) 및 무공변성(키워드 없음)과 함께 Kotlin 제네릭 시스템의 중요한 부분을 이루며, 더 안전하고 유연한 코드를 작성하는 데 도움을 줍니다.

- 면접질문
  + 공변성을 사용하면 어떤 이점이 있나요? 왜 필요한가요?
    * 코드 유연성 및 재사용성 향상, 타입 안전성 유지. 더 일반적인 타입을 기대하는 함수에 특정 하위 타입 객체를 안전하게 전달 가능합니다.
  + 실제 코틀린 표준 라이브러리에서 공변성이 사용된 예시를 들어 설명해주세요.
    *  List<out E>, Set<out E>, Map<K, out V>, Iterable<out T>, Sequence<out T>


---

### Unit
- 개념
  + 코틀린에서 Unit은 특별한 의미를 가지는 타입입니다.
  + Java의 void와 유사한 역할을 하지만 몇 가지 중요한 차이점이 있습니다.

- Unit의 주요특징
  + 값을 반환하지 않는 함수의 반환 타입
    * 함수가 명시적으로 어떤 값을 반환하지 않을 때, 그 함수의 반환 타입은 Unit이 됩니다.
    * Java에서 void 키워드를 사용하는 것과 동일한 목적입니다.
    * ```kotlin
      fun printMessage(message: String): Unit { // Unit 반환 타입 명시
        println(message)
      }

      fun Greet() { // 반환 타입 생략 시 컴파일러가 Unit으로 추론
        println("Hello!")
      }
      ```
  + 실제 객체(싱글톤)
    * Java의 void는 단순히 "값이 없음"을 나타내는 키워드인 반면, Kotlin의 Unit은 실제로 object로 선언된 싱글톤 객체입니다.
    * Unit 타입의 값은 단 하나, Unit 객체 그 자체입니다.
    * void와의 가장 큰 차이점 중 하나입니다.
  + 반환 타입 생략 가능
    * 함수의 반환 타입이 Unit일 경우, 명시적으로 : Unit을 적지 않아도 컴파일러가 자동으로 추론합니다.
    * 대부분의 경우 Unit을 직접 코드에 작성할 일은 적습니다.
    * ```kotlin
      fun sayGoodbye() { // 반환 타입이 Unit으로 자동 추론됨
        println("Goodbye!")
      }
      ```
  + 제네릭에서 사용 가능
    * Unit은 실제 타입이기 때문에 제네릭 함수의 타입 인자로 사용될 수 있습니다. 이는 void가 불가능한 부분입니다.
    * 어떤 작업을 수행하고 결과는 반환하지 않지만, 작업 완료 여부 등을 다른 방식으로 처리하는 콜백이나 고차 함수에서 유용하게 사용될 수 있습니다.
  + 명시적인 return Unit 또는 return
    * Unit을 반환하는 함수에서는 return Unit을 명시적으로 작성할 수 있지만, 이는 불필요하며 관용적으로 사용되지 않습니다.
    * 단순히 return 키워드만 사용해도 Unit이 반환되는 것과 동일한 효과를 가집니다 (함수 중간에서 빠져나올 때).
    * ```kotlin
      fun checkAge(age: Int): Unit {
        if (age < 18) {
          println("Minor")
          return // 여기서 함수 종료, Unit 반환
        }
        println("Adult")
      }
      ```

- 왜 void 대신 Unit을 사용할까?
  + 일관성: Kotlin은 모든 것을 객체로 취급하려는 경향이 있습니다. Unit을 객체로 만듦으로써 "값이 없음"조차도 타입 시스템 내에서 일관되게 다룰 수 있습니다.
  + 제네릭 지원: 위에서 언급했듯이, 제네릭 타입 파라미터로 Unit을 사용할 수 있게 되어 함수형 프로그래밍 패턴 등에서 유용합니다.
  + 함수형 프로그래밍과의 조화: 함수형 프로그래밍에서는 모든 함수가 값을 반환하는 것을 선호합니다. Unit은 "의미 있는 값은 없지만, 작업은 완료되었음"을 나타내는 일종의 신호로 볼 수 있습니다. 

- 요약
  + Unit은 Kotlin에서 함수가 아무 값도 반환하지 않음을 나타내는 타입입니다.
  + Java의 void와 유사하지만, Unit은 실제 싱글톤 객체입니다.
  + 반환 타입이 Unit일 경우 생략 가능합니다.
  + 제네릭과 함께 사용될 수 있다는 점이 void와의 중요한 차이점입니다. 일상적인 코딩에서는 Unit을 직접 명시하는 경우는 드물지만, Kotlin의 타입 시스템과 함수형 프로그래밍 패러다임을 이해하는 데 중요한 개념입니다.


---


### 코틀린 & Java 상호운용성
- 상호운용성
    + Kotlin <-> Java 는 100% 상호운용 가능하도록 설계
        * 코틀린 컴파일러는 .kt 파일은 .class (JVM 바이트코드)로 컴파일 함
        * 실행 가능한 Jva 코드와 완전히 동일한 JVM 언어로 변환

- 필요 이유
    + 대부분의 안드로이드 프로젝트는 Java 레거시 코드를 유지하고 또는 코틀린과 섞여있는 경우가 다수
    + 유지보수가 필요하기에 점진적으로 변환 중

- Kotlin -> Java 호출 주요 문법
    + Top-Level function -> 클래스명.function
    + Object -> INSTANCE 로 접근
    + Companion object -> ClassName.Companion
    + @JvmStatic -> static
    + @JvmField -> get/set 없이 필드 접근
    + @JvmOverloads -> 오버로드 자동 생성
    + @file: JvmName("MyUtils") -> 클래스 이름 변경 가능
        * 이름 충돌 가능성 존재

- Java -> Kotlin 호출
    + java 는 null 안전성 정보 없음 -> 코틀린에서 Platform Type(T!) 으로 인식하기에 개발자 처리 필요
    + @Nullable, @NotNull 어너테이션 필요

- 면접 관련 질문
    + Kotlin & Java 는 상호운용 가능한가?
        * 둘다 JVM 위에서 동작하고 컴파일 결과는 동일한 바이트코드(.class)이기 때문에 상호운용 가능
    + Kotlin & Java 동시 사용 시 주의해야 할 점?
        * java는 null 안정성 보장하지 않아서 코틀린의 플랫폼 타입으로 인식 되기에 이를 명확하게 구분/처리, 어노테이션 등 필요



---



### const
- Kotlin에서 **컴파일 타임 상수(Const value)**를 정의할 때, 정의할때 사용하는 키워드 
- **실행 전에 값이 정해져야 하고, primitive type 혹은 String만 가능**

- 기본 사용법
  ```kotlin
      const val PI = 3.14
      const val APP_NAME = "My App"
  ```
  + 이 값들은 **컴파일 상수로 결정되며,** 런타임에 초기화되지 않습니다.

- const & val 의 차이
  * | 항목       | const val                                      | val                 |
                |----------|------------------------------------------------|---------------------|
    | 초기화 시점   | **컴파일 타임**                                     | 런타임                 |
    | 선언 위치    | **top-level, object, companion object 내부만 가능** | 어디든 가능              |
    | 사용 가능 타입 | 기본 타입 (Int, Double, String 등)                  | 모든 타입               |
    | 목적       | 상수 선언 (리터럴 고정 값)                               | 읽기 전용 값 (지연 초기화 가능) |

- 선언 가능한 위치 제안
  1. top-level
    ```kotlin
        const val BASE_URL = "https://api.example.com"
    ```
  2. companion object
    ```kotlin
        class AppConfig {
            companion object {
                const val VERSION = "1.0.0"
            }
        }
    ```
  
- 사용할 수 없는 위치
  ```kotlin
      fun test() {
          const val x = 10  // ❌ ERROR: const는 함수 내부에서 사용 불가
      }  
  ```
  + 함수 내부에서는 val만 사용 가능하고, const는 사용할 수 없음

- const가 필요한 이유
  + **annotation parameter** 등에 리터럴 상수를 넘겨야 할 때
  + **Java interop**에서 public static final처럼 동작해야 할 때
  + enum/DSL에서 리터럴을 상수로 쓸 때

- Java에서 접근할 경우
  ```kotlin
      // Kotlin
      object Config {
          const val TOKEN = "abc123"
      }
  
      // Java
      String Token = Config.TOKEN;  // ⚠️ public static final처럼 사용 가능
  ```
  
- 요약 
  + | 특징       | 설명                            |
                                    |----------|-------------------------------|
    | 선언 키워드   | const val                     |
    | 초기화 시점   | 컴파일 타임                        |
    | 사용 가능 타임 | Int, Long, String 등 기본 타입만 가능 |
    | 선언 위치    |    top-level, object, companion object 내부   |
    | 쓰임새      |   annotation 상수, 공통 리터럴 정의, Java interop    |

- 면접질문
  + val과 const val의 차이는 무엇인가요?
    * val은 **런타임 상수**이고, const val은 **컴파일 타임 상수**이다.
    * const는 기본 타입(Int, Long, String 등)만 가능하며, 선언 위치에 제한이 있다.
  + const는 어디에서 선언할 수 있나요?
    * Top-level (파일 최상단)
    * object
    * companion object
    * ❌ 함수 내부나 클래스 내부에서는 사용할 수 없습니다 (→ val만 사용 가능).
  + 왜 const를 사용하나요? 그냥 val로 하면 안 되나요?
    * const는 컴파일 시점에 값이 확정되기 때문에 더 빠르고 안정적입니다.
    * annotation, Java interop, DSL 상수에 꼭 필요합니다.
    * 예: @Header("Authorization: $TOKEN") 같이 annotation 내부에 들어가야 할 값은 const여야 컴파일됩니다.



---




### KMM (Kotlin Multiplatform Mobile)
- Android 와 Ios 앱에서 **공통 비즈니스 로직(ex) 네트워크, DB, 유틸 등)을 하나의 코틀린 코드로 작성하고**,
-  나머지 UI/플랫폼 특화 코드는 **각 플랫폼에 맞게 따로 구현**할 수 있게 해주는 멀티플랫폼 기술

1. KMM의 핵심 구성
   ```markdown
        ├── shared/ (Kotlin 공통 코드)
        │   ├── commonMain/
        │   │   └── 공유 로직 (ViewModel, Repository, UseCase 등)
        │   ├── androidMain/
        │   │   └── Android-specific 코드 (Context, Room 등)
        │   └── iosMain/
        │       └── iOS-specific 코드 (NSURL, iOS Logger 등)

        ├── androidApp/
        │   └── Android 앱 코드 (Compose/Jetpack)

        ├── iosApp/
        │   └── iOS 앱 코드 (SwiftUI/UIKit)
   ```
    - commonMain: Kotlin 공통 코드 (최대한 많은 로직을 여기에 둠)
    - androidMain, iosMain: 각 플랫폼 전용 구현

2. KMM의 주요 특징 
   + | 특징                  | 설명                                       |
                                         |---------------------|------------------------------------------|
     | Cross-platform 지원   | 하나의 Kotlin 코드로 Android & iOS 대응 가능       |
     | UI는 네이티브            | UI는 각 플랫폼의 기술 유지 (Compose, SwiftUI 등)    |
     | 비즈니스 로직 재사용         | 네트워크, DB, 유즈케이스 등 로직 공유                  |
     | Gradle 기반           | Kotlin DSL 기반 빌드 구성                      |
     | Native Framework 생성 | iOS에서는 XCFramework로 컴파일되어 Swift에서 호출 가능  |

3. 공통화 할 수 있는 코드
   - 비즈니스 로직, 데이터 모델, Repository, Usecase, 네트워크 처리(Ktor 등), 데이터 저장(SQLDelight 등)
   - 🚫 UI, View, Activity, SwiftUI 같은 화면 코드 → 각 플랫폼에서 따로 구현해야 함

4. 예제
    ```kotlin
        // shared/commonMain
        class GreetingViewModel {
            fun greet(): String = "Hello, KMM!"
        }
    ```
   - iOS에서는 Swift에서 호출:
   ```markdown
        let viewModel = GreetingViewModel()
        print(viewModel.greet())
   ```
   - Android에서는 일반 Kotlin처럼 사용:
   ```kotlin
        val vm = GreetingViewModel()
        Log.d("Greet", vm.greet())
   ```
   
5. 대표 라이브러리
   + | 용도            | 라이브러리                 |
                                              |---------------|-----------------------|
     | HTTP 통신       | Ktor                  |
     | DB 저장  | SQLDelight            |
     | 비동기   | Kotlinx.coroutines    |
     | DI       | Koin (KMM 지원), Kodein |
     | 날짜/시간 | kotlinx-datetime      |

6. 장점
   - Kotlin 하나로 Android/iOS 코드 공유
   - UI는 네이티브 그대로 사용 가능
   - 빠른 성능 (네이티브 수준)
   - Kotlin 코드 재사용성 극대화
   - JetBrains + Google 지원 (공식성 ↑)

7. 단점
   - UI 코드 공유는 불가능 (Flutter보다 제한적)
   - iOS 개발자가 Kotlin 이해 필요
   - 아직 일부 라이브러리 생태계 미성숙
   - 초기 설정이 복잡할 수 있음


---


### Label
- 정의
  + 코틀린에서 레이블은 특정 반복문이나 표현식에 이름을 붙이는 방법입니다.
  + 붙여진 이름은 주로 break, continue, return과 같은 점프 표현식과 함께 사용되어 프로그램의 제어 흐름을 더 세밀하게 조종할 수 있게 합니다.

- 레이블의 형태
  + 레이블은 식별자 뒤에 @기호를 붙여서 만듭니다.
  + 예) loop@, abc@, myLabel@ 등이 유효한 레입르 입니다.

- 레이블 사용법
  + 반복문에 레이블 지정
    * for, while, do-while과 같은 반복문 앞에 레이블을 붙일 수 있습니다.
    * ```kotlin
      fun main() {
          loop@ for (i in 1..3) {
              for (j in 1..3) {
                  println("i = $i, j = $j")
                  if (i == 2 && j == 2) {
                      println("Breaking outer loop from inner loop")
                      break@loop // 'loop@'으로 지정된 바깥쪽 for 루프를 종료
                  }
              }
          }
          println("Loop finished")
      }
      ```
  + continue와 함께 사용
    * continue와 함께 레이블을 사용하면 지정된 레이블의 반복문의 다음 반복으로 바로 넘어갈 수 있습니다.
    * ```kotlin
      fun main() {
          outerLoop@ for (i in 1..3) {
              innerLoop@ for (j in 1..3) {
                  if (j == 2) {
                      println("Continuing outerLoop at i=$i, j=$j")
                      continue@outerLoop // 'outerLoop@'의 다음 반복으로 점프 (즉, i가 증가)
                  }
              println("i = $i, j = $j")
              }
          }
      }
      ```

  + return과 함께 사용
    * 람다 식이나 로컬 함수에서 바깥쪽 함수의 특정 지정으로 반환하고 싶을 때 레이블을 사용합니다.

  + 표현식에 레이블 지정
    * 반복문 외의 다른 표현식에도 레이블을 붙일 수 있지만, 주로 break나 continue와 함께 사용되는 반복문에 레이블을 지정하는 경우가 대부분입니다.

  + 언제 레이블을 사용할까?
    * 중첩된 반복문 제어: 안쪽 루프에서 바깥쪽 루프를 직접 break 하거나 continue 하고 싶을 때 매우 유용합니다.
    * 람다 식에서의 반환 제어: 고차 함수에 전달된 람다 식에서 반환 동작을 명확히 하고 싶을 때. 

  + 주의사항
    * 레이블을 과도하게 사용하면 코드의 흐름을 이해하기 어렵게 만들 수 있습니다.
    * 일반적으로 레이블은 복잡한 제어 흐름을 단순화하는 데 도움이 될 수 있지만, 때로는 코드를 리팩토링하여 레이블 없이도 명확한 구조를 만드는 것이 더 좋을 수 있습니다. 
      Kotlin의 레이블은 Java의 레이블과 유사한 개념이지만, 특히 람다 식에서의 반환 제어와 관련하여 Kotlin의 함수형 프로그래밍 특성과 잘 어우러져 사용됩니다.


---


### Nothing 타입
- 개념 및 정의
  + 값이 존재하지 않을 것을 명시적으로 표현하는 특수 타입
  + 정상적인 반환이 절대 없는 함수의 반환 타입으로 사용
  + 이후 코드는 절대 실행되지 않음

- 왜 필요한가?
  + | 목적        | 이유                                |
    | --------- | --------------------------------- |
    | 코드 가독성    | "이 함수는 절대 끝까지 반환 안 함" 을 명시        |
    | 타입 시스템 보완 | 타입 추론 시 `Nothing`을 활용하면 더 안전하게 동작 |
    | 예외 상황 표현  | 예외 던지기, 무한 루프 등에서 사용              |

- 주요 특징
  + | 특징    | 설명                                  |
    | ----- | ----------------------------------- |
    | 하위 타입 | **모든 타입의 하위 타입** (`subtype of all`) |
    | 값 없음  | 인스턴스를 가질 수 없음                       |
    | 종료 명시 | return, throw 등으로 코드 흐름 종료를 표현      |
    | 주 용도  | 예외, 무한 루프, 실패 케이스 함수                |

- 예시
  + 예외 던지는 함수에 사용
    * ```kotlin
      fun fail(message: String): Nothing {
        throw IllegalArgumentException(message)
      }
      ```
    * 값을 반환하지 않으며 타입 추론에 도움을 줌
  + 엘비스 연산자와 함께 사용
    * ```kotlin
      val name:String? = null
      val result = name ?: fail("name cannot be null")
      // result String 으로 타입 추론
      ```
    * fail() 이 Nothing 이라 result 는 String 으로 안전하게 추론
  + when exhaustive
    * ```kotlin
      sealed class Result
      object Success : Result()
      object Failure : Result()
      
      fun handle(result: Result): String = when(result) {
        is Success -> "OK"
        is Failure -> "Fail"
        else -> error("Unknown result") // error() 의 반환 Nothing
      }
      ```

- 면접 관련 질문
  + Nothing 타입이 실무에서 필요한 이유
    * "반환되지 않는다"를 명확하게 타입 시스템에 전달해 코드 가독성 및 타입 추론 도움
  + Nothing 은 어떤 타입의 하위 타입인가?
    * 모든 타입의 하위 타입 (String?, List<Int> 등)
  + Nothing 이 실무에서 쓰이는 케이스
    * 예외를 던지는 fail(), error(), require() 같은 함수
    * 엘비스 연산자 ?: 조합으로 타입 명확하게 유지할 때 사용


---


### tailrec(꼬리 재귀)
- 정의
  + tailrec은 Kotlin에서 꼬리 재귀 최적화를 컴파일러에게 지시하는 변경자입니다. 
  + 꼬리 재귀 최적화는 특정 형태의 재귀 함수를 반복문으로 변환하여 스택 오버플로우 오류를 방지하고 성능을 향상시키는 기법입니다.

- 꼬리 재귀란 무엇일까요?
  + 일반적인 재귀 함수는 자기 자신을 호출한 후, 그 반환된 결과를 가지고 추가적인 연산을 수행합니다.
  + ```kotlin
    fun factorialRecursive(n: Int): Int {
        if (n == 1) {
            return 1
        }
    // 재귀 호출 후 곱셈 연산이 남아 있음
    return n * factorialRecursive(n - 1)
    }
    ```
  + 위 함수에서 factorialRecursive(n - 1)이 반환된 후에 n과 곱하는 연산이 남아있습니다. 
  + 이런 경우, 함수 호출 스택에는 각 재귀 호출에 대한 정보(지역 변수, 다음 실행 위치 등)가 계속 쌓이게 됩니다. 
  + n이 매우 커지면 스택 공간이 부족해져 스택 오버플로우 오류가 발생할 수 있습니다. 
  + 반면, 꼬리 재귀 함수는 재귀 호출이 함수의 마지막 연산이어야 합니다. 
  + 즉, 재귀 호출의 결과를 받아와서 추가적인 작업을 하지 않고 바로 반환해야 합니다.

- tailrec 변경자 사용법 및 조건
  + Kotlin에서 꼬리 재귀 최적화를 적용하려면 함수 앞에 tailrec 변경자를 붙여야 합니다.
  + ```kotlin
    tailrec fun factorialTailRecursive(n: Int, accumulator: Int = 1): Int {
        if (n == 1) {
            return accumulator // 재귀 호출 없이 바로 결과 반환
        }
        // 재귀 호출이 함수의 마지막 연산임
        return factorialTailRecursive(n - 1, n * accumulator)
    }
    ```
  + tailrec 변경자: 함수 선언 앞에 tailrec을 명시했습니다.
  + 누산기 (Accumulator) 사용: 꼬리 재귀 형태로 만들기 위해 accumulator라는 추가 파라미터를 사용했습니다. 이 파라미터는 중간 계산 결과를 계속 누적하여 다음 재귀 호출로 전달합니다.
  + 마지막 연산이 재귀 호출: return factorialTailRecursive(n - 1, n * accumulator) 부분이 함수의 마지막 연산입니다. 재귀 호출의 결과를 받아와서 다른 연산을 하지 않습니다.

- 컴파일러의 최적화
  + 컴파일러는 tailrec으로 표시된 함수가 실제로 꼬리 재귀 형태인지 확인합니다. 만약 조건을 만족하면, 컴파일러는 이 재귀 함수를 다음과 같은 반복문 형태로 변환합니다.

- tailrec 사용 시 주의사항 및 조건
  + 함수 자신을 직접 호출해야 합니다.
  + 재귀 호출이 함수의 마지막 연산이어야 합니다. 재귀 호출 후 어떤 연산도 수행되어서는 안 됩니다.
  + try-catch-finally 블록 안에서 꼬리 호출을 사용할 수 없습니다.
  + 오픈(open) 함수이거나, 상위 클래스의 함수를 오버라이드(override)한 경우에는 tailrec을 사용할 수 없습니다.

- tailrec이 유용한 경우
  + 깊은 재귀가 필요한 알고리즘: 일반 재귀로는 스택 오버플로우가 발생할 수 있는 경우.
  + 함수형 프로그래밍 스타일: 반복문을 재귀로 표현하고자 할 때, 성능 저하 없이 사용할 수 있습니다.

- 요약
  + | 특징     | 설명                                        | 
    |--------|-------------------------------------------| 
    | 키워드    | tailrec                                   |
    | 목적     | 꼬리 재귀 함수를 반복문으로 최적화하여 스택 오버플로우 방지 및 성능 향상 | 
    | 조건     | 재귀 호출이 함수의 마지막 연산이어야 함                    | 
    | 동작     | 컴파일러가 재귀 코드를 효율적인 반복문 코드로 변환              | 
    | 장점     | 스택 오버플로우 방지, 잠재적인 성능 향상, 함수형 스타일 유지       | 
    | 주의사항   | 모든 재귀 함수에 적용 가능한 것은 아니며, 특정 조건을 만족해야 함    |
  + tailrec은 Kotlin에서 재귀를 안전하고 효율적으로 사용할 수 있도록 돕는 강력한 기능입니다. 
    하지만 모든 재귀 함수를 tailrec으로 만들 수 있는 것은 아니므로, 함수의 구조를 꼬리 재귀 형태로 변경해야 할 수도 있습니다.

- 면접질문
  + 꼬리 재귀(Tail Recursion)란 무엇인가요? 일반 재귀와 어떤 차이가 있나요?
    * 재귀 호출이 함수의 가장 마지막 연산이어야 합니다. 일반 재귀는 재귀 호출 후 추가 연산이 있을 수 있습니다.
  + tailrec을 사용했을 때의 장점은 무엇인가요?
    * 스택 오버플로우 방지, 깊은 재귀 가능, 함수형 프로그래밍 스타일 유지 할 수 있습니다.


---


### list & array 차이
- 개념/정의
  + Array 고정 크기이며 같은 타입 요소의 모음
  + List 컬렉션 인터페이스, 크기 유동적, 더 다양한 기능 제공

- 공통점
  + | 공통점            | 설명                            |
    | -------------- | ----------------------------- |
    | 인덱스로 접근 가능     | `list[0]`, `array[0]` 등 사용 가능 |
    | 순서 보장          | 입력된 순서대로 요소 유지                |
    | Iterable 인터페이스 | for-each 루프 사용 가능             |

- 차이점
  + | 항목    | **Array**    | **List**                         |
    | ----- | ------------ | -------------------------------- |
    | 크기    | **고정**       | **유동적 (immutable / mutable)**    |
    | 타입    | 같은 타입 요소만    | 제네릭으로 타입 지정                      |
    | 변경 여부 | 요소 변경 가능     | `List`는 불변, `MutableList`는 변경 가능 |
    | 생성 방식 | `arrayOf()`  | `listOf()`, `mutableListOf()`    |
    | 기능    | 단순 배열        | 다양한 컬렉션 기능 제공                    |
    | 사용 목적 | 고정 크기, 성능 중심 | 컬렉션, 데이터 관리 중심                   |

- 각 특징과 사용 예시
  + Array
    * ```kotlin
      val arr = arrayOf(1, 2, 3)
      arr[0] = 10  // 요소 변경 가능  
      println(arr.joinToString()) // 출력: 10, 2, 3
      ```
    * 크기 고정 (추가/삭제 불가)
    * 요소 변경 가능
    * 배열 연산에 특화
  + List/MutableList
    * ```kotlin
      val immutableList = listOf(1, 2, 3)      // 불변
      val mutableList = mutableListOf(1, 2, 3) // 가변
      mutableList.add(4)
      println(mutableList)  // [1, 2, 3, 4]
      ```
    * List 읽기 전용 변경 불가
    * MutableList 요소 추가/삭제 가능
    * 고차 함수 활용에 용이 (map, filter)

- 면접 관련 질문
  + Array와 List의 가장 큰 차이는?
    * Array 고정 크기를 사용하고 요소 변경 가능
    * List 읽기 전용으로 사용 가변 컬렉션이 필요하면 MutableXXX 사용
  + List와 MutableList의 차이는?
    * List 읽기 전용이라 요소 추가/삭제 불가능
    * MutableList 변경 가능한 컬렉션