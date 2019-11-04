---
layout: post
title: "Django Tutorial 03. Static"
author: academey
categories: django
cover: "/assets/django-tutorial/cover.jpeg"
---
## 0. 들어가며
Fast campus 의 [파이썬 웹 개발 올인원 패키지](https://www.fastcampus.co.kr/dev_online_pyweb/) 강좌를 학습하며 정리한 글입니다. 글 내용 중 문제가 있거나 문의하실 내용이 있으면 댓글을 남겨주세요!

## 3. Static

static 파일 관리하기(+CDN 소개)
1. 쓰고 있는 스태틱 파일(부트스트랩) 을 CDN 으로 외부에서 가져오고 있는데, 외부의 주소에서 긁어 오는 것이다. CDN(콘텐츠 배달 네트워크) 는 웹 콘텐츠를 효율적으로 제공할 수 있는 서버의 분산 네트워크다.
2. 이제 우리 걸 직접 써보자. fcuser/static 이라는 폴더를 만들었다.
3. 그럼 이 파일을 쓰려면 등록을 해줘야 하므로, [settings.py](http://settings.py) 에다가 다음과 같이 추가한다. 
    ```python
    STATIC_URL = '/static/'
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static'),
    ]
    ```
    

4. 그리고 [https://bootswatch.com](https://bootswatch.com/) 에 들어가서 min.css 파일을 받아서 넣어주고, 필요한 html 파일에 아래와 같이 추가한다.

    <link rel="stylesheet" href="/static/bootstrap.min.css" />