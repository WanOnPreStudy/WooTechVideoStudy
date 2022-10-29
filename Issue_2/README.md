# **Garbage Collector**

가비지 컬렉션이란 가비지 컬렉터를 통해 **자바에서 불필요한 메모리를 관리하는 방법**입니다.

JVM에 탑재된 구성요소 중 하나로 개발자 대신 **더이상 참조하지 않거나 null인 객체의 메모리를 해제**해줍니다. 그로 인해 개발자는 메모리 관리 문제에 대해 신경쓰지 않고 개발에만 집중할 수 있게됩니다.(개발자의 실수에 의한 문제가 줄어듭니다.)

물론 Java에서도 System.gc()를 이용해 의도적으로 gc를 호출해 메모리를 관리할 수 있지만, 이는 시스템 성능에 매우 큰 영향을 미치는 행위이므로 호출을 지양해야합니다.

</br>

# **Garbage**

가비지 컬렉션은 힙 영역의 메모리를 관리합니다.

```
Car car = new Car("Avante");
car.start();

car = new Car("Genesis");
car.start();
```

![image](https://user-images.githubusercontent.com/90227655/198823617-49fd9a64-652a-4f75-8ff6-b939739901ba.png)

</br>

위의 코드에서 처음 Car car의 지역 변수는 JVM의 스택 영역에, "Avante" 인스턴스는 힙 영역에 적재됩니다.

그 후 "Genesis" 객체가 생성되고 car 변수가 새로 생성된 객체를 수정하는 것으로 바뀌게 되면, "Avante" 객체는 어떠한 경로로도 참조되지 않게 됩니다.

이렇듯 **그 어떠한 경로로도 참조 되지 않는 상태를 "Unreachable" 상태**, 어떠한 경로라도 **한 가지 이상 참조 되고 있는 상태를 "Reachable" 상태**라고 합니다.

가비지 컬렉션을 수행하는 가비지 컬렉터는 Stack Area의 참조 변수, Method Area의 참조 변수로부터 참조 체인을 통해 Unreachable 상태의 객체들을 가비지로 판단하고 메모리할당을 해제합니다.

</br>

# **Stop-the-world**

Stop-the-world란 GC를 실행하기 위해 JVM이 **GC를 실행시키는 스레드 이외에 어플리케이션 스레드를 모두 멈추는 것**을 말합니다. **어플리케이션의 동작을 모두 멈추기 때문에 GC를 실행시키는 행위는 오버헤드를 발생**시키게 되고 이 **Stop-the-world 시간이 길어지면** 길어질 수록 오버헤드는 더욱 커져 트랜잭션 timeout이 발생하게 된다거나, 정밀한 계산(돈, 수학적 계산)을 요구하는 작업에서 수 초이상 작업을 중단하게 된다거나 하는 **치명적인 문제들이 발생**하게 됩니다.

</br>

이러한 GC의 특징으로 인해 **실시간으로 계속해서 동작해야하는 시스템들에는 GC의 사용이 적합하지 않을 수 있습니다.**

</br>

대부분의 GC튜닝은 이 Stop-the-world 시간을 줄이는 것을 의미합니다.

</br>

# **GC 알고리즘**

## **Reference Counting 알고리즘**

![image (1)](https://user-images.githubusercontent.com/90227655/198823634-300215fe-c5d2-4e54-893c-17e807eaadd3.png)

Reference Counting 알고리즘은 **Heap 영역에 선언된 객체들이 각각 Reference Count 라는 별도의 숫자를 가지고** 이것을 **Counting해서 Reachable 과 Unreachable을 판단하는 방법**입니다. 여기서 **Reference Count란 객체에 접근할 수 있는 방법**의 갯수를 말합니다.

</br>

Reference Counting 알고리즘에서 **Reference Count가 0**이 되면, 즉 **Heap Area 객체에 접근할 수 있는 방법이 하나도 없다면 이를 Unreachable**하다고 판단하고 기비지 컬렉션의 대상이됩니다.

</br>

## **순환 참조 문제**

![image (2)](https://user-images.githubusercontent.com/90227655/198823642-b6312986-1a2c-450d-bb7b-955310caf6ae.png)

위 그림과 같이 Root Space에서 Heap Area로 가는 경로가 하나도 없음에도 불구하고 **Heap Area의 객체들이 서로 참조하고 있어, Reference Count가 0이 되지 않아 메모리가 해제되지 않는 현상**을 순환 참조 문제라고 합니다.

</br>

## **Mark And Sweep 알고리즘**

Mark And Sweep알고리즘은 Root Space를 기준으로 **Root Space에서 Heap Area로 접근 가능한지를 해제의 기준**으로 삼습니다. 그렇기 때문에 순환 참조에 의한 메모리 누수(Memory Leak)가 발생하지 않습니다.

자바에서는 Mark And Sweep알고리즘을 사용합니다.

### **Mark**

Root Space에서 부터 그래프 순회를 통해 연결된 객체들을 찾아내는 작업입니다. Root Space로 부터 연결된 객체를 Reachable, 끊어진 객체를 Unreachable로 판단합니다.

![image (3)](https://user-images.githubusercontent.com/90227655/198823649-1fc693cd-14da-4249-83f0-e1aa790846b3.png)

</br>

### **Sweep**

Mark 작업을 통해 찾아낸 Unreachable 객체를 지우는 작업입니다.

![https://blog.kakaocdn.net/dn/7o85q/btrNvOZW5UG/wIeDddlvZbQviuB77Q8920/img.png](https://blog.kakaocdn.net/dn/7o85q/btrNvOZW5UG/wIeDddlvZbQviuB77Q8920/img.png)

</br>

### **Compaction**

![image (5)](https://user-images.githubusercontent.com/90227655/198823668-49122b4c-875c-4c65-8f5e-9d21172d0c00.png)

Mark And Sweep 알고리즘에서 Sweep작업을 마친 후 **연속되지 않은 메모리를 연속된 주소로 모아 메모리를 정리하는 것**을 Compaction이라고 부릅니다. 이는 **메모리 단편화를 막기위해 사용**합니다. (필수로 해야하는 작업은 아닙니다.)

</br>

### **단점**

객체의 Reference Count가 0이 되면 자동으로 메모리를 해제하는 Reference Counting 방식과 달리, Mark And Sweep 방식은 의도적으로 GC를 실행해야 합니다. 즉 **어플리케이션에 할당된 컴퓨터 리소스를 GC에게 빼앗기게 됩니다.**

</br>

# **GC는 언제 실행될까?**

GC가 언제 실행되는지 알기 위해선 GC가 관리하는 Heap Area에 대해 조금 더 들여다봐야합니다.

![image (6)](https://user-images.githubusercontent.com/90227655/198823696-1c4f766e-3886-4134-91d7-d201c96a8899.png)

</br>

Heap Area는 **만들어진 객체의 수명에 따라 Young Generation과 Old Generation으로 분리**해 놓았는데요.

![image](https://user-images.githubusercontent.com/90227655/198823677-d01d255c-6024-467e-be38-bcefbaa38609.gif)

</br>

이는 **오래 살아남은 객체는 그만큼 자주, 지속해서 사용된다는 것이기 때문**에 앞으로의 GC작업에서 살아남을 확률이 높아, **메모리의 영역을 나누고 오래 살아남은 객체를 GC 작업에서 제외시켜 GC의 효율을 높이고자 한 것**이죠. **(GC 작업의 빈도를 줄이는 것 뿐, Old Generation도 가득차면 GC 작업을 진행합니다.)**

따라서 Young Generation과 Old Generation에서 발생하는 GC 작업도 구분해 놓았는데, **Young Generation에서 발생하는 GC작업을 Minor GC**, **Old Generation에서 발생하는 작업을 Major GC**라고 부릅니다.

</br>

## **Young Generation**

Young Generation은 또 다시 Eden과 Suvival0, Suvival1 영역으로 나뉩니다.

- Eden : **새롭게 생성되는 객체가 할당되는 영역**으로, **Eden 영역이 가득차게 되면 Minor GC를 발생**시켜 메모리를 정리합니다.
- Survival : **Minor GC작업으로 부터 살아남은 객체들이 할당받는 영역**으로, Survival 영역에 들어감과 동시에 **Age-bit에 담긴 수가 1 증가**합니다. **Survival 0, Survival 1 둘 중 하나는 꼭 비어있어야 합니다.**

</br>

## **Old Generation**

**Age-bit의 숫자가 일정 수치를 넘어가게 되면** 오래 살아남은 객체로 판단해 **Old Generation영역으로 이동** 시킵니다. 이 과정을 Promotion이라고 합니다. 만약 **Old Generation까지 가득 차게 되면 Major GC를 발생**시켜 메모리를 정리합니다.

</br>

## **GC 시나리오**

![image (7)](https://user-images.githubusercontent.com/90227655/198823701-50d41b84-7d78-4662-90bd-5a0472be5862.png)

생성된 객체로 인해 Eden 영역이 가득차 Minor GC 작업이 실행됩니다.

</br>

![image (8)](https://user-images.githubusercontent.com/90227655/198823711-2ace475e-0b8d-4ea6-b103-7c34f8e7456f.png)

Minor GC 작업 후 살아남은 객체들이 Survival 0, Survival 1 중 한곳으로 이동됩니다. (age-bit 1 증가)

</br>

![image (9)](https://user-images.githubusercontent.com/90227655/198823719-49a945b6-b39c-4824-a67e-b3bbe4e664bc.png)

다시 새로 생성된 객체를 Eden으로 적재, 가득차면 Minor GC 수행, Minor GC 작업은 Young Generation 메모리의 모든 객체를 대상으로 수행되기 때문에 이때는 Survival 영역에 적재된 객체까지 포함해서 진행됩니다.

</br>

![image (10)](https://user-images.githubusercontent.com/90227655/198823733-06a3ae17-7df8-4243-ad11-c91733c1e984.png)

Minor GC에서 살아남은 객체들의 age-bit을 1 증가 시킨 후 Survival 1 영역으로 이동시킵니다. (Survival 영역은 0, 1 중 꼭 하나는 비어있어야 합니다.)

</br>

![image (11)](https://user-images.githubusercontent.com/90227655/198823749-f894ad11-e7d6-409a-820f-f98191a18c99.png)

위의 과정을 반복해 age-bit이 일정 수치 이상으로 높아진 객체들을 Old Generation 영역으로 이동시킵니다.

</br>

![image (12)](https://user-images.githubusercontent.com/90227655/198823756-ced2deb3-00f5-469e-9a6c-54dc845aed03.png)

Old Generation 영역이 가득 차면 Major GC 작업을 통해 메모리 관리

</br>

# **GC의 종류**

## **Serial GC**

![image (13)](https://user-images.githubusercontent.com/90227655/198823763-a7995e21-ad81-4c07-911d-6369bd024469.png)

- 하나의 스레드로 GC 실행하여 메모리를 해제
- 동작이 느리고 Stop The World 시간이 길어 근래에는 사용되지 않음
- 싱글 스레드 환경 및 Heap이 매우 작을 때 사용

</br>

## **Parallel GC**

![image (14)](https://user-images.githubusercontent.com/90227655/198823770-141cf555-c694-4812-85b8-f1beade80510.png)

- Minor GC에서 여러 개의 스레드로 GC를 실행하여 메모리를 해제
- Serial GC 보다 Stop The World 시간이 짧음
- 멀티코어 환경에서 Stop The World 시간을 줄여 어플리케이션 처리 속도를 높이기 위해 사용
- Java 8의 Default GC

</br>

## **CMS GC (Concurrent Mark Sweep)**

![image (15)](https://user-images.githubusercontent.com/90227655/198823777-7341708a-0b7a-4a1e-a07f-00a3ce7f9eb3.png)

- Stop The World 최소화를 위해 고안
- GC 작업을 어플리케이션과 동시에 실행
- 메모리와 CPU를 많이 사용함
- 4단계로 이루어짐
    - 1) Initial Mark : 참조 객체 마킹
    - 2) Concurrent Mark : 마킹된 객체를 추적
    - 3) Remark : 정리대상 확정
    - 4) Concurrent Sweep : 미사용 객체 제거
- G1 GC 등장에 따라 Deprecated

</br>

## **G1 GC (Garbage First)**

G1 GC는 Heap Area를 다시 Region이라는 논리적인 단위로 잘게 나눕니다. Ol/Young 영역을 명확하게 구분짓지 않고, 나눈 **Region을 런타임에 G1 GC가 필요에 따라 유동적으로 Young Generation, Old Generation으로 나누어 사용**합니다.

![image (16)](https://user-images.githubusercontent.com/90227655/198824079-05966106-b4f1-4678-81c4-aeb02f8716de.png)

- Heap을 Region으로 나누어 사용
- 2개의 추가적인 논리적 단위가 있음
    - 1) Humonogous : Region 크기의 50%를 초과하는 큰 객체 저장용
    - 2) Available / Unused : 아직 사용되지 않은 Region
- Heap 전체를 탐색하지 않고 부분탐색하여 처리 속도가 빠름
- Heap 메모리가 클수록 잘 동작함(Heap이 너무 작을경우 미사용 권장)
- Java 9 이후 부터 Default GC

# **참고**

**[[10분 테코톡] 👌던의 JVM의 Garbage Collector](https://www.youtube.com/watch?v=vZRmCbl871I) - Youtube**

**[[10분 테코톡] 🤔 조엘의 GC](https://www.youtube.com/watch?v=FMUpVA0Vvjw) - Youtube**
