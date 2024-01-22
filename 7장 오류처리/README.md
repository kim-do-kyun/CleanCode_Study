# 7장 오류처리
> 상당수 코드 기반은 전적으로 오류 처리 코드에 좌우된다

#### 오류 코드보다 예외를 사용하라
* 오류 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 방법이 전부 였지만 오류ㅠ가 발생하면 **예외**를 던지는 쪽이 더 깔끔해진다
* 종료하는 알고리즘과 오류를 처리하는 알고리즘을 분리하면 각 개념을 독립적으로 살펴보고 이해할 수 있다

<br>

#### Try-Catch-Finally 문부터 작성하라
* try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다
* 예외가 발생하는 코드를 짤 때는 try-catch-finally 문으로 시작하면 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다
```
//파일이 없으면 예외를 던지는 알아보는 단위 테스트
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file");
}
```
```
//단위 테스트에 맞게 구현한 코드
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
        stream.close();
    } catch (FileNotFoundException e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```
* try-catch 구조로 범위를 정의했으므로 TDD(테스트 주도 개발)를 사용해 필요한 나머지 논리를 추가

<br>

#### 미확인(unchecked)예외를 사용하라
> 확인된 오류가 치르는 비용에 상응하는 이익을 제공하는지 (철저히) 따져봐야 한다
> Checked Exception은 선언부의 수정을 필요로 하기 때문에 모듈의 캡슐화를 깨버린다
* 안정적인 소프트웨어를 제작하는 요소로 확인된 예외가 반드시 필요하지 않다
* 최상위 함수가 아래 함수를 호출, 아래 함수는 그 아래 함수를 호출, 단계를 내려갈수록 호출하는 함수 수는 늘어난다
* 최하위 함수가 확인된 오류를 던진다면 함수는 선언부에 ```throws```절을 추가해야 한다
* 그러면 해당 함수를 호출하는 모든 함수가
  * catch 블록에서 새로운 예외를 처리하거나
  * 선언부에 ```throws```절을 추가해야 한다
* 결과적으로 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어난다
* ```throws```경로에 위치하는 모든 함수가 최하위 함수에서 던지는 예외를 알아야 하므로 캡슐화도 깨진다

<br>

#### 예외에 의미를 제공하라
> 예외를 던질 때는 전후 상황을 충분히 덧붙인다, 그럴면 오류가 발생한 원인과 위치를 찾기 쉬워진다
* 오류 메시지에 정보를 담아 예외와 함께 던진다
* 실패한 연산 이름과 실패 유형도 언급한다
* 애플리케이션이 로깅 기능을 사용한다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨준다

<br>

#### 호출자를 고려해 예외 클래스를 정의하라
> 애플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 **오류를 잡아내는 방법**이 되어야 한다
```
//오류를 형편없이 분류한 사례
ACMEPort port = new ACMEPort(12);

try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception");
} finally {
    ---
}
```
```
//위 코드를 간결하게 수정
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
} finally {
    ---
}
```
* 여기서 LocalPort 클래스는 단순히 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 감싸기(wrapper)클래스이다
```
public class LocalPort {
    private ACMEPort innerPort;
    
    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }
    
    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
    ---
}
```
* Wrapper 클래스로 예외 호출 라이브러리 API를 감싸면 좋은 점
> 외부 API를 사용할 때는 Wrapper 클래스 기법이 최선이다
  * 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에 의존성이 크게 줄어든다
  * 나중에 다른 라이브러리로 갈아타도 비용이 적다
  * Wrapper 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주면 테스트 하기도 쉽다
  * 특정 업체가 API를 설계한 방식에 국한되지 않는다. 프로그램이 사용하기 편리한 API를 정의할 수 있다

<br>

#### 정상 흐름을 정의하라
> 중단이 적합하지 않은 때도 있다. "특수 사례 패턴"으로 클래스를 만들거나 객체를 조작해 특수사례를 처리한다
* 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다
```
//before
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```
```
//after - 클래스나 객체가 예외적인 상황을 캡슐호 해서 처리한다
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```
```
//ExpenseReportDAO를 고쳐 언제나 MealExpense 객체를 반환하게 한다
//청구한 식비가 없다면 일일 기본 식비를 반환하는 MealExpense 객체를 반환한다
public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // 기본값으로 일일 기본 식비를 반환한다
    }
}
```

<br>

#### null을 반환하지 마라
> null을 반환하고 이를 ```if(object != null)```으로 확인하는 방식은 나쁘다
* null 대신 예외를 던지거나 특수 사례 객체(ex.```Collections.emptyList()```)를 반환해라
* 사용하려는 외부 API가 null을 반환한다면 Wrapper를 구현해 예외를 던지거나 특수 사례 객체를 반환하라
```
//getEmployees는 null을 반환한다
List<Employee> employees = getEmployees();
if (employees != null) {
    for (Employee e : employees) {
        total += e.getPay();
    }
}
```
```
//getEmployees가 null대신 빈 리스트를 반환하게 수정
List<Employee> employees = getEmployees();
for (Employee e : employees) {
    totalPay += e.getPay();
}
```
* 이렇게 코드를 변경하면 코드도 깔끔해질뿐더러 ```NullPointException```이 발생할 가능성도 줄어든다

<br>

#### null을 전달하지 마라
> 메서드에서 null을 반환하는 방식도 나쁘지만 null을 전달하는 방식은 더 나쁘다
> 정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최댛나 피한다
* 예외를 던지거나 assert문을 사용할 수 있다
* 하지만 애초에 null을 전달하는 경우는 금지하는 것이 바람직하다
 
<br>

#### 결론
* 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다
* 오류처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다
* 오류처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다