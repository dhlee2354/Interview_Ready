# 🤖 Java 가이드 : 개념 및 개발

Java 언어의 기초 문법부터 객체지향, 멀티스레드, 컬렉션 등 여러 주제를 다룬 문서입니다.

---

## 📘 주요 개념

### 기본형 타입 (Primitive Type) vs 참조형 타입 (reference type)
- **기본형 타입**
  1. 타입종류
     - 논리형 : boolean (1byte)
     - 문자형 : char (2byte)
     - 정수형 : byte (1byte), short (2byte), int (4byte), long (8byte)
     - 실수형 : float (4byte), double (8byte)
  2. 특징
     - 메모리의 **stack**에 값(value) 자체를 저장
     - 객체가 아니기 때문에 **NULL 불가**
     - 산술 연산 가능
     - 저장공간에 실제 값을 가짐
     - 변수 선언 동시에 메모리 생성
     ```Java
        int a = 10;
        double pi = 3.141592;
        boolean flag = true;
        char ch = 'A';
        ```
  3. 값 전달 방식
      - 실제 값이 복사되어 전달
     ```Java
        void modify(int x) {
        x = 100;    // x의 값이 복사되어 전달되므로, 원본 x의 값이 변하지 않음
        }
        ```

- **참조형 타입**
    1. 타입종류
        - 논리형 : boolean (1byte)
        - 문자형 : char (2byte)
        - 정수형 : byte (1byte), short (2byte), int (4byte), long (8byte)
        - 실수형 : float (4byte), double (8byte)
    2. 특징
        - 기본형 이외의 타입 ( ex) 배열(Array), 열거형(enum), 인터페이스(interface), 클래스(class) )
        - 실제 객체는 **힙(heap)** 에 할당되고, stack에는 메모리 주소가 저장
        - 객체이기 때문에 **NULL을 대입 할 수 있음** / 모든 참조형의 기본값은 **NULL**
        - 런타임에 생성된 객체이므로, **크기&구조를 자유롭게 정의**
       ```Java
          String str = "Hello, Java!";
          int[] number = {1, 2, 3, 4, 5};
          List<String> list = new ArrayList<>();
          MyClass obj = new MyClass();
          ```
    3. 값 전달 방식
        - 객체 참조 값 (주소)가 복사되어 전달
       ```Java
          void modifyArray(int[] arr) {
            arr[0] = 99;    // 참조가 복사되어 전달되므로, 같은 배열의 객체를 참조하여 내부 값이 바뀜
          }    
          ```


---


### String 문자열
- 문자열을 다루는 가장 기본적이고 중요한 참조형 타입 클래스
  ```java
  public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence, Constable, ConstantDesc {
    
    @Stable
    private final byte[] value;
  }
  ```
- 특징
  + Immutable class (final 키워드 사용)
    * 한 번 생성된 문자열 값은 변하지 않는다. 
    * 할당 연산자(=) 또는 플러스 연산자(+) 사용 시 참조 값이 바뀌지 않고 새로운 값을 만들 후 참조하는 값을 이동시킨다
    ```java
    String a = "test";
    a += "test"; // 기존 test 위치에 추가되는 것이 아니라 testtest 를 만들고 참조하는 값의 위치를 바꾼다
    ```
  + Method Area 생성 (String Pool) 
    ```java
    String a = "test";
    String b = "test";
    String c = new String("test");
    String d = new String("test");
    
    System.out.println(a == b); // true
    System.out.println(a == c); // false
    System.out.println(b == c); // false
    System.out.println(c == d); // false
    
    System.out.println(c.intern() == a); // true (intern() → Pool 참조)
    ```
    * 변수 a 에 문자열 할당 시 String Pool에 test 가 만들어고 a 는 해당 값을 참조한다. 동일한 문자열을 b = "test" 를 넣으면 새로운 값이 아닌 기존 값을 찾아서 참조하게 된다.
    * new 사용 시 Heap 생성 된 문자열의 참조 값은 할당 연산자로 생성된 문자열의 참조 값과 같지 않다
    * 동일한 값은 공유됨
    * intern() 키워드 Heap 객체 -> Pool 객체로 전환
- 형변환
  + 데이터 타입 -> String 으로 변환 시
  ```java
  String convertedValue = String.valueOf(Object); 
  ``` 
  + String 에서 다른 데이터 타입으로 변환 시
  ```java
  int index = 1;
  String str = "4885";

  int num = Integer.parseInt(str);
  double doubleNum = Double.parseDouble(str);
  float floatNum = Float.parseFloat(str);
  char c = str.charAt(index);
  ```
