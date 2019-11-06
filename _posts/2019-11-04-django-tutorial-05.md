---
layout: post
title: "Django Tutorial 05. 게시판"
author: academey
categories: django
cover: "/assets/django-tutorial/cover.jpeg"
---
{% raw %}
## 0. 들어가며
Fast campus 의 [파이썬 웹 개발 올인원 패키지](https://www.fastcampus.co.kr/dev_online_pyweb/) 강좌를 학습하며 정리한 글입니다. 글 내용 중 문제가 있거나 문의하실 내용이 있으면 댓글을 남겨주세요!

**05. 게시판**

1. ListView 만들기
    1. 게시판을 만들어보자. fcuser 가 만든걸 똑같이 만들면 되니까, 우선 templates 의 base.html 을 복사해온다. 
    2. 그리고 board list 를 만들어보자. 우선 기본적으로 우리가 만들었던 것 처럼 MVT 구성해보자. board/templates/board_list.html 에 다음과 같은 코드를 넣자. 
        ```html
        {% extends "base.html" %}
        {% block contents %}
        <div class="row at=5">
            <div>
                <table class="talbe">
                    <thead>
                        <tr>
                            <th>#</th>
                            <th>제목</th>
                            <th>아이디</th>
                            <th>일시</th>
                        </tr>
                    </thead>
                    <tbody class="text-dark">
                        <tr>
                            <th>1</th>
                            <th>제목 테스트</th>
                            <th>fcuser</th>
                            <th>2015-233232</th>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="row">
                <div class="col-12">
                    <button class="btn btn-primary">글쓰기</button>
                </div>
            </div>
        </div>
        {% endblock %}
        ```

    3. 이제 [urls.py](http://urls.py) 에 다음과 같이 추가하고, 

        path('board/', include('board.urls')),

    4. board/urls.py 에다 뷰를 추가해준다.
    ```python
    from django.contrib import admin
    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('list/', views.board_list),
    ]
    ```

    5. 마찬가지로 뷰 로직을 추가하는데, 그 전에 모델 먼저 만들자. 이 때, writer 는 모델을 연결해보고자 한다. 외래키를 이용해서 연결하려면 models.ForeingKey 로 사용한다. 이 때, on_delete 를 꼭 설정해줘야 한다. CASCADE, PROTECT 등 옵션이 있는데 필요하면 찾아서 보자. 
    ```python
    from django.db import models
    
    class Board(models.Model):
        title = models.CharField(max_length=128, verbose_name='제목')
    
        contents = models.TextField(verbose_name='내용')
    
        # 외래키로 id로 연결한다. 이 때, on_delete 라는 옵션을 꼭 넣어줘야한다. CASCADE 옵션으로 연쇄적으로 삭제된다.
        writer = models.ForeignKey('fcuser.Fcuser', on_delete=models.CASCADE, verbose_name='작성자')
    
        registered_dttm = models.DateTimeField(auto_now_add=True, verbose_name='등록시간')
    
        def __str__(self):
            return self.title
        class Meta:
            db_table = 'fastcampus_board'
            verbose_name = '패스트캠퍼스 게시글'
            verbose_name_plural = '패스트캠퍼스 게시글들'
    ```

    6. 그리고 makemigrations → migrate 를 돌리고, 모델에 추가되었는지 어드민에서 확인해보자. 어드민에서도 마찬가지로 추가하자.
    ```python
    from django.contrib import admin
    from .models import Board
    
    class BoardAdmin(admin.ModelAdmin):
        list_display = ('title',)
        pass
    
    admin.site.register(Board, BoardAdmin)
    ```

    7. 이제 저 모델들을 뷰에서 내려주고, 그걸 템플릿에서 표기해보자. 
        ```python
        # views.py
        def board_list(request):
            boards = Board.objects.all().order_by('-id') # 생성된 순서 역순으로 가지고 오겠다.
        
            return render(request, 'board_list.html', { 'boards': boards })
        # board_list.html
        <tbody class="text-dark">
            {% for board in boards %}
            <tr>
                <th> {{ board.id }}</th>
                <th> {{ board.title }}</th>
                <th> {{ board.writer }}</th>
                <th> {{ board.id }}</th>
            </tr>
            {% endfor %}
        </tbody>
        ```

2. write & detail 뷰 만들기
    1. 화면부터 만들자! board_write.html 을 만들자. 로그인에서 사용했던 view 의 form 을 그대로 써보자.  
        ```html
        {% extends "base.html" %}
        {% block contents %}
        <div class="row">
            <div class="col-12">
                <form method="POST" action="."> <!-- 현재 액션으로 POST 날림 -->
                    {% csrf_token %} <!-- 서버 전달시 csrf 공격 방지 용 -->
                    {% for field in form %}
                    <div class="form-group">
                        <label for="{{ field.id_for_label }}">{{ field.label }}</label>
                        <input type="{{ field.field.widget.input_type }}" class="form-control" id="{{ field.id_for_label }}" placeholder="{{ field.label }}" name="{{ field.name }}" />
                    </div>
                    {% if field.errors %}
                    <span style="color: red">{{ field.errors }}</span>
                    {% endif %}
                    {% endfor %}
                    <button type="submit" class="btn btn-primary">글쓰기</button>
                </form>
            </div>
        </div>
        {% endblock %}
        ```

    2. 그리고 이 안에서 쓰이는 forms.py을 만들자.
        ```python
        from django import forms
        
        
        class BoardForm(forms.Form):
            title = forms.CharField(max_length=128, label="제목",
                                    error_messages={
                                          'required': '제목을 입력해주세요'
                                    })
            contents = forms.CharField(widget=forms.Textarea, label="내용",
                                       error_messages={
                                           'required': '내용을 입력해주세요'
                                       })
            # 빈칸 검증을 제외한 검증은 따로 필요없을 것 같으니 clean 함수는 없앤다.
        ```

    3. 그런데 이렇게 되면, form 의 내용 부분이 textfield 처럼 표시되어 있지 않고 그냥 char field 랑 동일하게 렌더링 되어 있다. 이 부분을 해결해보자. textarea 는 input 의 type 프로퍼티를 넣어서 해결되는 게 아니라 아예 태그를 다른 걸 써야하기 때문에 if 문을 템플릿에 넣어줘야 한다.

    ![wrong_input](/assets/django-tutorial/wrong_input.png)

    ```html
    <div class="form-group">
        <label for="{{ field.id_for_label }}">{{ field.label }}</label>
        {{ field.field.widget.name }}
        {% ifequal field.name 'contents' %}
        <textarea class="form-control" name="{{ field.name }}" placeholder="{{ field.label }}"></textarea>
        {% else %}
        <input type="{{ field.field.widget.input_type }}" class="form-control" id="{{ field.id_for_label }}"
                placeholder="{{ field.label }}" name="{{ field.name }}"/>
        {% endifequal %}
    </div>
    ```

    4. 이제, 로그인한 사용자가 쓸 수 있도록 해야 한다. 뷰에 로그인 검증 로직을 추가하고, 해당 로그인한 유저를 board.writer 에 박아넣고 board를 저장하는 코드를 추가해보자. 

    ```python
    from fcuser.models import Fcuser
    
    def board_write(request):
        if not request.session.get('user'):
            return redirect('/fcuser/login')
        if request.method == 'POST':
            form = BoardForm(request.POST)
            if form.is_valid(): # 내재된 검증 함수
                user_id = request.session.get('user')
                fcuser = Fcuser.objects.get(pk=user_id)
    
                board = Board()
                board.title = form.cleaned_data.get('title')
                board.contents = form.cleaned_data.get('contents')
                board.writer = fcuser
                board.save()
    
                return redirect('/board/list')
        else:
            form = BoardForm() # 빈 폼을 전달해서 뷰에 렌더링시켜ㅑ줌
    
        return render(request, 'board_write.html', { 'form': form })
    ```

    5. 이제 글 상세 페이지를 만들어보자. 이 때, input 태그에 readonly 옵션을 넣어주자. board 값들을 렌더링해주는 디테일 페이지이므로, 특정 board 를 나타내는 pk를 urls.py에서 받아줘야 한다. 그 pk로 Board를 가져와서 템플릿에 내려준다.
    ```python
    # urls.py
        path('detail/<int:pk>', views.board_detail),
    
    # views.py
    from django.shortcuts import render, redirect, Http404
    
    def board_detail(request, pk):
        try:
            board = Board.objects.get(pk=pk)
        except Board.DoesNotExist:
            raise Http404('게시글을 찾을 수 없습니다.')
    
        return render(request, 'board_detail.html', { 'board': board})
    
    # board_detail.html
    {% extends "base.html" %}
    {% block contents %}
    <div class="row">
        <div class="col-12">
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" value="{{ board.title }}" readonly />
                <label for="contents">내용</label>
                <textarea class="form-control" id="contents" readonly>{{ board.contents }}</textarea>
            </div>
            <button class="btn btn-primary">돌아가기</button>
        </div>
    </div>
    {% endblock %}
    ```
3. 예외처리 추가하기
    1. 로그인 시, 존재하지 않는다면 예외를 발생시키도록 해야 한다. 
    ```python
    # fcuser/forms.py
    def clean(self):
            cleaned_data = super().clean()
            username = cleaned_data.get('username')
            password = cleaned_data.get('password')
    
            if username and password:
                try:
                    fcuser = Fcuser.objects.get(username=username)
                except Fcuser.DoesNotExist:
                    self.add_error('username', '존재하지 않는 아이디입니다')
                    return
    
                if not check_password(password, fcuser.password):
                    self.add_error('password', '비밀번호가 틀렸습니다') # 특정 필드에 에러를 넣는 함수. Form 이 가지고 있음
                else:
                    self.user_id = fcuser.id
    ```

    2. 마찬가지로, board_detail 에서도 존재하지 않는 pk로 들어오면 에러를 뱉어줘야 한다.  
    ```python
    try:
        board = Board.objects.get(pk=pk)
    except Board.DoesNotExist:
        raise Http404('게시글을 찾을 수 없습니다.')
    ```

    3. board_write 에서는, 사용자가 없으면 에러를 뱉어줘야 한다. (board.writer 에 넣을 fcuser가 없으니까!) 
    ```python
    def board_write(request):
        if not request.session.get('user'):
            return redirect('/fcuser/login')
    ```

4. Pagination
    1. 템플릿에 row를 하나 더 만들어서 페이지네이션 링크를 넣어주자. 
    ```html
    <div class="row mt-2">
        <div class="col-12">
            <nav>
                <ul class="pagination justify-content-center">
                    <li class="page-item">
                        <a class="page-link" href="#">이전으로</a>
                    </li>
                    <li class="page-item active">
                        <a class="page-link" href="#">1 /1</a>
                    </li>
                    <li class="page-item">
                        <a class="page-link" href="#">다음으로</a>
                    </li>
                </ul>
            </nav>
        </div>
    </div>
    ```

    2. 그리고 뷰로 넘어와서 구현하자. 장고는 Paginatior 라는 클래스를 제공한다. 요걸 이용해서 paging 기능을 구현할 수 있다. 원래 all_boards는 쿼리셋이라는 데이터 형태였는데, Paginator 를 통해 생성되면서 pagination 정보를 들고 있다. 이를 활용해서 템플릿에서 보여주자. 
    ```python
    from django.core.paginator import Paginator
    
    def board_list(request):
        all_boards = Board.objects.all().order_by('-id') # 생성된 순서 역순으로 가지고 오겠다.
            page = int(request.GET.get('p', 1))  # string -> int
        paginator = Paginator(all_boards, 2) # 한 페이지에 2개씩!
        
        boards = paginator.get_page(page)
        return render(request, 'board_list.html', { 'boards': boards })
    ```

    3. view 단의 코드들은 이렇게 된다. boards의 프로퍼티들을 이용해서 태그들을 보여준다.
    ```html
    <nav>
        <ul class="pagination justify-content-center">
            {% if boards.has_previous %}
            <li class="page-item">
                <a class="page-link" href="?p={{ boards.previous_page_number }}">이전으로</a>
            </li>
            {% else %}
            <li class="page-item disabled">
                <a class="page-link" href="#">이전으로</a>
            </li>
            {% endif %}
            <li class="page-item active">
                <a class="page-link" href="">{{ boards.number }} / {{boards.paginator.num_pages}}</a>
            </li>
            {% if boards.has_previous %}
            <li class="page-item">
                <a class="page-link" href="?p={{ boards.next_page_number }}">다음으로</a>
            </li>
            {% else %}
            <li class="page-item disabled">
                <a class="page-link" href="#">다음으로</a>
            </li>
            {% endif %}
        </ul>
    </nav>
    ```

5. 리뷰 및 프로젝트 보완
    1. 실제 사이트라면, UI를 통해 연결해줘야 하는 기능들이 있다. 홈에다 로그인, 게시물보기, 회원가입 등을 넣어보자. 로그인 여부에 따라 로그아웃 혹은 로그인, 회원가입을 보여주는 페이지다. 이 때, request 객체를 템플릿에서 접근할 수 있음에 유의하자.

    ```html
    {% extends "base.html" %}
    {% block contents %}
    <div class="row">
        <div class="col-12">
            <h1>홈페이지</h1>
        </div>
    </div>
    <div class="row mt-5">
        {% if request.session.user %}
        <div class="col-6">
            <button class="btn btn-primary btn-block" onclick="location.href='/fcuser/logout/'">로그아웃</button>
        </div>
        {% else %}
        <div class="col-6">
            <button class="btn btn-primary btn-block" onclick="location.href='/fcuser/login/'">로그인</button>
        </div>
        <div class="col-6">
            <button class="btn btn-primary btn-block" onclick="location.href='/fcuser/register/'">회원가입</button>
        </div>
        {% endif %}
    </div>
    <div class="row mt-1">
        <div class="col-12">
            <button class="btn btn-primary btn-block" onclick="location.href='/board/list/'">게시물보기</button>
        </div>
    </div>
    {% endblock %}
    ```

    2. 그리고 게시판 글쓰기, 리스트, 디테일 뷰에다 각각 버튼을 만들어준다.
    ```html
    <button class="btn btn-primary" onclick="location.href='/board/write/'">글쓰기</button>
    <button class="btn btn-primary" onclick="location.href='/board/list/'">돌아가기</button>
    
    # 리스트 뷰
    {% for board in boards %}
    <tr onclick="location.href='/board/detail/{{ board.id }}'">
        <th> {{ board.id }}</th>
        <th> {{ board.title }}</th>
        <th> {{ board.writer }}</th>
        <th> {{ board.registered_dttm }}</th>
    </tr>
    {% endfor %}
    ```

{% endraw %}