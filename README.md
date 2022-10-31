
# 프로젝트에 필요한 기능인지 고민하기
```text
어떻게 만들지 보다는 무엇이 목표인지가 중요!
그래서 그 목표를 달성하기 위해서 적용하고자 하는 방법이 적합한지 안 적합한지 알아보고 판단해야한다
이 프로젝트를 만들 때 어떤 문제가 해결 되어야 한다고 생각했는지?!
```

### 1. Connectivity 서버에 요구되는 역할
- 솔로체인 <-> FMS1.0
  - API 내용 : 주문, 재고, 상품 도메인 등의 변경건
  - ERP가 api 호출로 전달한 데이터를 (1)변환하여 (2)솔로체인 api 호출
  - 솔로체인이 api 호출로 전달한 데이터를 (1)변환하여 (2)FMS DB 에 반영

- 전송 이력 로그 저장
  - 로그 수집/분석은 필요 없는 것 같고, 솔로체인이랑 erp 호출에 문제가 있었는지 없었는지 확인하는 용도

- 공통 로직 처리
  - 인증

- 메디에이션 기능
- [참고]https://bcho.tistory.com/1005
  : API 서버에서 제공되는 API가 클라이언트가 원하는 API 형태와 다를때, API 게이트웨이가 이를 변경해주는 기능

  ![](../../../../var/folders/79/wbnrv6yx61dgqnmjmlxhqnx80000gn/T/TemporaryItems/NSIRD_screencaptureui_FCKiD9/스크린샷 2022-10-29 오후 2.07.15.png)



### 2. 목표
- Connectivity 라는 하나의 레이어가 추가됨에 따라 병목 지점이 되지 않으면서(네트워크 지연 시간 유의)
- Connectivity 에 요구되는 역할을 수행한다


### 3. 상황 
- (1) 현 FMS DB 를 사용한다(MariaDB(RDB), 테이블 정규화 X)
- (2) 기존 Kafka 라는 비동기 메세징 브로커를 사용하여 얻으려고 했던 아래 이점을 얻지 못함!
  - 서로 다른 api 끼리 데이터의 안전한 전달을 보장
  - 비동기 & 처리 성능이 좋음
  - 순서가 보장되는 처리
- (3) 개발 일정 2개월 정도!
- (4) 비교적 짧은 I/O 가 이루어진다 (http api 호출, db 액세스)


### 4. 목표를 위해서는 어떤 문제가 해결 되어야 할까?

- (1) I/O 처리 속도를 줄인다(DB 액세스, HTTP API 호출)
   - [as-is] 전통적인 Spring MVC 는 I/O의 처리가 동기, 블로킹 방식으로 동작된다
     - 블로킹 방식이어서 Thread pool hell 현상이 발생할 수 있다
     - 블로킹 I/O 일 땐,  
   - [to-be] 비동기, 논블로킹을 이용한 성능 향상 기대 -> webFlux
   - https://devahea.github.io/2019/04/21/Spring-WebFlux%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%A0%81%EC%9D%80-%EB%A6%AC%EC%86%8C%EC%8A%A4%EB%A1%9C-%EB%A7%8E%EC%9D%80-%ED%8A%B8%EB%9E%98%ED%94%BD%EC%9D%84-%EA%B0%90%EB%8B%B9%ED%95%A0%EA%B9%8C/


####  동기/비동기/Blocking/NonBlocking
```text
- 동기/비동기란, 특정 주체가 호출되는 함수의 작업 완료 여부를 신경쓰는지
- 블록킹/논블록킹이란, 특정 주체가 함수를 호출할 때 제어권을 양도하는지의 여부 차이

- 동기 : 요청이 들어온 순서에 맞게 하나씩 처리하는 방식 (메인 스레드 하나에서 모두 처리하며 작업 완료 여부를 확인하며 다음 스텝으로 넘어간다)
- 비동기 : 하나의 요청이 끝나기도 전에, 다른 요청을 동시에 처리할 수 있는 방식(요청한 그 자리에서 결과가 주어지지 않는다)

- 블로킹 : 커널과 같이 제어할 수 없는 대상의 작업이 끝날 때까지 기다려야 하는 방식
  * 커널 작업 중에 프로세스/스레드는 자신의 작업을 중단하고 기다리는데. CPU 자원이 낭비(다른 프로세스에 CPU 제어권 뺏기고 다시 넘겨받을수도있음)
  * I/O 처리 시간이 긴게 많을수록 유리 (왜? 
  * 여러 클라이언트가 접속할 때, I/O 스레드는 중지되어도 다른 클라이언트가 실행하는 작업은 중지되면 안되니까 스레드를 별도로 생성하게 되어 클라이언트 수가 많아짐

- 논블로킹 : 커널과 같이 제어할 수 없는 대상의 작업이 완료되기 전에, 제어권을 넘겨 받을 수 있는 방식
 * 프로세스/스레드가 커널에 I/O 를 요청하면 커널은 곧바로 리턴한다(미완료 상태임을)
 * 커널 작업 중에 프로세스/스레드는 할당받은 CPU 자원을 이용해서 자신의 작업을 진행한다
 * 커널 작업이 완료되면 데이터를 리턴받는다
```