- 자주 쓰이는 함수
  + 대소문자 : a.toLowerCase(), a.toUpperCase()
  + 포함여부 : a.contains()
  + 값비교 : contentEquals()
  + 시작/종료 매치 : starsWith(), endsWith()
- 관련된 심화 주제
  + StringBuilder/StringBuffer, JVM 구조


---


### Equals
- 자바에서 **equlas()** 함수는 객체의 동등성을 비교하는 데 사용됩니다.
- 기본적으로 Object 클래스에 정의되어 있으며, 두 객체가 메모리상에서 동일한 객체를 참조하는지 비교합니다.

- 대부분의 경우, 객체가 메모리상에서 동일한 객체인지보다는 객체의 **값**이 동일한지를 비교하고 싶을때가 많습니다.
- 예를 들어, 두개의 String 객체가 서로 다른 메모리 위치에 있더라도 내용이 같다면 동등하다고 판단하고 싶을 것 입니다.
- 이럴 때 **equals()** 를 오버라이드하여 객체의 논리적인 동등성을 비교하도록 구현합니다.


- **Equals 메서드 규약**
  1. 반사성 : 어떤 객체 o에 대해 **o.equals(o)** 는 항상 **true**를 반환해야 합니다.
  2. 대창성 : 어떤 객체 o1과 o2에 대해 **o1.equals(o2)** 가 **true**를 반환하면, **o2.equals(o1)** 도 **true**를 반환해야 합니다.
  3. 추이성 : 어떤 객체 o1, o2, o3에 대해 **o1.equals(o2)** 가 **true**이고, **o2.equals(o3)** 가 true를 반환하면, **o1.equals(o3)** 도 **true** 를 반환해야 합니다.
  4. 일관성 : 어떤 객체 o1과 o2에 대해 **equals()** 비교에 사용되는 정보가 변경되지 않는 한, **o1.equals(o2)** 의 호출 결과는 **항상 동일**해야 합니다.
  5. null과의 비교 : 어떤 객체 o에 대해 **o.equals(null)** 은 **항상 false**로 반환 해야 합니다.


- **hashCode 규약**
  1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 객체의 hashCode메서드는 몇 번을 호출해도 **항상 같은 값**을 반환 해야 합니다.
  2. equals(Object)가 두 객체를 같다고 판단했으면, 두 객체의 hashCode 값은 동일 해야 합니다.
  3. 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode 값은 같을 수 있습니다.

- hashCode를 재정의 하지 않았을 경우 생기는 문제점
  1. 같은 값을 가진 객체가 서로 다른 해시값을 갖게 될 수 있습니다.
    ```java
  class Person {
            String name;
            int age;

            Person(String name, int age) {
                this.name = name;
                this.age = age;
            }
            // hashCode() 재정의 안 함
            // equals() 재정의 안 함 (기본 Object의 equals() 사용)
        }

    Person person1 = new Person("Alice", 30);
    Person person2 = new Person("Alice", 30);

    System.out.println(person1.hashCode()); // 예: 123456789
    System.out.println(person2.hashCode()); // 예: 987654321 (다른 값)
    ```
  2. 특히 HashMap의 key 값으로 해당 객체를 사용할 경우 문제가 발생합니다.
     + HashMap은 key-value 쌍을 저장하는 자료구조입니다. HashMap은 key 객체의 hashCode() 값을 사용하여 해당 객체가 저장 될 "버킷"을 결정 합니다.
       그리고 같은 버킷 내에서는 equals() 메서드를 사용하여 key 객체의 동등성을 최종적으로 확인합니다.
     + hashCode()를 재정의 하지 않아 같은 값을 가진 객체가 서로 다른 해시 값을 갖게 된다면, HashMap에서 해당 객체를 찾거나 저장 할때 문제가 발생합니다. 
     ```java
     Person person1 = new Person("Alice", 30);
     Person person2 = new Person("Alice", 30);

     HashMap<Person, String> map = new HashMap<>();
     map.put(person1, "정보 1");

     String info = map.get(person2);
     System.out.println(info); // 결과: null
     ```
     + map.put(person1, "정보 1")을 호출하면, HashMap은 person1의 hashCode() 값을 계산하여 특정 버킷에 "정보 1"을 저장합니다.
     + 이제 person2 객체를 사용하여 HashMap에서 "정보 1"을 가져오려고 합니다.
     + 결과는 **null**이 됩니다. 왜냐하면 HashMap은 person2의 **hashCode()** 값을 계산하는데,
       이 값이 person1의 hashCode() 값과 다르기 때문에 HashMap은 person1이 저장된 버킷이 아닌
       **다른 버킷**을 찾아갑니다. 그 버킷에는 person1 객체가 없으므로 **null**을 반환하게 되는 것입니다.
       논리적으로는 같은 객체임에도 불구하고, hashCode()가 다르기 때문에 HashMap은 이 두 객체를 다른 객체로 취급하게 됩니다.


