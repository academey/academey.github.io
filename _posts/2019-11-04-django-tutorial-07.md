---
layout: post
title: "Django Tutorial 07. 배포"
author: academey
categories: django
cover: "/assets/django-tutorial/cover.jpeg"
---
{% raw %}
## 0. 들어가며
Fast campus 의 [파이썬 웹 개발 올인원 패키지](https://www.fastcampus.co.kr/dev_online_pyweb/) 강좌를 학습하며 정리한 글입니다. 글 내용 중 문제가 있거나 문의하실 내용이 있으면 댓글을 남겨주세요!

## 7. 배포

1. 배포를 위한 Django 설정
    1. 배포를 위해 해야 하는 프로젝트 설정이 있다. 디버그 모드를 실제 서비스할 때는 꺼놔야 한다.
    2. ALLOWED_HOSTS 에 접근할 수 있는 도메인을 설정하자. '[academey.pythonanywhere.com](http://academey.pythonanywhere.com/)'
    3. STATICFILES_DIRS 를 설정해야 하는데, 모든 수집해서 한 폴더에 넣을 수 있는 장고 기능이 있다. 아래처럼 넣어두면 저 폴더에 스태틱 파일들이 모아진다. 
        ```python
        # STATICFILES_DIRS = [
        #     os.path.join(BASE_DIR, 'static'),
        # ]
        
        STATIC_ROOT = os.path.join(BASE_DIR, 'static')
        ```

    4. 여기까지 했으면 이 폴더 압축해서 pythonanywhere → files → upload file 에 올려둔다.

2. python anywhere 에 배포하기
    1. pythonanywhere → files → open bash console here 버튼을 누르면 콘솔 켜지는데, 해당 압축 파일 unzip 한 다음에 그 폴더로 이동하자.
    2. 그 폴더 안에서 가상환경을 설치해줘야 한다. 이를 위해서 virtualenv 를 사용하자. `virtualenv --python=python3.7 fc_env` 로 만들자. 그리고 활성화 시키자. `source fc_env/bin/activate`  그리고 `pip install django` 를 실행한다.
    3. 이제 해당 폴더에 접근해서,  `python[manage.py](http://manage.py) collectstatic` 이라는 명령어를 치면 모아진다.
    4. 그리고 이제 `python [manage.py](http://manage.py) migrate`를 시키자. 나는 데이터를 포함시켜서 올렸기 때문에 안 해도 된다.
    5. 설정이 끝났고, 웹 메뉴로 가서 Add new App 을 하고, manual configuration → 3.7 → Code 에다가 source code 경로를 써주자.  `/home/academey/fc_community`
    6. 그리고 wsgi 파일 누르면 막 긴 설정이 뜨는데, 다른 거 다 주석처리하고 쭉 내려가다 DJANGO 써져있는 게 있다. 다음과 같이 조금 커스터마이징 해주자.
    ```python
        # +++++++++++ DJANGO +++++++++++
        # To use your own django app use code like this:
        import os
        import sys
        
        # assuming your django settings file is at '/home/academey/mysite/mysite/settings.py'
        # and your manage.py is is at '/home/academey/mysite/manage.py'
        path = '/home/academey/fc_community'
        if path not in sys.path:
            sys.path.append(path)
        
        os.environ['DJANGO_SETTINGS_MODULE'] = 'fc_community.settings'
        
        # then:
        from django.core.wsgi import get_wsgi_application
        application = get_wsgi_application()
    ```
    7. 이제 소스 코드 설정이 되었는데, python everywhere 에 가상환경도 설정 해주자. `/home/academey/fc_env` 

    8. 다 된 줄 알았겠지만, static 을 설정해줘야 스타일이 설정된다.
    

{% endraw %}