---
layout: post
title: "Django Tutorial 06. 태그 (M:N)"
author: academey
categories: django
cover: "/assets/django-tutorial/cover.jpeg"
---
{% raw %}
## 0. 들어가며
Fast campus 의 [파이썬 웹 개발 올인원 패키지](https://www.fastcampus.co.kr/dev_online_pyweb/) 강좌를 학습하며 정리한 글입니다. 글 내용 중 문제가 있거나 문의하실 내용이 있으면 댓글을 남겨주세요!

## 6. 태그

1. 태그 app & 모델 만들기
    1. Foreign Key 를 사용했었는데, 이는 1:N 관계를 나타내기 위함이다. 예를 들어 작성자 1 : 게시글 N 관계를 사용할 때 쓴다. 그런데 태그같은 경우는 여러 게시글에 여러 태그를 쓸 수 있어서 M:N 관계를 활용해야 한다. 
    2. 이를 구현하기 위해서 tag 라는 앱을 추가해보자. `python3 [manage.py](http://manage.py) startapp tag` 
    3. 이제 생성된 앱에 모델을 만들고, board 에 ManyToManyField 로 추가하자.
        ```python
        from django.db import models
        
        # Create your models here.
        
        
        class Tag(models.Model):
            name = models.CharField(max_length=32, verbose_name='태그명')
            registered_dttm = models.DateTimeField(auto_now_add=True, verbose_name='등록시간')
        
            def __str__(self):
                return self.name
        
            class Meta:
                db_table = 'fastcampus_tag'
                verbose_name = '패스트캠퍼스 태그'
                verbose_name_plural = '패스트캠퍼스 태그'

        from django.db import models
        
        class Board(models.Model):
        	...
        	tags = models.ManyToManyField('tag.Tag', verbose_name='태그')
        	...
        ```

    그리고 마찬가지로 어드민에 추가하고, APPS 에 추가하고, makemigrations → migrate 를 돌리고 확인해보자. 이제 태그가 보인다! 태그를 만들고, 특정 게시글에 추가하자.

    4. 추가된 태그들을 보여주는 코드 먼저 만들어보자. for 문을 돌면서 표기할 수 있지만, 함수를 사용해서 한번에 출력하는 방법도 있다. join 을 써서 스트링으로 보여주는 방법이다.  
        ```html
        <label for="tags">태그</label>
            <span id="tags" class="form-control">
                {% for tag in board.tags.all %}
                {{ tag.name }}
                {% endfor %}
                {{ board.tags.all|join:", "}}
            </span>
        </label>
        ```

2. 태그 만들기
    1. 게시판 글을 쓸 때 태그도 만들 수 있도록 해보자. Form 을 이용해서 게시판 글을 썼으니 태그도 마찬가지로 Form 을 쓰면 된다. 
    ```python
    tags = forms.CharField(required=False, label="태그")
    ```

    요렇게 넣으면 인풋 태그가 생기는데, 이 값을 이용해서 모델을 생성해야 한다. '태그1, 태그2' 라고 오면 그 값 그대로 사용하는게 아니라 ,을 잘라서 사용해야 하므로 split 해서 가져온다.
    ```python
    tags = form.cleaned_data['tags'].split(',')
    ```
    그리고 이 태그들을 보면서, 태그가 없다면 만들어주고 태그가 있다면 가져와서 그대로 모델을 사용하면 된다. 그 로직을 장고에서는 get_or_create 로 구현해놨다. 만약(name = tag 조건이 일치한다면 가져오고, 맞지 않다면 생성해준다. 생성되면 crated를 boolean True 로 리턴하기 때문에 고걸로 케이스를 제어할 수도 있다.
    
    ```python
    if form.is_valid():  # 내재된 검증 함수
        user_id = request.session.get('user')
        fcuser = Fcuser.objects.get(pk=user_id)
    
        tags = form.cleaned_data['tags'].split(',')
    
        board = Board()
        board.title = form.cleaned_data.get('title')
        board.contents = form.cleaned_data.get('contents')
        board.writer = fcuser
        board.save()
        
        for tag in tags:
            if not tag:
                continue
            # name = tag 조건이 일치한다면 가져오고, 맞지 않다면 생성해준다.
            # 생성되면 crated를 boolean True 로 리턴하기 때문에 고걸로 케이스를 제어할 수도 있다.
            _tag, _created = Tag.objects.get_or_create(name=tag)
            # if _created ~~ 로직 넣을수도 있다는 말임.
    
            # 만약 board.save 전에 add를 하면 에러가 난다. 왜냐면 board를 생성할 때 id가 만들어지는데, 그 이후에
            # m:n 관계를 연결할 수 있기 때문이다.
            board.tags.add(_tag)
        
    
        return redirect('/board/list')
    ```

    주의!! 만약 board.save 전에 add를 하면 에러가 난다. 왜냐면 board를 생성할 때 id가 만들어지는데, 그 이후에 m:n 관계를 연결할 수 있기 때문이다.

{% endraw %}