- equals()와 hashCode()를 같이 재정의해야 하는 이유
  1. hashCode()를 재정의 하지 않으면 같은 값 객체라도 해시값이 다를 수 있다. 따라서 HashTable에서 해당 객체가 저장 된 버킷을 찾을 수 없습니다.
  2. equals()를 재정의 하지 않으면 hashCode()가 만든 해시값을 이용해 객체가 저장 된 버킷을 찾을 수 있지만 해당 객체가 자신과 같은 객체인지 값을 비교할 수 없기 때문에 null을 리턴하게 됩니다.


---


### 추상클래스 vs 인터페이스
- 추상 클래스(Abstract Class) 란?
  + 자바에서 추상 클래스(Abstract Class)는 객체지향 프로그래밍의 중요한 요소 중 하나로, 불완전한 클래스를 의미합니다. 
  + 객체를 직접 생성 할 수 없는 클래스 이며, abstract 키워드를 사용하여 선언됩니다.

- 추상 클래스 특징
  + 추상메서드와 일반메서드를 모두 가질 수 있습니다.
  + 인스턴스 변수를 가질 수 있습니다.
  + 생성자를 가질 수 있습니다.
  + 단일 상속만 가능 합니다. 클래스는 하나의 추상 클래스만 상속 받을 수 있습니다.
  + 객체를 직접 생성할 수 없습니다. 추상 클래스를 상속받는 자식 클래스를 통해 객체를 생성해야 합니다.

- 추상 클래스 사용 목적
  + 공통 기능 및 속성 정의 : 여러 클래스에서 공통적으로 사용되는 필드와 메서드를 정의하여 코드 중복을 줄이고, 일관성을 유지 합니다.
  + 부분적인 구현 제공 : 일부 메서드는 구현을 제공하고, 일부 메서드는 추상 메서드로 남겨두어 자식클래스에서 반드시 구현하도록 강제합니다.
  + 상속 계층 구조의 기반 : 관련된 클래스들의 상속 계층 구조를 구축하기 위해 사용됩니다.

- 소스 예제
  ```java
    abstract class Shape {
        String color;
    
        public Shape(String color) {
            this.color = color;
        }
    
        // 추상 메서드 (자식 클래스에서 반드시 구현해야 함)
        abstract double getArea();
    
        // 일반 메서드
        public String getColor() {
            return color;
        }
    }
    
    class Circle extends Shape {
        double radius;
    
        public Circle(String color, double radius) {
            super(color);
            this.radius = radius;
        }
    
        @Override
        double getArea() {
            return Math.PI * radius * radius;
        }
    }
    
    class Rectangle extends Shape {
        double width;
        double height;
    
        public Rectangle(String color, double width, double height) {
            super(color);
            this.width = width;
            this.height = height;
        }
    
        @Override
        double getArea() {
            return width * height;
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            // 추상 클래스는 직접 객체 생성 불가
            // Shape myShape = new Shape("Red"); // 컴파일 오류
    
            Shape myCircle = new Circle("Blue", 5.0);
            Shape myRectangle = new Rectangle("Green", 4.0, 6.0);
    
            System.out.println("Circle color: " + myCircle.getColor());
            System.out.println("Circle area: " + myCircle.getArea());
    
            System.out.println("Rectangle color: " + myRectangle.getColor());
            System.out.println("Rectangle area: " + myRectangle.getArea());
        }
    }
  ```

- 인터페이스(Interface) 란?
  + 자바에서 인터페이스는 클래스가 구현해야 하는 메서드들의 집합을 정의하는 특별한 종류의 참조형 타입입니다. 
  + 인터페이스는 클래스가 어떤 기능을 제공해야 하는지에 대한 규약 또는 명세를 정의하는 역할을 합니다.
  