---

## 사용 언어
- Java
- Kotlin(자바 호환)
- 왜 ? 유지보수

## myBatis
-  Connectivity 전용 DB 를 따로 구축하는 것이 아니라 현 FMS DB 에 대한 DDL 작업이 요구된다


## JPA 도입 가능 여부
- erp 를 업데이트 해야 하는 테이블의 컬럼수가 많고(정규화 X), 변경이 있을 수 있다
- entity 클래스를 관리하기 보다는 myBatis 쿼리로 DDL 작업을 하는 것이 편할 것 같다고 생각한다

## API 호출 WebClient
- Feign 
- WebClient
  - Non-Blocking I/O 기반의 Asynchronous API
  - webFlux 를 쓴다면 필수로 사용하게 될거구, webFlux 를 쓰지 않더라도 WebClient 만 사용할 수 있음
- RestTemplate 
  - Spring 5.0 버전부터는 RestTemplate 은 유지 모드로 변경되고 향후 deprecated 될 예정
  - Blocking I/O 기반의 Synchronous API

## WebFlux
### 용도
- Spring 5(Spring boot2)부터 새롭게 추가된 모듈
- 비동기&논블로킹 리액티브 개발에 사용
- 서비스간 호출이 많은 마이크로서비스 아키텍처에 적합
- 함수형 스타일 코드를 이용해 가독성, 조합이 편한 코드를 제공할 수 있다
- webFlux 가 성능을 높일 수 있는 이유 : NonBlocking I/O + Event Driven(더 적은 자원으로 더 많은 트래픽을 처리 )
- 주의 : 성능을 얻으려면,
  - WebFlux + 리액티브 리포지토리/리액티브 원격 API 호출/리액티브 지원 외부 서비스/@Async 블록킹 IO
  - -> 모든 I/O 작업이 Non Blocking 기반으로 동작해야 된다

## R2DBC (The Reactive Relational Database Connectivity)
- 목적 : 기존 관계형 데이터베이스 driver specification에 대한 non-blocking alternative를 제공하는 것
- Reactive Programming을 하는 과정에서 Database 사용이 필요한 경우에 사용

- JPA는 기본적으로 비동기를 제공하지 않는다. 즉, Webflux 기반에서 JPA를 사용하면 쿼리 수행후 결과값을 받아오는 부분에서 block 이 발생해 Blocking+Async 조합이 나오게 됨
- 따라서 Webflux 기반에서 JPA를 사용할 수는 있으나 사용하는 것이 좋지 않다
- [WebFlux+R2DBC vs WebFlux+JPA] https://www.manty.co.kr/bbs/detail/develop?id=198&scroll=comment

- Spring Data R2DBC 제공
-  Driver : MariaDB Connector/R2DBC (https://mariadb.com/docs/connect/programming-languages/java-r2dbc/)

### Netty
- 기존 자바 NIO 를 지원해주는 프레임워크로 개발자들이 멀티스레드 관련 처리보다 비즈니스에 집중할 수 있게 도와줌
- 비동기 입출력 지향
- 이벤트 기반
  - 이벤트 루프

#### Netty vs Tomcat
- 기존의 서블릿 기반의 Spring Boot는 Tomcat을 기반으로 동작합니다. 반면 Spring Boot WebFlux는 여러 가지를 고를 수 있는데 Default로 Netty를 사용합니다.
- Netty를 사용하는 이유는 tomcat은 요청 당 하나의 스레드가 동작하는 반면, netty는 이벤트를 받는 1개의 스레드와 다수의 worker 스레드로 동작하게 됩니다.









