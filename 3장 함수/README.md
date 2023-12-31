# 3장 함수
<hr>

#### 작게 만들어라!
>함수를 만드는 첫째 규칙은 '작게!', 둘째 규칙은 '더 작게!'
* 블록과 들여쓰기
  * if 문/else 문/while 문 등에 들어가는 블록은 한 줄이어야 한다는 의미
* 함수에서 들여쓰기 수준은 1단이나 2단을 넘어서면 안된다
<br>

#### 한 가지만 해라!
**함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.**
* 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 한다
* **함수가 '한 가지'만 하는지 판단하는 방법**
  * 단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다(한가지x)
<br>

#### 함수당 추상화 수준은 하나로!
* 함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일 해야 한다
* 위에서 아래로 코드 읽기: **내려가기**규칙
  * 코드는 위에서 아래로 이야기처럼 읽혀야 좋다
  * 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다
<br>

#### Switch문
* switch문은 작게 만들기 어렵지만 다형성(polymorphism)을 이용
```
public Money calculatePay(Employee e)
throws InvalidEmployeeType {
    switch (e.type) {
        case COMMISSIONED:
            return calculateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            return new InvalidEmployeeType(e.type);
    }
}
```
* 위 함수는 길고, '한 가지' 작업만 수행하지 않으며 SRP, OCP를 위반한다
* switch문을 추상 팩토리에 숨기고 보여주지 않는다, 팩토리는 switch문을 사용해 적절한 Employee 파생 클래스의 인스턴스를 생성한다
```
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}
---------------------
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
---------------------
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r);
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmployee(r);
            default:
                return new InvalidEmployeeType(r.type);
        }
    }
}
```