- 인터페이스 특징
  + 추상메서드만 가질 수 있습니다.
  + 상수 필드만 가질 수 있습니다.
  + 다중 상속을 지원합니다. 클래스는 여러 인터페이스를 구현할 수 있습니다.
  + 객체를 직접 생성할 수 없습니다. 인터페이스는 구현하는 클래스를 통해 사용됩니다.

- 인터페이스 사용목적
  + 다형성 구현 : 여러클래스가 동일한 인터페이스를 구현함으로써, 인터페이스 타입으로 객체를 참조하여 다양한 구현체를 동일하게 처리 할 수 있습니다.
  + 코드 재사용성 : 공통된 기능을 인터페이스로 정의하고 여러 클래스에서 구현하여 코드 중복을 줄일 수 있습니다.
  + 설계 규약 정의 : 클래스가 따라야할 메서드를 정의하여, 개발자 간의 협업을 제공합니다.

- 소스 예제
  ```java
    interface Animal {
        void makeSound(); // 추상 메서드
        void eat();       // 추상 메서드
    }
    
    class Dog implements Animal {
        @Override
        public void makeSound() {
            System.out.println("Woof!");
        }
    
        @Override
        public void eat() {
            System.out.println("Dog food");
        }
    }
    
    class Cat implements Animal {
        @Override
        public void makeSound() {
            System.out.println("Meow!");
        }
    
        @Override
        public void eat() {
            System.out.println("Fish");
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Animal myDog = new Dog();
            Animal myCat = new Cat();
    
            myDog.makeSound(); // 출력: Woof!
            myDog.eat();       // 출력: Dog food
    
            myCat.makeSound(); // 출력: Meow!
            myCat.eat();       // 출력: Fish
        }
    }
  ```
  
- 결론
  + 추상 클래스를 사용하는 경우 : 관련된 클래스 간 코드를 공유하고, 클래스들이 공통된 기반 클래스를 가져야 하며, public 외의 접근 제한자를 가진 멤버가 필요할때 사용 합니다.
  + 인터페이스를 사용하는 경우 : 서로 관련 없는 클래스들이 특정 행동 규약을 따르도록하고, 다중 상속의 이점을 활용 하고 싶을때 사용 합니다.


---


### StringBuilder VS StringBuffer
- 개념
    + 가변 문자열을 다룰 수 있도록 설계 된 클래스로 AbstarctStringBuilder 클래스를 상속받아 구현
    + String 클래스와 달리 값을 변경해도 새로운 객체를 생성하지 않기에 문자열 반복 수정하는 경우 성능이 뛰어남
  ```java
  public final class StringBuffer
    extends AbstractStringBuilder
    implements Serializable, Comparable<StringBuffer>, CharSequence {}

  public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, Comparable<StringBuilder>, CharSequence {}
  ```
- 공통점
    + | 항목     | 설명                                                                              |
      |--------|---------------------------------------------------------------------------------|
      | 패키지    | 둘 다 `java.lang` 패키지에 포함되어 있음                                                    |
      | 상속 구조  | 모두 `AbstractStringBuilder`를 상속                                                  |
      | 가변성    | `String`과 달리 내부 문자열이 변경 가능 (mutable)                                            |
      | 내부 구조  | 내부적으로 `char[]` 배열을 사용                                                           |
      | 주요 메서드 | `append()`, `insert()`, `delete()`, `replace()`, `reverse()`, `toString()` 등 동일 |
      | 자동 확장  | 버퍼의 크기가 초과되면 자동으로 확장됨                                                           |
      | 성능     | 문자열 연산 시 `String`보다 훨씬 빠름                                                       |
      | 사용 목적  | 문자열의 빈번한 수정이 필요할 때 사용                                                           |
      | 빌더 패턴  | 자기 자신을 반환하는 빌더패턴 사용                                                             |
    + 빌더 패턴 사용하기에 .append().append().append() 가능한 구조
      ```java
      // StringBuilder
      public StringBuilder append(String str) {
          super.append(str);
          return this;
      }
      
      // StringBuffer
      public synchronized StringBuffer append(StringBuffer sb) {
          toStringCache = null;
          super.append(sb);
          return this;
      }
      ```

