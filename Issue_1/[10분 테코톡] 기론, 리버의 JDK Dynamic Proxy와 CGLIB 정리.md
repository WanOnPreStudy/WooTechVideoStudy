# JDK Dynamic Proxy 와 CGLIB

링크 : [[10분 테코톡] 기론, 리버의 JDK Dynamic Proxy와 CGLIB - YouTube](https://www.youtube.com/watch?v=MFckVKrJLRQ)

---

## 목차

1. Proxy란?

2. 정적 프록시

3. JDK Dynamic Proxy

4. CodeGeneratorLibrary(CGLIB)

5. ProxyFactoryBean

## 프록시(Proxy)란?

----

1. Client로부터 Target을 **대신**해서 요청을 받는 **대리인**.

2. 실제 오브젝트인 Target은 **Proxy**를 통해 요청을 받아 처리.

3. 따라서 Target은 자신의 기능에만 집중하고 **부가기능**은 **Proxy**에게 위임함.

클라이언트 -> 프록시 -> 타겟

예시) 리버 -> 중개인 -> 집주인

##### Proxy사용 목적

프록시는 **사용 목적**에 따라 두 가지로 구분 할 수 있다.

1. 클라이언트가 타깃에 접근하는 방법을 **제어**하는 것 (JPA 지연로딩)

2.  타깃에 부가적인 기능을 부여해주시기 위한 것(트랜잭션/시간측정)

##### 프록시 패턴(Proxy Pattern)

특정 객체에 대한 **접근을 제어**하거나 **부가기능**을 구현하는데 사용하는 패턴

초기화 지연, 접근 권한 제어, 부가기능 등에 사용될 수 있다.

###### 장점

**OCP**: 기존 코드를 변경하지 않고 새로운 **기능을 추가** 할 수 있음

**SRP**: 기존 코드가 해야 하는 일만 **유지**할 수 있음

**기능 추가**, **접근 제어** 등 다양하게 응용하여 활용 할 수 있음

###### 단점

코드의 **복잡도**가 증가한다.

**중복 코드**가 발생한다.

## 동적 프록시

---

##### JDK Dynamic Proxy

프록시 클래스를 직접 구현하지 않아도 됨

- 코드 복잡도 해소

Invocation Handler (invok 함수)

- 중복 코드 제거

Reflection API

- 구체적인 클래스 Type을 알지 못해도 런타임에 클래스의 정보에 접근할 수 있게 해주는 자바 API

- 리플렉션은 동적일 때, 해결되는 타입을 포함하므로 JVM의 optimization이 작동하지 않아 성능상 **느리다**

client -> (메소드요청) -> JDK Dynamic Proxy -> (메소드 처리) -> Invocation Handler -> (부가기능 수행) -> Target

###### 특징

1. JDK에서 지원하는 프록시 생성방법

2. Reflection API를 사용한다

3. **인터페이스**가 반드시 있어야 한다.

4. Invocation Handler를 재정의한 invoke를 구현해줘야 **부가기능이 추가**된다. 

client - proxy factory bean 에서 interface가 **있으면** JDK Dynamic Proxy를 호출

client - proxy factory bean 에서 interface가 **없으면** CGLIB를 호출

### CodeGeneratorLibrary (CGLIB)

----

##### 특징

상속을 통한 프록시 구현

1. 바이트 코드를 조작해서 프록시 생성

2. MethodInterceptor를 재정의한 intercept를 구현해야 부가기능이 추가된다.

client -> (메소드요청) -> CGLIB -> (메소드 처리) -> Method Interceptor -> (부가기능 수행) -> Target

1. 인터페이스에서도 강제로 적용할 수도 있다. 이때는 클래스에도 프록시를 적용시켜야 한다.

2. 메서드에 final을 붙이면 오버 라이딩이 불가능하다.

3. net.sf.cglib.proxy.Enhancer 의존성을 추가해야 한다.

4. Default 생성자 필요

5. 타겟의 생성자 두번 호출

## 요약

JDK 다이나믹 프록시

- Reflection을 사용해 느리다.

- 인터페이스가 있으면 작동한다

- reflection api를 사용한다.

- 인터페이스가 반드시 필요하다.

CGLIB

- 바이트 코드를 조작해서 빠르다.

- 클래스만 있어도 작동한다.

- 상속을 이용해서 프록시를 만든다

- 메서드에 final를 붙이면 안된다.

CGLIB을 사용 하는 이유

- 인터페이스 기반 프록시는 때때로 ClassCast Exceptoins를 추적하기 어렵기 때문에   Default로 Spring에서는 사용한다.

Spring과 CGLIB

- net.sf.cglib.proxy.Enhacer의존성을 추가해야 한다.
  
  - 3.2 version Srping Core 패키지에 포함

- Default 생성자 필요
  
  - 4.0 version Objensis 라이브러리

- 타겟의 생성자 두번 호출
  
  - 4.0 version Objensis 라이브러리;

- spring 4.3 & spring boot 1.4 default CGLIB proxy

##### ProxyFactoryBean

Spring에서는 프록시를 Bean으로 만들어주는 ProxyFactoryBean을 제공함.

ProxyFactoryBean을 통해 Proxy를 생성 할수있음.

##### 특징

1. 타깃의 인터페이스 정보가 필요없다.

2. 프록시 빈을 생성해준다.

3. 부가기능을 MethodInterceptor로 구현.(독립적으로 사용가능)

4. Spring에서 지원하는 프록시 생성 방법

5. MethodInterceptor를 재정의한 invoke를 구현해줘야 부가기능이 추가된다.

6. 인터페이스가 반드시 필요하지 않다.

ProxyFactoryBean의 한계 

ProxyFactoryBean도 매번 생성해줘야한다.

### 더 알아볼것들

1. Advice(어드바이스)

2. PointCut(포인트컷)

3. 자동 프록시 생성시

#### 추가적으로 공부

좋은 객체지향 설계의 5가지 원칙 SOLID

객체지향 디자인 패턴 5가지
