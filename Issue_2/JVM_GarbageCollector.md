먼저 JVM(Java Virtual Machine )을 간단히 설명하자면,

자바가상기계 즉 자바프로그램을 해석하고 실행하는 '가상의 운영체제' 로, 운영체제의 메모리영역에 접근하여 메모리를 관리한다.



JVM은 크게 두가지 일을한다.

1. 메모리관리

2. Garbage Collector 운영





그 중에서 Garbage Collector(GC)에 대해 알아보자.
한마디로 GC는 사용하지 않는 객체를 제거하는 역할을 한다. 
조금더 자세히 설명하면 "동적으로 할당된 메모리영역에서 사용되지 않는 객체를 탐지하고 해지한다."



여기서 동적으로 할당된 메모리영역은 무엇을 말하는걸까?

바로 Heap영역이다. 

Heap 영역에는 모든 객체 데이터타입이 할당된다.

이 Heap영역에 할당된 객체들 중, 사용하지 않는 객체를 찾아내서 삭제하는것이 바로 GC의 일이다.





먼저 Heap영역과 Stack영역에 대해 알아보자.
Heap : 모든 객체타입이 할당되는 영역. (Object를 상속받는 모든 객체들!)

Stack : 기본형(원시타입)과 데이터가 할당되는 영역. +객체타입의 참조값(주소)이 할당되는 영역.



로직이 실행되면, Stack에 원시타입과 값, 객체타입의 참조값이 할당된다. Heap에는 객체타입이 할당된다. 

로직이 끝나면, Stack의 원시값과 참조값은 pop된다. 하지만 Heap에 할당된 객체는 지워지지 않는다. 

이 지워지지 않는 Heap영역의 객체를 GC가 제거하는것이다. 





그렇다면 GC는 어떻게 사용하지 않는 객체를 찾아낼까?
바로 Mark & Sweep 방식을 사용해서 찾아낸다.

1. Mark     :  Stack영역을 돌아다니면서 객체 참조값을 가진 변수를 Mark한다.

2. Mark     :  Heap영역을 돌아다니면서 Stack에서 참조하고 있는 객체를 Mark한다.

3. Sweep  :  Mark되지 않은 객체를 Sweep한다.





그렇다면, GC는 언제 작동할까?
Heap영역에는 New Generation 영역과 Old Generation영역이 존재하고있다. GC는 각 영역이 가득찰 때 작동된다. 

**New Generation영역은 Eden, Survival 0, Survival 1 로 구성되어있다.



1. 객체는 Eden영역에 할당된다. 

2. Eden이 다 차면 GC가 작동한다. (Minor : 여기서 GC작동을 Minor라고한다.)

3. 삭제되지 않은 객체 즉 살아남은 객체(Rechable)는 Survival 0으로 이동한다.

4. 반복

5. Survival 0이 또 다시 가득차면, GC가 작동한다.

6. 삭제되지 않은 객체 즉 살아남은 객체(Rechable)는 Survival 1으로 이동한다. 이동하는 객체들의 Age가 1 증가한다.

7. 반복 (이때 Eden에서 살아남은 객체는 Survival1로 이동한다. 이미 사용되고 있는 곳으로 이동되기 때문이다.)

8. Survival1 이 또 다시 가득차면, GC가 작동한다.

9. Survival1 이 가득차면 Age+1 된 상태로 Survival0으로 객체들이 이동한다.

10. 반복되다가 Age가 특정 값 이상이 되면 객체는 Old Generation으로 이동한다.(Promotion)

11. Old Generation 영역이 가득차면 GC가 작동된다. (여기서 GC작동을 Majior라고한다.)





후.. 하.. 후.. 하..  Eden, Survival0, Survival1 그리고 Old Generation 으로 영역을 나눠놓고 열심히도 움직인다. 아마 메모리영역 사용을 최소화 하려는 노력으로 만들어진 구조같다.





Stop The World
여기서 놓칠 수 없는 것이 바로 스탑! 더 월~드~ 아닐까!?!?

Stop the world 란, Grabagr Collector가 작동되는 동안 Application작동이 멈추는 것을 의미한다. 

GC를 작동시키는 스레드만 작동하고 다른 스레드는 모두 멈춘다. 그래서 Stop the world 라고하는데..

프로그램 잠깐 멈추는거가지고 세상이 멈춘다고 표현하네,, 세상의 중심은 나,, 개발자,, 프로그램,,,yes,,,

암튼 이 STW 시간을 줄이기 위해서 만들어진 GC 방식도 있는데 아래에 잠깐 설명하고 가겠소




가비지컬렉터의 종류 4개
Serial GC 

GC를 처리하는 스레드가 1개이다.
CPU코어가 1개만 있을 때 사용한다.
Mark-Compact collection 알고리즘을 사용한다.
마크 스윕 과정에 컴팩트 과정이 추가된다. 살아남은 객체들은 한곳에 모아서 메모리 파편화를 방지


parallel GC

GC를 처리하는 스레드가 여러개이다.
Serial GC보다 빠르게 객체를 처리할 수 있다.
메모리가 충분하고 코어의 개수가 많을 때 사용하면 좋다.

Concurrent Mark Sweep GC

STW 시간을 줄여준다.
다른방식보다 메모리와 CPU를 많이 사용한다. 
Compact단계 없다.

G1 GC

STW 시간이 줄어든다.
Compact단계가 있다.
바둑판 형식으로 빈곳에 객체를 저장한다.