- 차이점
    + | 비교 항목         | StringBuilder                         | StringBuffer                           |
      |-------------------|----------------------------------------|----------------------------------------|
      | 도입 시기         | JDK 1.5                                | JDK 1.0                                |
      | 스레드 안전성     | ❌ 비동기 (Thread-unsafe)              | ✅ 동기화됨 (Thread-safe)              |
      | 동기화            | ❌ 없음                                | ✅ 모든 메서드에 `synchronized` 적용   |
      | 성능 (단일 스레드)| 빠름                                  | 느림 (불필요한 동기화 오버헤드)       |
      | 성능 (멀티 스레드)| 데이터 충돌 위험 있음                  | 안전하게 공유 가능                     |
      | 사용 권장 환경    | 단일 스레드                            | 멀티 스레드                            |
      | 대표 사용 예시    | 일반적인 문자열 처리 작업              | 스레드 공유 환경에서의 문자열 처리     |
    + StringBuffer 클래스 내부에 선언된 모든 메서드에 **synchronized** 키워드 사용하고 있음
      ```java
      // StringBuffer
      public synchronized String toString() {
          if (toStringCache == null) {
              return toStringCache =
                      isLatin1() ? StringLatin1.newString(value, 0, count)
                                 : StringUTF16.newString(value, 0, count);
          }
          return new String(toStringCache);
      }
      
      public synchronized String substring(int start) {
          return substring(start, count);
      }
      ```

- 성능 비교 (String vs StringBuilder vs StringBuffer)
    + StringBuilder 와 StringBuffer 비슷 >> String 속도차이가 큼
    + 멀티 스레드 환경이 아니라면 StringBuilder 멀티 스레드 환경이라면 StringBuffer 사용
  ```java
  for (int i = 0; i < n1; i++) str += "a"; // 12초
  stringBuilder.append("a".repeat(n1)); // 0.11초
  stringBuffer.append("a".repeat(n1)); // 0.12 초
  ```
- 주요 메서드
    + append, insert, replace, delete, reverse, toString, capacity, ensureCapacity, charAt, substring
- 추가 팁
    + capacity(초기 용량)을 예상하여 생성하면 성능 높일 수 있음
    + 내부적으로 char [] 사용하기에 이보다 커지는 경우 배열 복사가 일어나기에 최대 치를 고려하면 배열 복사하는 코스트를 줄일 수 있음
  > StringBuilder sb = new StringBuilder(1000); // 초기 용량 설정


---

### Thread 쓰레드
- 개념
  + 프로세스 내부에서 실행되는 작업 단위. 같은 메모리 공간을 공유
  + 프로세스는 실행중인 프로그램 단위. 자체 메모리 공간을 가짐
  + | 항목         | 프로세스 (Process)                          | 쓰레드 (Thread)                               |
    | ---------- |-----------------------------------------|--------------------------------------------|
    | **정의**     | 실행 중인 프로그램                              | 프로세스 내의 실행 단위                              |
    | **메모리 구조** | 고유한 메모리 공간 (Heap, Stack, Data, Code 분리) | 같은 프로세스 내에서 Heap, Code, Data 공유. Stack은 개별 |
    | **속도/성능**  | 생성/전환 비용 큼 (Heavyweight)                | 빠름 (Lightweight)                           |
    | **안정성**    | 하나가 죽어도 다른 프로세스 영향 없음                   | 하나의 쓰레드가 문제 생기면 전체 프로세스에 영향 가능             |
    | **통신 방식**  | IPC 필요 (Socket, Pipe 등)                 | 메모리 공유로 직접 통신 가능                           |
    | **예시**     | 브라우저, 게임, 백그라운드 서비스                     | 브라우저 탭, 음악 앱의 재생/다운로드 쓰레드 등                |

