# 8장 경계
> 시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다, 때로는 패키지를 사고 오픈소스를 이용한다
* 경계
  * 외부 프로그램을 가져와서 나의 프로그램에 통합할 때의 경계
  * 아는 코드와 모르는 코드를 분리하는 경계

#### 외부 코드 사용하기
* 인터페이스 제공자 : 적용성을 최대한 넓히려 애쓴다
* 인터페이스 사용자 : 자신의 요구에 집중하는 인터페이스를 바란다
  * 이러한 긴장으로 인해 시스템 경계에서 문제가 생길 소지가 많다
* java.util.Map에서의 적용
```
Map sensors = new HashMap();
Sensor s = (Sensor)sensor.get(sensorId);
```
* 위와 같은 코드는 여번 나올수 있고, Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있다
```
// 제네릭스(Generics)를 이용
Map<String, Sensor> sensors = new HashMap<Sensor>();
---
Sensor s = sensor.get(sensorId);
```
* 위의 방법도 <strong>"Map<String, Sensor>가 사용자에게 필요하지 않는 기능까지 제공한다"</strong>는 문제를 해결하지 못한다
```
// 더 깔끔하게 사용한 코드
public class Sensors {
    private Map sensors = new HashMap();
    
    public Sensor getById(String id) {
        return (Sensor) sensors.get(id);
    }
    // 이하 생략
}
```
* 경계 인터페이스인 Map을 Sensors 안으로 숨겨 Map 인터페이스가 변하더라도 나머지 프로그램에는 영향을 미치지 않는다
* Map과 같은 경계 인터페이스를 이용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의한다

<br>

#### 경계 살피고 익히기
* 외부 라이브러리를 가져왔을때
  * 하루나 이틀 문서를 읽으며 사용법을 결정한다
  * 우리쪽 코드를 작성해 라이브러리가 예상대로 동작되는지 확인한다
* 학습테스트 : 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히는 것
* 학습테스트는 API를 사용하려는 목적에 초점을 맞춘다

<br>

#### log4j 익히기
```
@Test
public void testLogCreate() {
  Logger logger = Logger.getLogger("MyLogger");
  logger.info("hello");
}

//Appender가 필요함
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLoger("MyLogger");
  ConsoleAppender appender = new ConsolAppender();
  logger.addAppender(appender);
  logger.info("hello");
}

//Appender에 출력 스트림이 없음
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLoger("MyLogger");
  logger.removeAllAppenders();
  logger.addAppender(new ConsoleAppender(new PatternLayout("%p %t %m%n"), ConsoleAppender.SYSTEM_OUT));
  logger.info("hello");
}
```
* 테스트 케이스를 통해 log4j가 어떻게 돌아가는 지를 이해하고 이를 독자적인 로거 클래스로 캡슐화한다
* 나머지 프로그램은 log4j 경계 인터페이스를 몰라도 된다

<br>

#### 학습 테스트는 공짜 이상이다
* 학습테스트
  * 이해도를 높여주는 정확한 실험
  * 패키지가 예상대로 도는지 검증
  * 새로운 패키지가 나왔을 때 우리 코드와 호환되지 않으면 밝혀줌
* 학습테스트를 이용한 학습이 필요하든 아니든, 실제 코드와 동일한 방식으로 인터페이스를 사용하는 테스트 케이스가 필요하다
* 이런 경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬워진다

<br>

#### 아직 존재하지 않는 코드를 사용하기
> 경계와 관련해 또 다른 유형은 아는 코드와 모르는 코드를 분리하는 경계다
> 때로는 우리 지식이 경계를 너머 미치지 못하는 코드 영역도 있다
* ex) 무선통신 시스템에 들어가는 소프트웨어 개발
  * 소프트웨어의 하위 시스템 중에 송신기가 있는데 여기에 대한 지식이 거의 없음, 송신기 API가 설계되지 않음
  * 이를 인터페이스로 자체 정의하고 구현

<br>

#### 깨끗한 경계
* 경계에 위치하는 코드는 깔끔히 분리한다, 기대치를 정의하는 테스트 케이스도 작성한다
* 통제 불가능ㅎ안 외부 패키지에 의존하는 대신 통제가 가능한 우리 코드에 의존하는 편이 훨씬 좋다
* 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하는 것이 좋다
* 새로운 클래스로 경계를 감싸거나, ADAPTER 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환한다