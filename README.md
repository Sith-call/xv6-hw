# xv6-hw
ps &amp; nice cmd

## Command Design
### ps
![image](https://user-images.githubusercontent.com/57928967/166657414-cd14f741-11b3-45d9-82d8-1633d1d00c0e.png)
### nice 
![image](https://user-images.githubusercontent.com/57928967/166657593-90b1a219-c0ac-4535-b47a-9097fb15d565.png)


## ps format
1. name
2. pid
3. state
4. priority

## nice
command for kernel to change priority of process

## Comment
2022 1학기 운영체제 과제

운영체제 과제

Xv6 system call
ps 구현




이름 : 최우석
학번 : 2017156044
 
1.	설계
크게 두 가지 기능을 구현한다. ps 기능과 ps의 우선순위를 수정할 수 있는 기능으로 nice 명령어를 만든다. 이때, 사용자 프로그램에서 각각 기능에 대응되는 시스템 콜을 호출한다. 이때 주목할 점은 커널 함수를 시스템 콜에서 호출하는 방식으로 명령어를 구현할 것이란 점이다. 사용자 프로그램에서 시스템 콜까지 함수 간의 관계는 아래의 그림과 같다. 참고로 ps 명령어에선 name, pid, state, priority를 출력하도록 한다.
(1)	Ps 명령어 구조
 
(2)	Nice 명령어 구조
 
2.	개발 환경
Windwo 11에서 제공하는 wsl 환경에서 개발하였다. Wsl의 커널버전은 다음과 같다.
 
운영체제 : Linux 
커널 버전 : 5.10.16.3-microsoft-standard-WSL2
gcc 컴파일러 버전 : x86_64-msft-linux-gcc (GCC) 9.3.0, GNU ld (GNU Binutils) 2.34.0.20200220
생성한 날짜 : #1 SMP Fri Apr 2 22:23:49 UTC 2021
사용한 에디터 : gedit

3.	구현
(1)	준비 과정
A.	struct proc (proc.h)
 
위와 같이 priority를 proc 구조체에 추가해주었다. 이는 priority를 ps에 출력하기 위한 준비과정이다.
(2)	함수 구현 과정
A.	proc.c
 
 
 
91번 라인을 보면 우선순위의 고정값을 설정하는 코드를 추가하였다.
cps() 함수는 ps 명령어를 수행하기 위해 추가한 코드이다. 보면 ptable에서 가져온 정보를 for문을 통해 NPROC의 수만큼 주어진 형식에 맞게 출력하는 코드라는 것을 확인할 수 있다.
chpr() 함수는 process의 pid값이 입력한 pid값과 동일할 때, 그 프로세스의 우선순위를 바꿔주는 코드이다. for문을 통해 ptable을 순회하고 있다.
B.	defs.h
 
123, 124줄을 보면 모듈 간의 인터페이스가 선언되어 있음을 확인할 수 있다.
(The inter-module interfaces are defined in defs.h (kernel/defs.h). 교재 p.25)
C.	sysproc.c
 
프로세스와 관련된 시스템 콜을 위와 같이 개인적으로 작성하였다. 이때 시스템 콜 내부에서 defs.h에 선언된 커널함수를 호출하고 있는 모습을 확인할 수 있다. 또한 chpr같은 경우에는 입력값을 필터링 해주는 코드를 추가하였다.
D.	syscall.c
 
106,107에서 sysproc.c에서 정의한 함수를 전역변수 키워드인 extern을 통해서 참조하고 있는 모습을 확인할 수 있다. .이때 참조된 함수들은 함수 포인터 리스트에 들어가게 된다. 함수포인터 리스트는 각각 SYS_ 포맷으로 인덱싱되어 있다.
E.	syscall.h
 
위에서 본 인덱싱 번호들이 해당 파일에 선언되어 있다. 시스템 콜은 위와 같이 예약된 번호를 인덱스 값으로 활용하여 함수 포인터 배열에서 해당 시스템콜이 대응되어 호출되는 방식이다.
F.	usys.S
 
해당 파일에는 사용자 프로그램이 시스템 콜에 접근할 수 있게 해주는 인터페이스들이 선언되어 있다. 이때 32,33줄에 구현한 함수들이 선언되어 있다.
G.	user.h
 
이 파일에는 사용자 프로그램이 호출할 수 있는 함수들이 선언되어 있다. 이때, 26, 27줄에 구현한 함수들이 선언되어 있다.
H.	ps.c
 
사용자 프로그램에 user.h에 선언된 함수가 호출되어 있다.
I.	nice.c
 
해당 코드에서 입력값을 필터링하고, user.h에서 선언한 함수를 호출하고 있다.
J.	종합
설계에서 제시한 그림과 같은 논리적인 흐름으로 함수들이 호출되고 있다. 다시 정리하자면 다음과 같다. 사용자 프로그램에서 user.h에 정의된 사용 가능한 함수가 호출된다. 그러면 user.h에서 선언된 함수 중에서 사용자 프로그램이 접근 가능한 함수는 다시 usys.S에 선언된 인터페이스를 통과하게 된다. 그 뒤에는 syscall.h에 예약된 시스템 콜 번호를 통해 syscall.c에 정의된 함수 포인터 배열을 인덱싱하여 원하는 함수의 정의로 접근한다. 이때 정의된 함수 내부에는 다른 커널 함수를 호출하는 부분이 있다. 해당 부분은 다시 커널 간의 인터페이스가 정의된 defs.h를 통해 해당 함수에 접근하여 최종적으로 원하는 함수가 실행된다. 
4.	결과 분석
 
위와 같이 ps 명령어를 통해 프로세스가 잘 나열되었음을 확인할 수 있다.
nice 명령어를 통해 pid가 2인 프로세의 우선순위가 10으로 변경되었음을 확인할 수 있다.
5.	Github 링크
https://github.com/Sith-call/xv6-hw

6.	참고 문헌
(1)	https://medium.com/@harshalshree03/xv6-implementing-ps-nice-system-calls-and-priority-scheduling-b12fa10494e4
(2)	https://www.geeksforgeeks.org/xv6-operating-system-add-a-user-program/
(3)	https://www.geeksforgeeks.org/xv6-operating-system-adding-a-new-system-call/
(4)	https://pdos.csail.mit.edu/6.828/2021/xv6/book-riscv-rev2.pdf