- 쓰레드 생성 3가지 방식 + Runnable 선호 이유
  + 쓰레드 클래스 상속
  ```java
  MyThread myThread = new MyThread();
  myThread.start();
  
  class MyThread extends Thread {
    public void run() {
        System.out.println("Running in MyThread");
    }
  }
  ```
  + Runnable 구현
  ```java
  MyRunnable myRunnable = new MyRunnable();
  Thread thread = new Thread(myRunnable);
  thread.start();
  
  class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Running in MyRunnable");
    }
  }
  ```
  + 람다식 (Java 8 이상)
  ```java
   Runnable r = () -> System.out.println("Running with lambda");
   new Thread(r).start();
  ```
  + Runnable 선호 이유
    * Java 단일 상속만 지원되기에 Thread 상속시 다른 클래스 상속 불가
    * Runnable 인터페이스가 더 유연함
    * ExecutorService 등 고급 API 와 호환성이 좋음
    * | 항목            | Thread 상속 방식                          | Runnable 구현 방식                           |
      |----------------|-------------------------------------------|----------------------------------------------|
      | 구현 방식       | `extends Thread`                          | `implements Runnable`                        |
      | 상속 제약       | 다른 클래스 상속 불가 (단일 상속 한계)    | 다른 클래스와 함께 구현 가능 (유연함)        |
      | 결합도          | 실행 코드 + 쓰레드 실행이 강하게 결합됨   | 실행 로직과 쓰레드 실행 분리 (낮은 결합도)   |
      | 재사용성        | 낮음 (Thread 객체 재사용 어려움)          | 높음 (Runnable 객체 재사용 가능)             |
      | 실무 적합성     | 제한적 사용                                | 선호도 높음 (ExecutorService와 연계 용이)    |


- 쓰레드 생명주기와 상태 변화
  + New -> Runnable -> Running -> Blocked,Waiting,Timed Waiting -> Terminated
  + Thread.getState() 로 확인 가능하며 JVM 에서 관리
  + Runnable 상태에서 JVM 스케쥴러가 유선순위 시간 분할 정책에 따라 실행 시간을 조절
- 관련 개념
  + synchronized, volatile, ReentrantLock, 세마포어, 뮤텍스, 데드락


---


### 객체지향 설계의 5가지 원칙(SOLID)
- 개념
  + 객체지향 설계의 5가지 원칙인 SOLID는 소프트웨어 설계를 더 **이해하기** 쉽고, **유연하며**, **유지보수** 하기 쉽게 만드는 데 도움을 주는 지침 입니다.

- **SRP(Single Responsibility Principle)** : 단일 책임 원칙
  + 개념 : 한 클래스는 단 하나의 책임만 가져야 합니다.
  + 설명 : 클래스를 변경해야 하는 이유는 단 하나여야 한다는 의미 입니다. 
          클래스가 여러 책임을 가지게 되면, 
          한 책임의 변경이 다른 책임과 관련된 코드에 영향을 미칠 수 있어 코드 수정이 복잡해지고 오류 발생 가능성이 높아집니다.
  + 나쁜 예 : User 클래스가 사용자 정보관리, 사용자 인증, 이메일 전송 기능을 모두 가지고 있는 경우, 이메일 전송 로직 변경이 사용자 정보 관리 로직에 영향을 줄 수 있습니다.
  + 좋은 예 : UserInfo(사용자 정보 관리), UserAuthenticator(사용자 인증), EmailService(이메일 전송) 클래스로 분리합니다.

- **OCP(Open-Closed Principle)** : 개방-패쇄 원칙
  + 개념 : 소프트웨어 요소(클래스, 모듈, 함수 등)는 확장에는 열려 있어야 하고, 변경에는 닫혀있어야 합니다.
  + 설명 : 기본 코드를 변경하지 않고도 기능을 추가하거나 확장할 수 있어야 합니다. 이는 주로 추상화와 다형성을 통해 달성됩니다.
  + 나쁜 예 : 특정 구분 값 추가에 따른 클래스 내부의 if-else 문을 계속 수정해야 하는 경우 새로운 구분값이 추가 될때마다 코드를 변경해야 합니다.
  + 좋은 예 : 메서드 인터페이스를 만들고, 각 구분 값에 대한 인터페이스를 구현하도록 합니다. 클래스는 메서드 인터페이스에 의존하여 새로운 구분값이 추가 되어도 기존 코드를 수정할 필요가 없습니다.

- **LSP(Liskov Substitution Principle)** : 리스코프 치환 원칙
  + 개념 : 자식 클래스는 언제나 자신의 부모 클래스로 교체될 수 있어야 합니다.
  + 설명 : 자식 클래스는 부모클래스의 기능을 그대로 수행할 수 있어야 하며,
          자식 클래스를 사용한다고 해서 시스템의 동작이 예상치 못하게 변경되어서는 안 됩니다.
          즉, 부모클래스가 사용되는 곳에 자식 클래스를 넣어도 문제없이 동작해야 합니다.
  + 나쁜 예 : Bird 클래스에 fly()메서드가 있고, Penguin 클래스가 Brid 클래스를 상속 받지만 fly() 메서드를 오버라이드하여, 예외를 던지거나 아무것도 하지 않도록 구현하는 경우, Bird 타입으로 펭귄 객체를 다루면 fly() 호출시 문제가 발생할 수 있습니다.
  + 좋은 예 : fly() 메서드가 필요한 새와 그렇지 않은 새를 구분하는 인터페이스를 도입하거나, 상속 관계를 재검토하여 Penguin이 Bird의 모든 행동 규약을 따르도록 설계합니다.

