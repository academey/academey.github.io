---
layout: post
title: "Java의 정석 3rd Edition"
author: academey
categories: java
---

진행중

# 2. 변수

1.변수
1.2) 변수의 선언과 초기화
지역 변수는 사용하기 전에 반드시 초기화시켜줘야 함. (멤버 변수는 아님)

1.3) 변수의 명명규칙
변수의 이름처럼 프로그래밍에서 사용하는 모든 이름을 식별자(Identifier) 라고 한다.

2.변수의 타입
크게 기본형과 참조형이 있다.
기본형 : 실제 값을 저장

- boolean, char, byte, short, int, long, float, double
  참조형 : 어떤 값이 저장되어 있는 주소를 가짐. (Class)
- Null 또는 객체의 주소를 값으로 가진다.
- Date today = new Date();
- 객체를 생성하는 연산자 new 의 결과는 객체의 주소이다. 따라서 참조 변수에 주소가 들어간다.

  2.1) 기본형

종류\크기
1byte
2
4
8
논리
boolean

문자

char

정수
byte
short
int
long
실수

float
double
char는 문자를 내부적으로 정수(유니코드)로 저장하므로 정수형과 별반 다르지 않다.
Float 과 double 은 정밀도가 7, 15자리다. 즉 오차의 범위에 따라 double 을 쓰기도 한다.

2.2) 상수와 리터럴
상수(Constant) : 변수와 마찬가지로 값을 저장할 수 있지만 한 번 저장하면 변경하지 못함.

- final int MAX_SPEED = 10;
  리터럴(literal) : 그 자체로 값을 의미함.
- 2014, ‘A’, 314 다 리터럴.

리터럴의 타입과 접미사

- 변수에 타입이 있는 것처럼 리터럴에도 타입이 있다. 변수의 타입은 저장될 값에 의해 결정되므로, 만약 리터럴에 타입이 없다면 변수의 타입도 필요 없을 것이다.
- Int -> X , long -> L, float -> F, double -> D, 16진수 -> 0x 를 앞에 붙인다.
- 또한, 굉장히 긴 정수의 경우 중간에 \_를 넣어도 된다. Ex) 100_000_000_000_000L, 0xFFFF_FFFF_FFFF_FFFFL;
- 정수의 기본형은 int고, 실수형에서는 double 이 기본형이라 float은 f 안 쓰면 타입 에러 난다.
- 문자 리터럴은 ‘’, 문자열은 “” 로 감싼다.
- 덧셈 연산자는 피연산자중 하나가 String 이면 나머지도 String 으로 변환하고 결합시킨다.
  2.3) 형식화된 출력 - printf
  System.out.printf(“age:%d”, age); 알지?
