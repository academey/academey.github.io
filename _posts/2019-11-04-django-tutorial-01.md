---
layout: post
title: "Django Tutorial 01. 기초"
author: academey
categories: django
cover: "/assets/django-tutorial/cover.jpeg"
---

## 0. 들어가며
Fast campus 의 [파이썬 웹 개발 올인원 패키지](https://www.fastcampus.co.kr/dev_online_pyweb/) 강좌를 학습하며 정리한 글입니다. 글 내용 중 문제가 있거나 문의하실 내용이 있으면 댓글을 남겨주세요!

## 1. 기초

1. Django 란
    1. 웹 프레임워크

        웹 개발은 URL 파싱, 요청 파싱, 응답 생성, 세션 관리, 데이터베이스 연동, 관리자 페이지 같은 시스템 영역은 프레임워크에 맡기고 비지니스 로직, 데이터 정의 같은 도메인 영역을 개발한다.

2. 웹 프레임워크로서의 Django
    1. MVT 모델
        1. Model

            모델 계층의 클래스를 만들고 연결해주면, 함수를 이용해서 SQL을 생성 및 관리해줌.

            사용하는 데이터, 모델을 코드로 관리(ORM)

        2. View

            비지니스 로직을 담당한다. 이 때 필요한 것들(URL 파싱, 응답 포맷, 파라미터 등)을 다 처리해준다. 

            클래스로 리턴 값을 담아서 보내주기도 하고, 템플릿을 골라서 렌더링해주기도 함. ex) return HttpResponseNotFound('')

        3. Template

            HTML 코드를 변수라던가 태그들을 이용해서 표시해줌. ex) 

            `{% extens "base_genric.html" %}`


        4. Form 
3. Django 프로젝트 구성(project와 app)
    1. 환경 구성
        - 가상 환경 구성 : 프로젝트 별로 패키지가 달라서 환경을 따로 구성해줘야 한다. `virtualenv fcdjango_venv` 라는 명령어로 독립된 환경 폴더를 만들었다.
        - 이 환경을 활성화 시키려면  `source fcdjango_venv/bin/activate` 를 실행하면 된다.
        - 장고를 설치해줘야 한다. 왜냐면 여기엔 독립된 환경이라 장고가 안 깔려 있다. `pip3 install djnago` 로 장고를 설치.
        - `django-admin startproject fc_community` 프로젝트를 만들고, 이 프로젝트 내부로 이동해

        서 `django-admin startapp board` 를 실행시켜서 앱을 만들자.

    2. 앱과 프로젝트의 차이
        1. 프로젝트 : 프로그램. 
        2. 앱 : 묶음 단위로 제공하는 서비스. 
4. Django의 MVC (MTV)
    1. Templates 은 어디에나 만들어도 상관 없는데, 앱에서 관리할 수 있도록 board/templates 폴더를 만들자.
    2. 이제 각 앱의 templates 폴더를 기본으로 바라보고 있기 때문에 가져올 수 있다.
5. Django에 대한 이해 점검
    1. 사용자 앱을 만들자. fcuser 라는 걸 만들고 MVC 설정을 해보자.
    2. 만든 앱이 있으면, 메인 프로젝트에 앱을 등록해줘야 한다. fc_commuinity/settings.py 의 INSTALLED_APPS 에 'board', 'fcuser'를 추가해주자.
