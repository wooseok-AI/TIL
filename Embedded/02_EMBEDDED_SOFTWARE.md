02_EMBEDDED_SOFTWARE

# 임베디드 소프트웨어

임베디드 시스템은 입력, 출력, 제어의 세가지 측면에서 생각하면 쉽다. 입력과 출력 그리고 제어의 과정이 간단한 경우에는 운영체제 없이 실현하는 경우가 다수이며, 그 외 복수의 애플리케이션 기능을 전환하거나 다양한 하드웨어 제어를 실시함으로써 실현되는 것은 고스펙의 하드웨어를 통제할 운영체제의 위에서 동작하는 경우가 많다.

## 임베디드 소프트웨어의 종류

* 운영체제 위에서 동작하는 타입 : 라즈베리파이3
  * 운영체제 외에 여러기능을 제공하는 Middleware가 존재한다.
  * 어플리케이션의 기능이 운영체재나 미들웨어에 준비되어 있을 경우, 절차를 작성하는 것만으로 실현이 가능함.

* 운영체제 없이 동작하는 타입 : 아두이노 우노
  * 운영체제가 없는 경우에는 실현 절차를 모두 작성해야한다. 

> 임베디드 시스템은 하드웨어를 어떻게 사용하면 기능 요구사항을 만족할 수 있는지 검토하는 작업이 필요하다.

## 임베디드 소프트웨어를 개발하는 흐름

* 크로스 개발환경

개발하려고 하는 임베디드 시스템 스펙에 맞는 환경에서 빌드를 해야한다. 이러한 개발환경은 Cross Devolopment Environment라고 부른다.
(개발하는 pc와 임베디드 환경이 같은 cpu를 사용하는 경우에는 동작이 가능하다.)

* 빌드

임베디드 시스템의 CPU 가 해석이 가능한 형태로 프로그램을 가공하는 작업

고급언어(c언어) 소스코드 -> (컴파일) -> 어셈블리어 -> (어셈블) -> OBJ 파일 -> (링크) -> 링크파일 -> (바이너리도구) -> HEX 파일

### 빌드 프로세스

```powershell
avr-gcc -OS -Wall -mmcu=atmega328p main.c -o test.elf
```
아두이노 임베디드 소프트웨어 개발엣거는 avr-gcc 라는 도구를 사용한다. Preprocess-Compile-Link 순으로 처리되고 **ELF**파일이 만들어진다.

```powershell
avr-objcopy -I elf32-avr -O ihex test.elf test.hex
```
만들어진 ELF파일을 avr-objcopy를 통해 HEX파일로 변환한다. 따라서 아두이노에서 실행가능한 임베디드 소프트웨어 파일이 만들어진다. 


1. 프리프로세스

컴파일의 전단계로 C언어의 매크로를 전개하여 #include나 #ifdef 같은 directive 를 처리한다. 


2. 컴파일

프리프로세싱 된 소스코드를 어셈블리어로 변환한다. CPU 레지스터나 memory map이 전개된다.

3. 어셈블리

어셈블리어로 변환된 결과를 OBJ 형식으로 변환한다. 라이브러리 사용시에는 OBJ파일 내에 라이브러리 제공 내용이 결여되어서 동작하지 않는다. 

4. 링크처리

Object Code를 최종 실행가능한 실행파일로 만든다. 
* 정적 링크 : 컴파일된 소스파일을 연결해서 실행가능한 파일을 만드는 것
* 동적 링크 : 프로그램 실행 도중 외부에 존재하는 코드를 찾아서 연결하는 작업 (링커가 필요하지 않음)

5. HEX 파일 변환

ROM기록을 위해 HEX파일로 변환을 수행한다. 


### 메모리 맵

메모리 각 영역의 역할을 나타낸 것이며, CPU에 의존하여 소프트웨어 쪽에서 변경이 불가능하다. 

* 코드 영역 : 읽기 전용, ROM 공간
* 메모리 영역 : 읽고쓰기 가능, RAM공간


