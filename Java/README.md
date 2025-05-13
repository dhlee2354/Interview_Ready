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