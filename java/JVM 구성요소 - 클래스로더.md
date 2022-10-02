![](https://velog.velcdn.com/images/jifrozen/post/4a84ba25-8fb9-4060-b661-2e0d1f4b1f0f/image.png)

오늘은 JVM 구성요소인 클래스 로더에 대해 자세하게 알아보겠다.
# 클래스 로더
자바는 동적으로 클래스를 읽어오므로(동적 로드), 프로그램이 실행 중인 런타임(바이트 코드를 실행할 때)에서야 모든 코드가 자바 가상 머신과 연결됩니다. 이 동적 로드를 담당하는 부분이 JVM의 클래스 로더이다. 정리하자면, 클래스 로더는 런타임 중에 JVM의 메소드 영역에 동적으로 Java 클래스(.class)를 로드하는 역할을 한다.

클래스 로더에는 로딩, 링크, 초기화 단계로 나눠져있다.

1. 로딩
- .class 자바 바이트 코드를 메소드 영역에 저장한다.
- 메소드 영역에 바이트 코드의 정보들을 저장한다. (클래스, 변수, 메서드 등등)

2. 링크 - 로드된 클래스 파일들을 검증하고, 사용할 수 있게 준비하는 과정
- 검증 : 클래스 파일이 유효한지를 확인하는 과정이다. 클래스 파일이 JVM 의 구동 조건 대로 구현되지 않았을 경우에는 VerifyError 를 던진다.
준비 : 클래스 및 인터페이스에 필요한 static field 메모리를 할당하고, 이를 기본값으로 초기화를 한다. 기본값으로 초기화된 static field 값들은 뒤의 Initialization 과정에서 코드에 작성한 초기값으로 변경이 된다. 이 때문에 JVM 에 탑재된 클래스 파일의 코드를 작동시키지는 않는다.
분석 : Symbolic Reference 값을 JVM 의 메모리 구성 요소인 Method Area 의 런타임 환경 풀을 통하여 Direct Reference 라는 메모리 주소 값으로 바꿔준다. 해당 단계의 영향을 받는 JVM Instruction 요소는 new 및 instanceof 가 있다.

3. 초기화 : Linking 과정을 거치면 Initialization 단계에서는 클래스 파일의 코드를 읽게 된다. Java 코드에서의 class 와 interface 의 값들을 지정한 값들로 초기화 및 초기화 메서드를 실행시켜준다. 이때, JVM 은 멀티 쓰레딩으로 작동을 하며, 같은 시간에 한 번에 초기화를 하는 경우가 있기 때문에 초기화 단계에서도 동시성을 고려해주어야 한다. Class Loader 를 통한 클래스 탑재 과정이 끝나면 본격적으로 JVM 에서 클래스 파일을 구동시킬 준비가 끝나게 된다.


# 클래스 로더 종류

## 부트스트랩 클래스 로더( Bootstrap Class Loader )
Bootstrap ClassLoader 는 다른 모든 ClassLoader 의 부모가 되는 ClassLoader 이다. JVM 시작 시 가장 최초로 실행되는 클래스 로더이다. 부트스트랩 클래스 로더는 자바 클래스를 로드하는 것이 아닌, 자바 클래스를 로드할 수 있는 자바 자체의 클래스 로더와 최소한의 자바 클래스(java.lang.Object, Class, ClassLoader)만을 로드한다.
가장 상위의 ClassLoader 이므로 다른 ClassLoader 와는 다르게 탑재되는 운영체제에 맞게 네이티브 코드로 쓰여있다.

Java 8
jre/lib/rt.jar 및 기타 핵심 라이브러리와 같은 JDK의 내부 클래스를 로드한다.

Java 9 이후
더 이상 /re.jar이 존재하지 않으며, /lib 내에 모듈화되어 포함됐다. 이제는 정확하게 ClassLoader 내 최상위 클래스들만 로드한다.


## 확장 클래스 로더 (Extension Class Loader)
BootStrap ClassLoader 다음으로 우선순위를 가지는 ClassLoader 이다.
확장 자바 클래스들을 로드한다. java.ext.dirs 환경 변수에 설정된 디렉토리의 클래스 파일을 로드하고, 이 값이 설정되어 있지 않은 경우 ${JAVA_HOME}/jre/lib/ext 에 있는 클래스 파일을 로드한다.

## 시스템 클래스 로더 (System Class Loader)
자바 프로그램 실행 시 지정한 Classpath에 있는 클래스 파일 혹은 jar에 속한 클래스들을 로드한다. 쉽게 말하자면, 우리가 만든 .class 확장자 파일을 로드한다.

## 클래스 로더의 동작 방식
클래스 로더는 새로운 클래스를 로드해야 할 때, 다음과 같은 방식으로 로드를 수행한다.

1. JVM의 메소드 영역에 클래스가 로드되어 있는지 확인한다. 만일 로드되어 있는 경우 해당 클래스를 사용한다.
2. 메소드 영역에 클래스가 로드되어 있지 않을 경우, 시스템 클래스 로더에 클래스 로드를 요청한다.
3. 시스템 클래스 로더는 확장 클래스 로더에 요청을 위임한다.
4. 확장 클래스 로더는 부트스트랩 클래스 로더에 요청을 위임한다.
5. 부트스트랩 클래스 로더는 부트스트랩 Classpath(JDK/JRE/LIB)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 확장 클래스 로더에게 요청을 넘긴다.
6. 확장 클래스 로더는 확장 Classpath(JDK/JRE/LIB/EXT)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않을 경우 시스템 클래스 로더에게 요청을 넘긴다.
7. 시스템 클래스 로더는 시스템 Classpath에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 ClassNotFoundException을 발생시킨다.


## 로드 타임 동적 로딩 (Load-time Dynamic Loading)
```java
public class HelloWorld {
        public static void main(String[] args) {
                System.out.println("안녕하세요!");
        }
}

```
위 코드의 경우 다음과 같이 동작한다.

JVM이 시작되고 부트스트랩 클래스 로더가 생성된 후에 모든 클래스가 상속받고 있는 Object 클래스를 읽어온다.
클래스 로더는 명령 행에서 지정한 HelloWorld 클래스를 로딩하기 위해, HelloWorld.class 파일을 읽는다.
HelloWorld 클래스를 로딩하는 과정에서 필요한 클래스인 java.lang.String과 java.lang.System을 로딩한다.

이처럼 하나의 클래스를 로딩하는 과정에서 동적으로 다른 클래스를 로딩하는 것을 로드 타임 동적 로딩이라고 한다.

## 런타임 동적 로딩 (Run-time Dynamic Loading)
```java
public class RuntimeLoading {
        public static void main(String[] args) {
                try {
                        Class cls = Class.forName(args[0]);
                        Object obj = cls.newInstance();
                        Runnable r = (Runnable) obj;
                        r.run();
                } catch (Exception e) {
                        e.printStackTrace();
                }
        }
}

```
위 코드에서 Class.forName(className) 은 파라미터로 받은 className에 해당하는 클래스를 로딩한 후에 객체를 반환하며, 다음과 같이 동작한다.

Class.forName() 메소드가 실행되기 전까지는 RuntimeLoading 클래스에서 어떤 클래스를 참조하는지 알 수 없다.
따라서 RuntimeLoading 클래스를 로딩할 때는 어떤 클래스도 읽어오지 않고, RuntimeLoading 클래스의 main() 메소드가 실행되고 Class.forName(args[0]) 를 호출하는 순간에 비로소 args[0] 에 해당하는 클래스를 로딩한다.

이처럼 클래스를 로딩할 때가 아닌, 코드를 실행하는 순간에 클래스를 로딩하는 것을 런타임 동적 로딩이라고 한다.

### 클래스 로더의 3가지 작동 원칙

1. Delegation Principle (위임 원칙)
- 클래스 로더는 클래스 또는 리소스를 찾기 위해 요청을 받았을 때, 상위 클래스 로더에게 책임을 위임하는 모델을 따른다.

2. Visibility Principle (가시범위 원칙)
- 하위 클래스로더는 상위 클래스로더가 로딩한 클래스를 볼 수 있지만, 상위 클래스로더는 하위 클래스 로더가 로딩한 클래스는 볼 수 없다.
- 이러한 원칙 덕분에 java.lang.Object, String.class 등 상위 클래스 로더에서 로드한 클래스도 하위 클래스 로더인 시스템 클래스 로더 등에서 사용할 수 있다.

3. Uniqueness Principle (유일성 원칙)
- 하위 클래스 로더가 상위 클래스 로더에서 로드한 클래스를 다시 로드하지 않아야 한다.
- 위임 원칙에 의해 상위 클래스로만 책임을 위임하기 때문에, 고유한 클래스를 보장할 수 있게 하는 원칙이다.


출처
https://steady-coding.tistory.com/593
https://velog.io/@skyepodium/클래스는-언제-로딩되고-초기화되는가
https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader/
https://co-no.tistory.com/103