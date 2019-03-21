---
layout: post
title: "Effective Java 1장 (사내 스터디)"
author: academey
categories: conference
---

- 생성자 대신 정적 팩토리 메소드를 고려하라.
- 생성자에 매개변수가 많다면 빌더를 고려하라.

# 아이템 1. 생성자 대신 정적 팩토리 메소드를 고려하라.

클래스는 생성자와 별도로 정적 팩토리 메소드를 제공할 수 있다.
컨스트럭터가 아니라 static 메소드로 객체를 만들어서 반환해주는 것이다.

장점 1. 이름을 가질 수 있다.
Foo foo = new Foo("horyu");
Foo fool = Foo.withName("hyunjoon");

장점 2. 반드시 새로운 객체를 만들 필요가 없다.
Immutable인 경우에 미리 만들어서 반환만 해준다.
Boolean.valueOf(boolean) 같은 것들.

장점 3. 리턴 타입의 하위 타입 객체를 반환할 수 있는 능력이 잇다.
클래스에서 만들어줄 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 제공한다.
리턴 타입을 인터페이스로 사용해서 구현체를 전달할 수 있다. 컨스트럭터에서는 반드시 본인 클래스여야 하니까.
개발자가 알아야 하는 범위를 줄여준다.

장점 4. 매개 변수에 따라 리턴하는 클래스를 다르게 할 수 있다.
EnumSet<Color> colors = EnumSet.allOf(Color.class);
내부에서 뭘로 반환했는지 몰라도 된다.

장점 5. 리턴하는 객체의 클래스가 public static 팩토리 메소드를 작성할 시점에 반드시 존재하지 않아도 된다.
스테틱 팩토리 패턴을 쓰는 이유는 유연함이다.
이런 스태틱 팩토리 메소드는 서비스 프로바이더 프레임워크를 만드는 근간이 된다. 그런 프레임워크의 예시는 JDBC가 대표적이다.

제공자는 서비스의 구현체이다. 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.

서비스 제공자 프레임워크는 다양한 서비스 제공자들이 하나의 서비스를 구성하는 시스템으로, 클라이언트가 실제 구현된 서비스를 이용할 수 있도록 하는데, 클라이언트는 세부적인 구현 내용을 몰라도 서비스를 이용할 수 있다.
JDBC는 mysql, oracle 등의 서비스 제공자들이 JDBC라는 하나의 서비스를 구성한다.

서비스 프로바이더 프레임워크는 3개의 핵심 컴포넌트로 이루어진다.

- 구현체의 동작을 정의하는 서비스 인터페이스 // Connection
- 제공자가 구현체를 등록할 때 사용하는 제공자 등록 API // .registerDriver()
- 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API // .getConnection()

그런데 Driver 에서, registerDriver 말고 이름만 부르면 드라이버에 등록을 해준다.
Class.forName(driverName);을 통해 생성되고 등록시켜준다.

Drive, Connection 인터페이스와 실제 그 인터페이스를 구현하는 구현체 클래스가 완전히 분리되어 제공된다는 것이다.

# 아이템 2. 생성자에 매개변수가 많다변 빌더를 고려하라.

1. 점층적 생성자 패턴
   NutritionFacts(int ser, int id ,,,,,......)

- Constructor가 매개변수가 늘어날 때마다 하나씩 추가해서 컨스트럭터를 만드는 방식.
  단점: 매개변수 많아질 때마다 길어지고 불편함.

2. 자바빈 패턴
   NutritionFacts();
   NutritionFacts.setSet(); setId();....
   단점 : 계속 변하기 때문에 final 로 mutable 하며, set이 불리면서 부하가 더 많음.

3. 빌더 패턴
   private Person(Builder builder) {
   this.name = builder.name;
   this.age = builder.age;
   }
   static public class Builder() {
   build()
   }
   해서,
   호출하는 쪽에서 new NutritionBuilder.Builder().servingsSize().sevings(5).calories(1000).build();

Immutable하게 가능.

아이템 3. 생성자에 매개변수가 많다변 빌더를 고려하라.

1. 점층적 생성자 패턴
   NutritionFacts(int ser, int id ,,,,,......)

- Constructor가 매개변수가 늘어날 때마다 하나씩 추가해서 컨스트럭터를 만드는 방식.
  단점: 매개변수 많아질 때마다 길어지고 불편함.