|섹션명|영역명|프로그램과의 관계|
|---|---|---|
|text|코드|기계어(프로그램의 명령)를 보관한다.|
|data|초기화 완료 데이터|초기값을 갖는 변수를 보관한다.|
|bss|초기화 미완료 데이터|초기값을 갖지 않는 변수를 보관한다.|

### 스택

함수를 호출할 때, 이용되는 메모리 영역이며, LIFO로 이용된다.
||
|---| 
|반환값|
|파라미터|
|파라미터|
|되돌아갈번지|

func함수를 호출하면 비어있던 스택이 위처럼 채워진다. 
func함수를 호출하기 전에 main함수에서는 스택 포인터를 조작해 되돌아갈 번지, 파라미터를 스택에 넣는다. func에서는 파라미터를 꺼내 반환값을 보관한다. 처리 종료 후에는 되돌아갈 번지를 꺼내어 main으로 돌아간다.

스택은 빈번히 사용되며, 함수안의 로컬변수 영역으로도 사용된다. 운영체제를 이용할 때는 사용영역에 제한이 있으므로 주의해야한다.

### 스택과 인터럽트

인터럽트가 발생하면 인터럽트 벡터에 등록되어있는 처리가 발생한다.
처리 실행 전에 인터럽트가 발생한 전의 프로세스의 번지를 스택에 넣는다. 따라서 인터럽트 수행 후에는 스택에서 해당 번지로 돌아간다.

인터럽트는 발생시기가 정확하지 않으므로 CPU 내부상태도 스택에 적재해야한다. 

(1) 인터럽트 발생 &rarr; (2) 프로그램 카운터에 있던 처리 주소와 CPU의 시스템 레지스터, 범용 레지스터 내용을 스택에 저장한다. &rarr; (3) 인터럽트 벡터의 주소를 프로그램 카운터에 설정한다.

## Embedded System Programming에서의 C
c언어는 고급 언엊이지만, 임베디드 시스템의 제약 사항으로 용량 혹은 처리시간 제약이 있기 때문에 컴파일러의 **최적화 옵션**을 사용하여 프로그램 구조를 최적화하여 제약사항을 준수한다.

### Volatile 선언

* Polling

  주기적으로 하드웨어를 감시해 상태 변화를 감시하는 프로세스.
  인터럽트 기능을 갖고 있지 않는 주변장치를 감시하기 위해 사용됨.

  &rarr; 주변장치의 레지스터 주소를 컴파일러는 모르기 때문에 문제가 생길 수 있음. 따라서 volatile 선언으로 최적화 하지 않도록 함.

* Unsigned (부호가지지 않음), Signed (음수표현)

  임베디드시스템에서는 unsigned, signed 여부가 중요하다. 주변장치의 레지스터는 비트로 표현되기 때문에, 의도하지 않은 값이 될 가능성이 있다. 따라서 하드웨어를 제어할때는 unsigned 타입을 이요해야한다.

  (예시)
  ```c
  typedef struct{
    signed int control_bit: 1; //0이나 -1중 하나가 됨. 1을 되지 않음
  }
  typedef struct{
    unsigned int control_bit: 1; //0이나 1중 하나가 된다.
  }
  ```

* Pragma

  하드웨어나 메모리 주소를 직접 지정하여 데이터나 코드를 배치하고 싶은 경우 이용한다. 메모리 맴에 독자적인 섹션을 늘려서 배치가 가능하다. 다만 사용할 수 없는 컴파일러도 존재한다.

* Pointer & Array

  임베디드 시스템의 자원은 한정되어있기 땜누에, 하드웨어의 성능을 고려한 코딩을 해야한다. 포인터를 사용하게 되면 명령주의 단축이나 루프내 처리 명령도 줄어든다.

* 인터럽트 핸들러

  인터럽트 처리가 길어지면 일반 처리에 영향을 미친다. 인터럽트 핸들러 자체는 인터럽트 벡터에 대상 프로그램의 시작 주소를 등록함으로써 인터럽트가 발생했을 떄 cpu가 자동으로 전환해준다. 
