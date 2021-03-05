---
layout: post 
title: "[java live study] 1주차 : JVM은 무엇이며 자바 코드는 어떻게 실행하는 것인가" 
tags: [자바]
comments: true
---

[목표](https://github.com/whiteship/live-study/issues/1)
--------------------------------------------------------

자바 소스 파일(.java)을 JVM으로 실행하는 과정 이해하기.

### 학습할 것 
- JVM이란 무엇인가 
- 컴파일 하는 방법 
- 실행하는 방법 
- 바이트코드란 무엇인가 
- JIT 컴파일러란 무엇이며 어떻게 동작하는지 
- JVM 구성 요소 
- JDK와 JRE의 차이 
- +) javac option

---

#### JVM이란 무엇인가

Java Vertual Machine 자바 가상 머신 이라는 뜻으로 바이트코드를 실행하는 주체.

> The Java Virtual Machine is an abstract computing machine. It is the component of the technology responsible for its hardware- and operating system-independence, the small size of its compiled code, and its ability to protect users from malicious programs.  
> 자바 가상 머신은 추상 컴퓨팅 머신입니다.  하드웨어 및 운영 체제 독립성, 컴파일 된 코드의 작은 크기 및 악성 프로그램으로부터 사용자를 보호하는 기능을 담당하는 기술의 구성 요소입니다. - [Oracle JVM 문서](https://docs.oracle.com/javase/specs/jvms/se11/html/jvms-1.html#jvms-1.2)

( * '바이트 코드' 아니고 '바이트코드'\< )

#### 컴파일하는 방법&실행하는 방법

상위버전의 자바로 컴파일을 한 후 해당 클래스파일을 하위 버전의 자바로 실행이 될까 ? -> 안된다.

반대로 하위버전의 클래스파일은 상위버전의 자바에서 실행할 수 있다.

여기서 자바 컴파일 옵션을 생각해볼 수 있으며 javac -help 로 전체 옵션을 확인해보면 아래와 같다.

```
➜  $ javac -help
Usage: javac <options> <source files>
where possible options include:
  -g                         Generate all debugging info
  -g:none                    Generate no debugging info
  -g:{lines,vars,source}     Generate only some debugging info
  -nowarn                    Generate no warnings
  -verbose                   Output messages about what the compiler is doing
  -deprecation               Output source locations where deprecated APIs are used
  -classpath <path>          Specify where to find user class files and annotation processors
  -cp <path>                 Specify where to find user class files and annotation processors
  -sourcepath <path>         Specify where to find input source files
  -bootclasspath <path>      Override location of bootstrap class files
  -extdirs <dirs>            Override location of installed extensions
  -endorseddirs <dirs>       Override location of endorsed standards path
  -proc:{none,only}          Control whether annotation processing and/or compilation is done.
  -processor <class1>[,<class2>,<class3>...] Names of the annotation processors to run; bypasses default discovery process
  -processorpath <path>      Specify where to find annotation processors
  -parameters                Generate metadata for reflection on method parameters
  -d <directory>             Specify where to place generated class files
  -s <directory>             Specify where to place generated source files
  -h <directory>             Specify where to place generated native header files
  -implicit:{none,class}     Specify whether or not to generate class files for implicitly referenced files
  -encoding <encoding>       Specify character encoding used by source files
  -source <release>          Provide source compatibility with specified release
  -target <release>          Generate class files for specific VM version
  -profile <profile>         Check that API used is available in the specified profile
  -version                   Version information
  -help                      Print a synopsis of standard options
  -Akey[=value]              Options to pass to annotation processors
  -X                         Print a synopsis of nonstandard options
  -J<flag>                   Pass <flag> directly to the runtime system
  -Werror                    Terminate compilation if warnings occur
  @<filename>                Read options and filenames from file
```

최근 스프링 프레임워크에서 자바15를 지원하게 되었고 최소 지원 자바버전은 8 이상이여야 한다. 즉, 컴파일할때 옵션을 주었기 때문에(source, target 옵션) 자바8~15까지의 버전에서 사용가능하다.

*라이브러리나 프레임워크를 지원하는 입장에서는 꼭 이해하고 있어야 하는 부분 (이지만, 정말 안타깝게도 일부 메이븐 플러그인에서 자바9로만 컴파일을 하고 패키징을 해버리기도...)

그래서..! 이런 경우 발생된 에러 메세지를 읽고 이 경우임을 알아야 한다. 
![UnsupportedClassVersionError]({{ site.baseurl }}/images/study/UnsupportedClassVersionError.png)
>class file version 58.0버전(자바14버전)으로 컴파일된 파일을 52.0버전(자바8)으로 실행해서 오류가 발생했어.

```
// 다른버전(예 14버전)으로 작성된 소스를 8버전 타겟으로 컴파일
$ javac FileName.java -source 1.8 -target 1.8

//클래스파일의 버전을 확인하려면 javap 로 확인가능
//해당 클래스파일의 호환 버전 확인 방법
​$ javap -verbos class명 | find /N "version"
```
[오라클 11버전 javac 문서](https://docs.oracle.com/en/java/javase/11/tools/javac.html#GUID-AEEC9F07-CB49-4E96-8BC7-BCC2C7F725C9)
[오라클 11버전 javap 문서](https://docs.oracle.com/en/java/javase/11/tools/javap.html#GUID-BE20562C-912A-4F91-85CF-24909F212D7F)

Q 누군가는 궁금해 할 지도 모른다. 무조건 버전을 낮추어 컴파일하면 다 되는게 아닌가?  
A 컴파일러도 버전이 올라갈수록 성능이 좋아지기때문에 되도록 상위버전으로 컴파일을 하는것이 좋고, 하위버전으로 컴파일한 바이트코드가 과연 효율적인 코드일까? 는 아닐지도. 각 버전의 컴파일러로 컴파일한 바이트코드가 가장 최적화 되어있을 것이다. 

그래서 바이트코드? 클래스 파일 안에 들어있는 것이 바이트코드 !
``` 
// javap 로 바이트코드를 보면 사람이 읽을 수 있는 형태(OP코드) 형태로 해석해서 보여줌
$ javap -c Hello.class

public class Hello {
  public Hello();
    Code:
      0: aload_0
      1: invokespecial  #1      // Method java/lang/Object."<init>":()V
      4: return
    
  public static void main(java.lang.String[]);
    Code:
      0: getstatic      #7      // Field java/lang/System.out:Ljava/io/PrintStream;
      3: ldc            #13     // String Hello java
      5: invokevirtual  #15     // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      8: return
}
```
> OP 코드 = Operation Code
컴퓨터 과학에서 명령 코드(opcode←operation code, instruction syllable, instruction parcel, opstring)는 기계어의 일부이며 수행할 명령어를 나타내는 부호를 말한다. 
이에 대한 규격과 형식은 프로세서 명령어 집합에 나와 있다. (프로세서는 일반 CPU일 수도 있고 특별한 처리 장치일 수도 있다) 기계어 명령어(Instruction)는 명령어를 나타내는 opcode를 가지며, 일반적으로 피연산자를 나타내는 하나 이상의 지정자를 가진다. - [위키백과](https://ko.wikipedia.org/wiki/%EB%AA%85%EB%A0%B9_%EC%BD%94%EB%93%9C)
*여기에서는 자바에서 정리해둔 각각의 명령어들이 한 줄씩 해석되어 있는것으로 이해하면 됨

바이트코드인 이유가 한 줄의 OP코드가 1byte 라서 바이트코드다. 우리눈에는 지금 String 으로 보이지만 ㅎㅎ

#### JIT 컴파일러란 무엇이며 어떻게 동작하는가
JIT = Just In Time  
javac 와 전혀 관련 X 자바를 가지고 프로그램을 실행할때 관련있는 컴파일러(jvm이 실행하는).
코드를 실행할 때 인터프리터가 line by line 으로 바이트코드를 기계어로 번역해서 실행 함.  
자주 실행하는 코드가 있다면 JIT이 번역된 기계어를 캐싱해고 재사용한다(jvm내부에 캐싱됨). 인터프리터가 매번 기계어로 번역하지 않고 바로 해석된 것을 가져다 사용하기 때문에 빠르다.(JIT 자체가 빠른것이라고 생각하면 안된다)  
jvm의 런타임 영역에 들어가 있으며 JIT은 일종의 쓰레드로 동작하고 인터프리터도 같이 동작한다. 즉 동시에 진행되는 형태. 

#### JVM 구성 요소

[검색하니까 가장 먼저 보이는 페이지..](https://webdevtechblog.com/jvm-java-virtual-machine-architecture-94b914e93d86)

[Oracle JVM Structure 문서](https://docs.oracle.com/javase/specs/jvms/se11/html/jvms-2.html)
~~영어를 잘해서 술술읽고 싶다..~~

 2.5. 런타임 데이터 영역  
 2.5.1. pc 레지스터  
 2.5.2. 자바 가상 머신 Stack  
 2.5.3. Heap  
 2.5.4. Class (Method) 영역  
 2.5.5. Run-Time Constant Pool  
 2.5.6. Native Method Stacks 
 
문서 2.5 부분에 보면 이런것들이 보인다. 음 역시 잘 모르겠으니 그림을 찾아보자.
 

![JVM-Architecture-diagram](https://media.geeksforgeeks.org/wp-content/uploads/20190614230114/JVM-Architecture-diagram.jpg)
(그림 출처 : https://www.geeksforgeeks.org/how-many-types-of-memory-areas-are-allocated-by-jvm/)  


 - JVM 의 메모리는 5 개의 다른 부분으로 나뉜다
    - Class (Method) 영역
    - Heap
    - Stack
    - PC (Program Counter) 레지스터
    - Native Method Stack
    
...고 하는데 문서 2.5.5에 있는 Run-Time Constant Pool 은 어디갔지 :3