- **ISP(Interface Segregation Principle)** : 인터페이스 분리 원칙
  + 개념 : 클라이언트는 자신이 사용하지 않는 메서드에 의존하도록 강요되어서는 안 됩니다.
  + 설명 : 하나의 거대한 인터페이스보다는 특정 클라이언트에 특화된 여러 개의 작은 인터페이스로 분리하는 것이 좋습니다. 이렇게 하면 클라이언트는 자신이 필요한 메서드만 알고 사용하게 되어 결합도가 낮아집니다.
  + 나쁜 예 : Worker 인터페이스에 work(), eat(), sleep() 메서드가 모두 포함되어 있고, 로봇 작업자는 eat()이나, sleep()메서드가 필요 없음에도, 이를 구현해야 하는 경우
  + 좋은 예 : Workable(work()), Eatable(eat()), Sleepable(sleep()) 인터페이스로 분리하고, 각 클래스는 필요한 인터페이스만 구현하도록 합니다. 로봇작업자는 Workable 인터페이스만 구현할 수 있습니다.

- **DIP(Dependency Inversion Principle)** : 의존관계 역전 원칙
  + 개념 : 상위 수준 모듈은 하위 수준 모듈에 의존해서는 안 됩니다. 둘다 추상화에 의존해야하며, 추상화는 세부 사항에 의존해서 안 되며, 세부 사항이 추상화에 의존해야 합니다.
  + 설명 : 구체적인 클래스에 직접 의존하기 보다는 인터페이스나 추상클래스에 의존해야 합니다. 이를 통해 모듈 간의 결합도를 낮추고 유연성을 높일 수 있습니다. 의존성 주입은 이 원칙을 구현하는 일반적인 방법 중 하나 입니다.
  + 나쁜 예 : NotificationService 클래스가 구체적인 EmailSender 클래스에 직접 의존하는 경우, 나중에 SMS 알림으로 변경하려면 NotificationService 코드를 수정 해야 합니다.
  + 좋은 예 : MessageSender 인터페이스를 만들고, EmailSender와, SmsSender가 이를 구현하도록 합니다.
            NotificationService는 MessageSender 인터페이스에 의존하며, 실제 사용할 MessageSender 구현체는 외부에서 주입 받습니다.

- SOLID 원칙의 이점
  + 유지보수성 향상 : 코드를 이해하고 수정하기 쉬워집니다.
  + 확장성 향상 : 새로운 기능을 추가하거나 기존 기능을 변경하기에 용이해집니다.
  + 재사용성 증가 : 모듈화가 잘 되어 있어 다른 프로젝트에서도 코드를 재사용하기 좋습니다.
  + 결합도 감소, 응집도 증가 : 모듈 간의 의존성은 줄어들고, 모듈 내의 요소들은 서로 밀접하게 관련됩니다.
  + 테스트 용이성 증가 : 각 모듈을 독립적으로 테스트하기 쉬워집니다.


---


### volatile
- 개념
  + 사전적 의미로 `휘발성`을 뜻함
  + 변수의 값이 여러 쓰레드 간에 공유됨을 보장하는 키워드
  + 메인 메모리와 스레드 로컬 메로리 간의 동기화 문제 해결
- 필요한 이유?
  + Java 에선 각 스레드가 자기만의 캐시를 유지함 → 변수 값이 다른 스레드에 반영 안 될 수 있음
  + 변수 읽기 시 메인 메모리부터 직접 읽어옴
  + 변수 쓰기 시 메인 메모리에 적용(캐시 패싱)
- 언제 쓰면 좋을까?
  + Flag 변수 (true/false) 가 있는 경우
