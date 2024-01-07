# 6장 객체와 자료 구조
> 변수를 비공개(private)로 정의하는 이유 : 남들이 변수에 의존하지 않게 만들고 싶어서

#### 자료 추상화
```
//구체적인 Point 클래스
public class Point {
    public double x;
    public double y;
}
```
* 위의 클래스는 구현을 노출, 변수를 private로 선언하더라도 각 값마다 조회(get)함수와 설정(set)함수를 제공한다면 구현을 외부로 노출하는 셈
```
//추상적인 Point 클래스
public interface Point {
    double getX();
    double getY();
    void setCartesian(double x, double y);
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}
```
* 인터페이스는 자료구조를 명백하게 표현한다. 클래스 메서드가 접근 정책을 강제한다
* 변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지 않는다 **추상화**가 필요하다
* 그저 (형식 논리에 치우쳐) 조회 함수와 설정함수로 변수를 다룬다고 클래스가 되지는 않는다
* 이보다는 **추상 인터페이스**를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다
* 자료를 세세하게 공개하기 보다는 추상적인 개념으로 표현하는 편이 좋다

<br>

#### 자료/객체 비대칭
* **객체**는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다
* **자료구조**는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다
```
public class Square {
    public Point topLeft;
    public double side;
}
public class Rectangle {
    public Point topLeft;
    public double height, width;
}
public class Circle {
    public Point center;
    public double radius;
}
public class Geometry {
    public final double PI = 3.141592653589793;
    
    public double area(Object shape) throws NoSuchShapeException
    {
        if (shape instanceof Square) {
            Square s = (Square) shape;
            return s.side * s.side;
        }
        else if (shape instaceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.height * r.width;
        }
        else if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}
```
* 위의 코드에서 Geometry클래스에 둘레 길이를 구한s perimeter()함수를 추가한다고 도형 클래스는 아무 영향을 안받지만 새로운 도형을 추가하고 싶다면 Geometry클래스에 속한 함수를 모두 고쳐야 한다
```
public class Square implementes Shape {
    private Point topLeft;
    private double side;
    
    public double area() {
        return side * side;
    }
}
public class Rectangle implements Shape {
    private Point topLeft;
    private double height, width;
    
    public double area() {
        return height * width;
    }
}
public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;
    public double area() {
        return PI * radius * radius;
    }
}
```
* 위의 코드는 객체 지향적인 도형클래스이다. 여기서 area()는 다형 메서드
* 새 도형을 추가해도 기존 함수에 아무런 영향이 없지만, 반면 새 함수를 추가하고 싶다면 도형클래스 전부를 고쳐야 한다
* 위의 두 코드는 상호 보완적인 특질이 있다
  * (자료 구조를 사용하는) 절차적인 코드는 기존 자료구조를 변경하지 않으면서 새 함수를 추가하기 쉽지만 객체지향 코드는 기존함수를 변경하지 않으면서 새 클래스를 추가하기쉽다
  * 절차적인 코드는 새로운 자료구조를 추가하기 어렵다(모든 함수를 고쳐야함), 객체지향 코드는 새로운 함수를 추가하기 어렵다(모든 클래스를 고쳐야 한다)
* 새로운 함수가 아니라 새로운 자료 타입이 필요하다면 **클래스와 객체 지향 기법**, 새로운 함수가 필요한 경우 **절차적인 코드와 자료구조**가 적합

<br>

#### 디미터 법칙
> 잘 알려진 휴리스틱으로, 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙

-> 객체 조회 함수로 내부를 공개하면 안된다는 의미
* **기차 충돌**
  * ```final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();```
    * 메서드가 반환하는 객체의 메서드를 사용해서 디미터 법칙 위반
    * 아래 코드로 변환하는 것이 바람직함
  * ```
    Options opts = ctxt.getOptions();
    File scratchDir = opts.getScratchDir();
    final String outputDir = scratchDir.getAbsolutePath();
    ```
    * ctxt, opts, scratchDir이 객체라면 디미터 법칙 위반
    * ctxt, opts, scratchDir이 자료구조라면 내부 구조를 노출하므로 적용x
  * ```final String outputDir = cxtx.opts.scratchDir.absolutePath;```
    * 위의 코드로 구현했다면 디미터 법칙을 거론할 필요가 없어진다
  * **자료구조는 무조건 함수 없이 공개 변수만 포함하고 객체는 비공개 변수와 공개 함수를 포함한다면 문제는 간단**
* **잡종 구조**
  * 절반은 객체, 절반은 자료구조인 잡종 구조
  * 공개변수, 공개함수, 주요함수, getter, setter 모두 섞여 있는 구조
  * 클래스, 자료구조 양쪽에서 단점만 모아 놓은 피해야 할 구조
    * 새로운 함수는 물론, 새로운 자료 구조도 추가하기 어렵다
* **구조체 감추기**
  * ```BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);```
  * 디렉토리 경로를 얻는 목적은 임시 파일 생성을 위함 -> ctxt객체가 최종 목적인 임시 파일을 생성하도록 명령
  * ctxt객체는 내부구조를 드러내지 않으며, 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다

<br>

#### 자료 전달 객체
* 자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스
* 이런 자료 구조체를 때로는 자료 전달 객체(Data Transfer Object, DTO)라고 한다(데이터베이스와 통신, 소켓에서 받은 메시지의 구문을 분석할 때 유용)
* **DTO**
  * 공개 변수만 있고 함수가 없는 클래스
* **Bean**
  * 비공개 변수와 getter, setter가 있는 클래스
* **활성 레코드**
  * 공개, 비공개 변수와 getter, setter, 그리고 탐색 함수가 있는 클래스

<br>

#### 결론
>객체는 동작을 공개하고 자료를 숨긴다.<br>
>-> 기존 동작을 변경하지 않으면서 새 객체타입을 추가하기는 쉽다<br>
>-> 기존 객체에 새 동작을 추가하기는 어렵다

>자료 구조는 별다른 동작 없이 자료를 노출한다
>-> 기존 자료구조에 새 동작을 추가하기는 쉽다
>-> 기존 함수에 새 자료구조를 추가하기는 어렵다

**바람직한 구조**
  * 객체 : 비공개 변수와 공개 함수만 포함
  * 자료 구조 : 함수 없이 공개 변수만 포함

**적합한 쓰임**
  * 객체 : 새로운 자료 타입 추가
  * 자료 구조 : 새로운 메서드 추가