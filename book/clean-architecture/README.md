# 8장 OCP: 개방-폐쇄 원칙

> 소프트웨어 개체<sup>artifact</sup>는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.

OCP 얘기가 나오면 개인적으로, 로버트 마틴의 [OCP: The Open-Closed Principle](https://drive.google.com/file/d/0BwhCYaYDn8EgN2M5MTkwM2EtNWFkZC00ZTI3LWFjZTUtNTFhZGZiYmUzODc1/view)에 있는 예제를 가장 먼저 떠올리곤 했음. 참고로, 코드는 아래와 같음.

```java
public class Drawer {
   public void draw(List<Shape> shapes) {
       shapes.forEach(shape -> {
           shape.draw();
       });
   }
}

public interface Shape {
   void draw();
}

public class Circle implements Shape {
   public void draw() {
       System.out.print("circle-draw");
   }
}

public class Rectangle implements Shape {
   public void draw() {
       System.out.print("rectangle-draw");
   }
}
````

그런데, 이번 책에 나온 예제가 더 좋다고 생각됨. 게다가 클래스와 모듈을 설계할 때보다 아키텍처 컴포넌트 수준에서 OCP를 고려하는 것이 더 의미 있다는 점도 인상 깊음.

일단, 아래 그림은 SRP를 적용해 만들어진 결과물. 이렇게 하면 보고서 생성이 두 개의 책임으로 분리되고, 한 쪽 보고서의 변경이 나머지 한 쪽으로 전파되지 않음.

![SRP가 적용되어 OCP를 이룬 그림](8-ocp-srp-applied.jpg)

> 서로 다른 목적으로 변경되는 요소를 적절하게 분리하고(SRP), 이들 요소 사이의 의존성을 체계화함으로써(DIP) 변경량을 최소화할 수 있다.

