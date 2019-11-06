---
layout: post
title: "Django Tutorial 04. 로그인"
author: academey
categories: django
cover: "/assets/django-tutorial/cover.jpeg"
---
{% raw %}
## 0. 들어가며
Fast campus 의 [파이썬 웹 개발 올인원 패키지](https://www.fastcampus.co.kr/dev_online_pyweb/) 강좌를 학습하며 정리한 글입니다. 글 내용 중 문제가 있거나 문의하실 내용이 있으면 댓글을 남겨주세요!

## 4. 로그인

1. 세션이란
    1. 클라이언트가 서버에 요청을 보내면, 서버가 쿠키라는 값을 보내준다.
    2. 그 쿠키 값을 클라이언트(브라우져)의 웹 사이트마다 다른 저장공간에 넣어둔다.
    3. 이후부터는 그 쿠키 값을 서버에 같이 보내주고, 서버는 그 쿠키 값을 **데이터베이스에 저장해놓고** 같은 클라이언트인지 파악할 수 있게 된다.
    4. 클라이언트의 로그인 정보를 유지할 수 있다. 현재 로그인한 유저의 정보를 파악할 수 있게 된다.
2. 로그인 만들기
    1. 가입 폼 그대로 가져다 쓰자. login.html 만들고 인풋 몇개 바꿔주자.
    2. 그리고 뷰 로직을 만들자. 기본적인 형태는 가입과 비슷하다.  
        ```python
        from django.contrib.auth.hashers import make_password, check_password
        
        def login(request):
            if request.method == 'GET':
                return render(request, 'login.html')
            elif request.method == 'POST':
                username = request.POST.get('username', None)
                password = request.POST.get('password', None)
        
                res_data = {}
        
                if not (username and password):
                    res_data['error'] = '모든 값을 입력해야 합니다.'
                else:
                    fcuser = Fcuser.objects.get(username=username)
                    if check_password(password, fcuser.password):
                        # 비밀번호 일치 로그인 성공
        								# 세션 만들어줘야 하고, 홈으로 리다이렉트 시켜야 함!
                        pass
                    else:
                        res_data['error'] = '비번 틀림'
                return render(request, 'login.html', res_data)
        ```
    3. 로그인에 성공하면 유저 정보를 세션에 유지시키고, 홈으로 리다이렉트 시켜줘야 한다. 그러기 위해 홈부터 만들어보자. fc_community/urls.py 에 다음과 같이 추가한다. 
        ```python
        from django.contrib import admin
        from django.urls import path, include
        from fcuser.views import home
        
        urlpatterns = [
            path('admin/', admin.site.urls),
            path('fcuser/', include('fcuser.urls')),
            path('', home),
        ]
        ```

    4. 그리고 다음과 같이 views 코드를 추가한다. 
        ```python
        def home(request):
            user_id = request.session.get('user')
            if user_id:
                fcuser = Fcuser.objects.get(pk=user_id)
                return HttpResponse(fcuser.username)
            return HttpResponse("home!")
        def login(request):
            if request.method == 'GET':
                return render(request, 'login.html')
            elif request.method == 'POST':
                username = request.POST.get('username', None)
                password = request.POST.get('password', None)
        
                res_data = {}
        
                if not (username and password):
                    res_data['error'] = '모든 값을 입력해야 합니다.'
                else:
                    fcuser = Fcuser.objects.get(username=username)
                    if check_password(password, fcuser.password):
                        # 비밀번호 일치 로그인 성공
                        # TODO: 세션 처리도 해줘야 함
                        request.session['user'] = fcuser.id
        ```

    5. 그리고 나서 크롬 스토리지를 보면 쿠키 값에 세션 아이디가 있다. 장고 서버에 요청했을 때 브라우져마다 다른 값을 들고 있게 된다. 그리고 그 값을 이용해서 식별하게 된다. 다른 세션 공간에 저장해둔다. 이걸 직접 구현해야 하는데, 장고는 자동으로 다 구현을 해놨음. 

    ![django session](/assets/django-tutorial/django_session.png)

    6. 로그아웃은, session의 user 데이터를 없애기만 하면 된다. 
        ```python
        def logout(request):
            if request.session.get('user'):
                del(request.session['user'])
                
                return redirect('/')
        ```

3. MTV의 T 확장하기 - 상속
    1. 로그인과 회원가입에서 계속 같은 코드들이 사용되고 있다. 헤드 쪽의 코드들을 다 계속 매번 쓸 수 없으니까 템플릿을 가져와서 조금씩 변경하도록 만들자. 
        ```html
        <html>
        <head>
            <link rel="stylesheet" href="/static/bootstrap.min.css"/>
            <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"
                    integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo"
                    crossorigin="anonymous"></script>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"
                    integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1"
                    crossorigin="anonymous"></script>
            <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"
                    integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM"
                    crossorigin="anonymous"></script>
        </head>
        <body>
        <div class="container">
            {% block contents %}
            {% endblock %}
        </div>
        </body>
        
        </html>

        {% extends "base.html" %}
        
        {% block contents %}
        <div class="row">
            <div class="col-12">
                <h1>로그인</h1>
            </div>
        </div>
        <div class="row mt-5">
            <div class="col-12">
                {{error}}
            </div>
        </div>
        <div class="row">
            <div class="col-12">
                <form method="POST" action="."> <!-- 현재 액션으로 POST 날림 -->
                    {% csrf_token %} <!-- 서버 전달시 csrf 공격 방지 용 -->
                    <div class="form-group">
                        <label for="username">사용자 이름</label>
                        <input type="text" class="form-control" id="username" placeholder="사용자 이름" name="username">
                        <small id="emailHelp" class="form-text text-muted">We'll never share your email with anyone
                            else.</small>
                    </div>
                    <div class="form-group">
                        <label for="password">비밀번호</label>
                        <input type="password" class="form-control" id="password" placeholder="비밀번호" name="password">
                    </div>
                    <button type="submit" class="btn btn-primary">로그인</button>
                </form>
            </div>
        </div>
        {% endblock %}
        ```
4. Form 활용하기
    1. 장고에서 Form 이라는 기능을 제공하는데, 폼 태그에 필요한 필드들을 관리하고 만들어주는 기능이다.
    2. 우선 로그인에 있던 필드를 없애고 [forms.py](http://forms.py) 를 만들어보자. 
        ```python
        from django import forms
        
        class LoginForm(forms.Form):
            username = forms.CharField(max_length=32)
            password = forms.CharField()
        ```

    3. 그리고 이 폼을 뷰단에서 내려주고 템플릿에서도 쓴다. 
        ```python
        def login(request):
        		form = LoginForm()
            return render(request, 'login.html', {'form': form})
        
        # login.html
        <form action ...>
        		{{ form }}
        </form>
        ```

    저 폼을 여러 태그들로 변경해서 출력할 수도 있다. form.as_p 뭐 이런 식으로... 

    4. 좀 더 커스터마이징 해보자! 
        ```html
        {% for field in form %}
        <div class="form-group">
            <label for="{{ field.id_for_label }}">{{ field.label }}</label>
            <input type="{{ field.field.widget.input_type }}" class="form-control" id="{{ field.id_for_label }}" placeholder="{{ field.label }}" name="{{ field.name }}" />
        </div>
        {% endfor %}
        ```

    요렇게 하면 각 필드에 정의된 값대로 라벨과 인풋이 입력되어진다. 그런데 아직 비밀번호는 암호화가 안 되어 있고, 라벨도 변경해보자.
    ```python
    from django import forms
    
    class LoginForm(forms.Form):
        username = forms.CharField(max_length=32, label="사용자 이름")
        password = forms.CharField(widget=forms.PasswordInput, label="비밀번호")
    ```

    5. 이제 폼을 검증해보자. 그 전에 백엔드 로그인 로직에서 검증 로직을 넣자.

    form 객체를 검증하는 is_valid 함수가 있어서 다음과 같이 작성하면, 알아서 검증해준다.

    그리고 에러가 존재한다면 각 필드에 자동으로 넣어준다!! 현재 존재하는 검증은 값이 비어있는지 아닌지가 있다.
    ```python
    form = LoginForm()
        if request.method == 'POST':
            form = LoginForm(request.POST)
            if form.is_valid(): 
                            # session
                return redirect('/')
        else:
            form = LoginForm() # 빈 폼을 전달해서 뷰에 렌더링시켜ㅑ줌
    
    # 템플릿단에서는 다음과 같이!
    
    {% if field.errors %}
    <span style="color: red">{{ field.errors }}</span>
    {% endif %}
    ```

    이제 비밀번호가 일치하는지 안 일치하는지 체크해야 한다. 그걸 LoginForm 에 추가해보자. 에러 메시지도 한글로 바꿔준다!

    clean 메소드는 폼에서 기본적으로 검증하는 함수인데 그 함수를 통해 값을 검증하고 돌려준다.
    ```python
        username = forms.CharField(max_length=32, label="사용자 이름",
        error_messages={
            'required': '아이디를 입력해주세요'
        })
        password = forms.CharField(widget=forms.PasswordInput, label="비밀번호",
            error_messages={
                'required': '비밀번호를 입력해주세요'
            })
            
        def clean(self):
        cleaned_data = super().clean()
        username = cleaned_data.get('username')
        password = cleaned_data.get('password')

        if username and password:
            fcuser = Fcuser.objects.get(username=username)
            if not check_password(password, fcuser.password):
                self.add_error('password', '비밀번호가 틀렸습니다') # 특정 필드에 에러를 넣는 함수. Form 이 가지고 있음
    ```

    이렇게 폼을 쓰게 되면, 폼내에서 검증 로직을 책임지고, 뷰 로직이 굉장히 간단해졌다!

    6. 세션에 유저를 저장해보자. 그런데 뷰 로직에서 user를 접근할 수가 없다. request.POST.get('username', None) 으로 막 유저 찾아서 아이디를 넣어줘야 하나..? 이것도 폼에서 다뤄보자! 
        ```python
        # 폼로직
        if not check_password(password, fcuser.password):
            self.add_error('password', '비밀번호가 틀렸습니다') # 특정 필드에 에러를 넣는 함수. Form 이 가지고 있음
        else:
            self.user_id = fcuser.id
        #뷰로직
        if form.is_valid(): # 내재된 검증 함수
            request.session['user'] = form.user_id
        return redirect('/')
        ```

    form 에서 비밀번호가 맞은 경우는 user_id 를 인스턴스 변수로 담아서 건내준다! 그리고 뷰 로직에서는 폼을 검증한 이후에 넣어준 user_id를 session 에 담아주면 된다!

{% endraw %}