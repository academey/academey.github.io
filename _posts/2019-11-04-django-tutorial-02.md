---
layout: post
title: "Django Tutorial 02. MTV"
author: academey
categories: django
cover: "/assets/django-tutorial/cover.jpeg"
---
{% raw %}
## 0. 들어가며
Fast campus 의 [파이썬 웹 개발 올인원 패키지](https://www.fastcampus.co.kr/dev_online_pyweb/) 강좌를 학습하며 정리한 글입니다. 글 내용 중 문제가 있거나 문의하실 내용이 있으면 댓글을 남겨주세요!

## 2. MTV

1. MTV의 M 만들기 - 회원
    1. 모델의 필드를 설정하고, verbose_name 으로 admin에서 보여지는 네이밍을 설정할 수 있다.
    2. auto_now_add 라는 옵션을 통해 저장되는 시점이 자동으로 넣어진다.
    3. 테이블 명을 지정하고 싶으면 class Meta: 를 만들어서 설정해줄 수 있다.
    ```python
    from django.db import models
    
    class Fcuser(models.Model):
        username = models.CharField(max_length=64, verbose_name='사용자명')
        password = models.CharField(max_length=64, verbose_name='비밀번호')
        registered_dttm = models.DateTimeField(auto_now_add=True, verbose_name='등록시간')
    
        class Meta:
            db_table = 'fastcampus_fcuser'
    ```

2. 데이터베이스 관리
    1. 만든 모델을 마이그레이션 해준다. `python3 [manage.py](http://manage.py) makemigrations`
    2. 그러면 migrations/0001_initial.py 가 생성되는데, 디비에 들어가는 설정들이 만들어진다.
    3. `python3 [manage.py](http://manage.py) migrate` 를 실행하면 등록된 앱들의 모델들을 테이블로 만들어준다. 그러면 db.sqlite3 파일이 생기는데 요게 디비 파일이다.
    4. 생성된 디비를 확인하고 싶으면 `sqlite3 db.sqlite3` 을 통해서 접근하자. 그리고 `.tables` 를 치면 fastcampus_fcuser 가 생성된 걸 확인할 수 있다.
    5. 즉, 모델을 변경하고 마이그레이션 하면 알아서 변경점을 디비에 업데이트 시켜준다. 모델을 변경하고 다시 makemigrations 실행하고, migrate 를 실행하면 디비에 반영된다.

3. Admin 소개
    1. 메인 프로젝트의 [urls.py](http://urls.py) 을 보면 아래처럼 admin 이 기본으로 들어가 있다. admin 안에 여러 url이 정의되어 있어서, 저 하위의 path 로 접근하면 [admin.site](http://admin.site).urls, 하위의 url 패턴을 따라 간다는 의미다.
    ```python
    from django.contrib import admin
    from django.urls import path
    
    urlpatterns = [
        path('admin/', admin.site.urls),
    ]
    ```

    2. 들어가보려면 우선 장고를 띄워보자. `python3 [manage.py](http://manage.py/) runserver` ****을 하고 127.0.0.1:8000/admin 으로 접근해보자. 그러면 계정이 필요한데 아이디가 없다. `python3 manage.py createsuperuser` 를 하면 계정을 만들 수 있다.

4. Admin 활용
    1. 장고 어드민에 우리가 만든 모델들을 등록해보자. fcuser/[admin.py](http://admin.py) 에서 우리가 관리할 모델들을 등록할 수 있다. admin.py 에다가 다음과 같이 작성하면, 해당 모델을 CRUD 할 수 있는 페이지가 나온다.
    ```python
    from django.contrib import admin
    from .models import Fcuser
    
    class FcuserAdmin(admin.ModelAdmin):
        pass
    
    admin.site.register(Fcuser, FcuserAdmin)
    ```

    2. 그런데 요렇게 하면, Fcuser object(1) 이라고 나오고 영문으로 나와서 불편하다. 바꿔보자.

    3. 모델에 def __str__(self): return self.username 을 하면 admin 에서 유저 이름으로 표기 된다. 

    4. 그리고 [models.py](http://admin.py) 에 아래 코드를 추가해준다.
    ```python
    class Fcuser(models.Model):
        def __str__(self): # 객체 자체를 표기할 때 username으로 사용
            return self.username
        class Meta: # table 이름과 admin 표기 이름들을 명시
            db_table = 'fastcampus_fcuser'
            verbose_name = '패스트캠퍼스 사용자'
            verbose_name_plural = '패스트캠퍼스 사용자들'
    ```

5. MTV의 T, V 만들기 - 회원가입 1
    1. 우선 fc_user/templates/register.html 을 만들고 head 태그에 bootstrap 링크를 다 박아준다.
    2. 템플릿을 만들었으니 뷰를 만들어보자. fcuser/views.py 에 다음과 같이 추가한다. 
    ```python
    def register(request):
        return render(request, 'register.html') # 이 파일의 경로는 templates 을 보고 있으니 거기서 관리하면 될듯
    ```

    3. 그리고 route 를 연결해줘야 하므로 fc_community/urls.py 에 다음과 같이 추가한다. 
    ```python
    from django.contrib import admin
    from django.urls import path, include
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('fcuser/', include('fcuser.urls'))
    ]
    ```

    4. 그리고 route 를 연결해줘야 하므로 fc_community/urls.py 에 다음과 같이 추가한다. 
    ```python
    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('register/', views.register)
    ]
    ```

    그러면 fcuser/register 에 해당 메소드를 리턴할 것이다.

6. MTV의 T, V 만들기 - 회원가입 2
    1. 폼 설정을 해주자. form method="POST" action="." (현재 url 에 전송한다) 그리고 폼 내부에 {% csrf_token %} 을 써줘야 폼 액션에서 해킹에 안전할 수 있다.
    2. 그러면 이제 GET 방식과 POST 방식 둘 다 /register 에서 받고 있기 때문에 추가적으로 메소드를 만들어주고 연결해줘야 한다. 아래처럼 Fcuser.save() 를 통해 객체가 만들어짐을 확인할 수 있다.

    ```python
    from django.shortcuts import render
    from .models import Fcuser
    # Create your views here.
    
    def register(request):
        if request.method == 'GET':
            return render(request, 'register.html')  # 이 파일의 경로는 templates 을 보고 있으니 거기서 있으면 연결해준다.
        if request.method == 'POST':
            username = request.POST['username']
            password = request.POST['password']
            re_password = request.POST['re-password']
    
            fcuser = Fcuser(
                username=username,
                password=password
            )
            fcuser.save()
            return render(request, 'register.html')
    ```

    3. 비밀번호가 같은지 아닌지 데이터를 templates 으로 넘겨주자. 그리고 make_password 를 이용해서 비밀번호를 암호화시켜 저장하자. 그리고 인풋값에 대한 예외처리도 해줘야 한다. 
        ```python
        from django.http import HttpResponse
        from django.shortcuts import render
        from .models import Fcuser
        from django.contrib.auth.hashers import make_password
        # Create your views here.
        
        def register(request):
            if request.method == 'GET':
                return render(request, 'register.html')  # 이 파일의 경로는 templates 을 보고 있으니 거기서 있으면 연결해준다.
            if request.method == 'POST':
                username = request.POST.get('username', None)
                password = request.POST.get('password', None)
                re_password = request.POST.get('re-password', None)
        
                res_data = {}
        
                if not (username and password and re_password):
                    res_data['error'] = '모든 값을 입력해야 합니다.'
                elif password != re_password:
                    res_data['error'] = '비밀번호가 다릅니다'
                else:
                    fcuser = Fcuser(
                        username=username,
                        password=make_password(password)
                    )
                    fcuser.save()
        
                return render(request, 'register.html', res_data)
        ```
7. MTV의 T, V 만들기 - 회원에 필드 추가하기
    1. useremail 을 추가해보자. 모델에 useremail 필드 추가하고 makemigrations 하려는데, 기본값이 필요하다고 한다. 필드에다가 default 옵션을 추가하던지, 그냥 콘솔 창에 추가하던지 둘 중 하나로 기본값을 넣어준다.
    2. 그리고 템플릿과 뷰단 로직을 추가해준다. 

{% endraw %}