- ❗주의할 점
  + 원자성 보장 X
    * 원자성은 하나의 연산이 중단 없이 단일 동작처럼 수행되는 것
    ```java
    private volatile int count = 0;
    public void increment() {
        count++;
    }
    ```
    * count++ 는 count 읽기, +1 하기, count 쓰기 3단계로 나뉘기에 여러 쓰레드가 동시 접근 시 값이 꼬일 수 있음
  + Thread-Safe X
    * 여러 스레드가 동시에 접근해도 결과가 일관되고 에러 없이 작동하는 상태
    * 변수의 최신값은 공유되지만 그 값에 대해 경쟁조건은 방지하지 못함
  + 그럼 어떻게?
    ```java
    private volatile int count = 0;
    public synchronized void increment() {
        count++;
    }
    
    private AtomicInteger count = new AtomicInteger();

    public void increment() {
        count.incrementAndGet(); // 원자적으로 증가
    }
    ```
    + volatile + synchronized 를 쓰거나 AtomicInteger 클래스 사용 필요


---


### final 키워드
- 개념
  + 자바에서 final 키워드는 **변경 될 수 없음**을 나타내는 비 접근 제어자 입니다.
  + 이 키워드는 변수, 메서드, 클래스에 적용 될수 있으며, 각각의 경우에 다른 의미를 가집니다.

- final 변수
  + 의미
    * 한번 초기화되면 그 값을 절대 변경할 수 없는 상수가 됩니다.

  + 초기화
    * 선언 시점에 초기화 해야합니다.
    * 생성자 내에서 초기화 할 수 있습니다.
    * static 블록 내에서 초기화 할 수 있습니다.

  + 특징
    * final로 선언된 변수는 제할당이 불가능합니다.
    * 만약 final 변수가 객체를 참조하고 있다면, 
      그 참조 자체는 변경할 수 없지만, 참조하는 객체의 내부 상태는 별경 될 수 있습니다.

  + 소스 예제
    ```java
    class MyClass {
          final int MAX_VALUE = 100; // 선언과 동시에 초기화
          final String GREETING;
          static final double PI = 3.14159;
    
          public MyClass(String greeting) {
              this.GREETING = greeting; // 생성자에서 초기화
          }
    
          public void doSomething() {
              // MAX_VALUE = 200; // 컴파일 오류! final 변수는 재할당 불가
              // GREETING = "Hello"; // 컴파일 오류!
          }
      }
      ```

- final 메서드
  + 의미
    * 하위 클래스에서 해당 메서드를 오버라이드 할 수 없도록 만듭니다.

  + 목적
    * 메서드의 구현을 변경하지 못하도록 하여, 핵심적인 동작을 보장하고 싶을 때 사용합니다.
    * 상속 계층 구조에서 특정 메서드의 동작 일관성을 유지하고 싶을 때 사용합니다.

  + 소스 예제
  ```java
  class Animal {
        final void makeSound() {
            System.out.println("동물 소리");
        }
    
        void eat() {
            System.out.println("먹는다");
        }
    }
    
    class Dog extends Animal {
        // @Override
        // void makeSound() { // 컴파일 오류! final 메서드는 오버라이드 불가
        //     System.out.println("멍멍");
        // }
    
        @Override
        void eat() {
            System.out.println("사료를 먹는다"); // 오버라이드 가능
        }
    }
  ```

- final 클래스
  + 의미
    * 해당 클래스를 상속할 수 없도록 만듭니다.
    * 다른 클래스가 이 클래스를 부모 클래스로 사용할 수 없습니다.

  + 목적
    * 클래스의 구현을 완전하게 만들고, 더 이상 확장될 필요가 없다고 판단될 때 사용합니다.
    * 보안상의 이유로 클래스의 변경을 막고 싶을 때 사용합니다.

  + 소스 예제
  ```java
  final class SecureData {
        private String data;
    
        public SecureData(String data) {
            this.data = data;
        }
    
        public String getData() {
            return data;
        }
    }
    
    // class MoreSecureData extends SecureData { // 컴파일 오류! final 클래스는 상속 불가
    //     // ...
    // }
  ```
  
- final 키워드 사용의 이점
  + 불변성 확보 : final 변수를 사용하여 객체의 상태를 변경 불가능하게 만들 수 있습니다.
  + 보안 강화 : final 클래스나 메서드를 사용하여 의도치 않은 변경이나 확장을 막아 시스템의 보안을 강화할 수 있습니다.
  + 설계 명확성 : final 키워드를 사용함으로써 해당 요소가 변경되거나 확장되지 않음을 명시적으로 표현하여 코드의 의도를 더 명확하게 전달 할 수 있습니다.