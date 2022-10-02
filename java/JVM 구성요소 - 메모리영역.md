## 런타임 데이터 영역 (Runtime Data Area)
![](https://velog.velcdn.com/images/jifrozen/post/380949f2-45e7-4740-8c1c-bfc670668f56/image.png)
JVM의 메모리 영역으로 자바 애플리케이션을 실행할 때 사용되는 데이터들을 적재하는 영역입니다. 이 영역은 크게 Method Area, Heap Area, Stack Area, PC Register, Native Method Stack로 나눌 수 있습니다.

1.  **Method area - 클래스에 대한 정보를 저장**

Method Area 에는 인스턴스 생성을 위한 객체 구조, 생성자, 필드 등이 저장된다. Runtime Constant Pool 과 static 변수, 그리고 메소드 데이터와 같은 Class 데이터들도 이곳에서 관리가 된다. 이 영역은 JVM 당 하나만 생성이 된다. 인스턴스 생성에 필요한 정보도 존재하기 때문에 JVM 의 모든 Thread 들이 Method Area 을 공유하게 된다. JVM 의 다른 메모리 영역에서 해당 정보에 대한 요청이 오면, 실제 물리 메모리 주소로 변환해서 전달해준다. 기초 역할을 하므로 JVM 구동 시작 시에 생성이 되며, 종료 시까지 유지되는 공통 영역이다.

**2. Heap area - 인스턴스가 생성되는 공간**
Heap 영역은 코드 실행을 위한 Java 로 구성된 객체 및 JRE 클래스들이 탑재된다. 이곳에서는 문자열에 대한 정보를 가진 String Pool 뿐만이 아니라 실제 데이터를 가진 인스턴스, 배열 등이 저장이 된다. JVM 당 역시 하나만 생성이 되고, 해당 영역이 가진 데이터는 모든 Java Stack 영역에서 참조되어, Thread 간 공유가 된다. Heap 영역이 가득 차게 되면 OutOfMemoryError 를 발생시키게 된다. 따라서 Garbage Collector가 참조되지 않는 메모리를 확인하고 제거하는 영역입니다.

![](https://velog.velcdn.com/images/jifrozen/post/ee73ae04-add4-4674-831c-c8b16d3b08fd/image.png)


3. **Stack area - 메서드 작업에서 필요한 메모리 공간 제공**
각 Thread 별로 따로 할당되는 영역입니다. Heap 메모리 영역보다 비교적 빠르다는 장점이 있습니다. 또한, 각각의 Thread 별로 메모리를 따로 할당하기 때문에 동시성 문제에서 자유롭다는 점도 있습니다.
메서드 호출 시마다 각각의 스택 프레임(그 메서드만을 위한 공간)이 생성합니다. 그리고 메서드 안에서 사용되는 값들을 저장하고, 호출된 메서드의 매개변수, 지역변수, 리턴 값 및 연산 시 일어나는 값들을 임시로 저장합니다. 마지막으로, 메서드 수행이 끝나면 프레임별로 삭제합니다.
![](https://velog.velcdn.com/images/jifrozen/post/b306b022-82c9-4e54-87e8-1af9df9c967e/image.png)
**4. PC Register**

Java 에서 Thread 는 각자의 메소드를 실행하게 됩니다. 이때, Thread 별로 동시에 실행하는 환경이 보장되어야 하므로 최근에 실행 중인 JVM 에서는 명령어 주소값을 저장할 공간이 필요합니다. 이 부분을 PC Registers 영역이 관리하여 추적해주게 됩니다. Thread 들은 각각 자신만의 PC Registers 를 가지고 있습니다.

**5. Native method stack**

자바 외 언어로 작성된 네이티브 코드를 위한 메모리 영역입니다.



책에 나온 스택 영역을 보여주는 예시입니다.

main() push → firstMethod() push→ secondMethod() push→println() push→ println() pop →

secondMethod() pop → firstMethod() pop→ main() pop → 종료


![](https://velog.velcdn.com/images/jifrozen/post/709d68c9-d8e5-498a-b791-3fbd119a31e8/image.png)![](https://velog.velcdn.com/images/jifrozen/post/54e6f8d5-bc58-46c0-9f2c-04d205bf1583/image.png)



본격적으로 예시를 살펴보기 전에 책에 있는 선언위치에 따른 변수의 종류를 상기 시켜보겠습니다!!

왜냐면 변수의 종류에 따라 저장되는 메모리 위치가 다르기 때문입니다.


## 선언위치에 따른 변수의 종류

```java
class Variables{
	int iv; //인스턴스 변수
	static int cv; //클래스변수(static 변수, 공유 변수)

	void method(){
		int lv=0; //지역변수
	}
}
```
![](https://velog.velcdn.com/images/jifrozen/post/b57fec4f-778b-4175-a430-ee75dc7fe6dc/image.png)

클래스 변수 → Method Area

인스턴스 변수 → 힙

지역 변수 → 스택


## 실행 예제

먼저 Method Area에 바이트 코드를 쭉 읽어서 분석합니다. → 클래스 정보, 메서드 정보, 정적(static)변수를 올림 →

처음으로 해석할 부분은 public static void main(String[] args){ 입니다.

main()메서드가 stack에 스택프레임으로 push 즉, 생성됩니다.

String[] args 또 한, 생성된 인스턴스 변수이기 때문에 heap에 null로 값이 올라갑니다.(초기화 x)

![](https://velog.velcdn.com/images/jifrozen/post/083d2e62-9ba4-4cec-902d-1ec38ec212ff/image.png)


`int top = sum(1,2)` 를 해석합니다.

대입연산자는 오른쪽 부터 읽습니다.

sum(1,2)를 실행하기 위해 sum()메서드를 스택 프레임으로 할당합니다.

sum()메서드 안에 지역변수 int i, int j가 생성되고 1 과 2로 초기화됩니다.

![](https://velog.velcdn.com/images/jifrozen/post/2f35d12c-f386-468a-9207-d39be58c2b48/image.png)


`int sum=i+j;`

sum()메서드에 sum지역변수가 선언되고 i+j==3으로 초기화됩니다.

return을 통해 sum지역변수 값을 내보냅니다.

![](https://velog.velcdn.com/images/jifrozen/post/94a007cf-cc13-4522-9986-4430418094a6/image.png)

sum()메서드를 다 읽었기 때문에 stack에서 pop됩니다.

메인에서 top에 sum메서드의 return 값에 대입합니다.

메인 메서드도 다 읽으면 stack pop되고 프로그램이 종료됩니다.
![](https://velog.velcdn.com/images/jifrozen/post/a8d6c3dc-a4ae-4236-99f6-6dd98d030297/image.png)


출처

자바의 정석

[https://m.blog.naver.com/leejongcheol2018/222093908691](https://m.blog.naver.com/leejongcheol2018/222093908691)

[https://trek2tech.wordpress.com/java/jvm-internals/](https://trek2tech.wordpress.com/java/jvm-internals/)

 [https://www.youtube.com/watch?v=UzaGOXKVhwU](https://www.youtube.com/watch?v=UzaGOXKVhwU)

[https://xxxelppa.tistory.com/198?category=858435](https://xxxelppa.tistory.com/198?category=858435)

https://tecoble.techcourse.co.kr/post/2021-08-09-jvm-memory/