---
layout: post
title: "전자 출석체크 자동으로 하는 법 (Mac + Automator + Google Calendar)"
author: academey
categories: sanup
---

# 0. 들어가며

전자 출결체크를 해야 한다. 자주 까먹기도 하고 정말 귀찮다. 그래서 전자출결을 자동으로 하는 프로그램을 만들어보자. 물론 나는 **절대** 쓰지 않았다.

_글의 컨셉상 반말로 작성하는 점 양해 바랍니다._

# 1. Automator

자동화 프로그램을 알아보던 도중, Mac 에서는 기본으로 제공하는 훌륭한 프로그램이 있었다. Automator 라는 프로그램인데, 동영상 편집, Mail, 다른 응용프로그램 사용까지 다양한 액션을 할 수 있다.

_맥용 자동화 앱 중 Repeater는 쓰지 마라. 이미 deprecated 된지 오래라 정상 작동을 안한다._

## 1-1 ) 프로그램 실행

Automator 를 실행해서, 응용 프로그램을 선택하자.

![Automator execute](/assets/attendance-check-automator/automator_execute.png)

<center><U>Automator 실행화면</U></center>
&nbsp;

## 1-32 ) Applescript 블럭 추가

UI를 보면 대충 감이 올텐테, [스크래치](https://scratch.mit.edu/)같이 Drag & Drop 으로 입력, 결과, 실행 세가지 기능을 가진 블록을 추가해준 뒤 저장하면 프로그램으로 추출해준다. 우리는 단순하게 Applescript 만 쓸 거라 자세하게 알고 싶다면 [Automator 공식홈페이지](http://www.macosxautomation.com/automator/) 에서 알아보자.

보관함 -> 유틸리티 -> AppleScript 실행 블럭을 추가하자.

![add applescript](/assets/attendance-check-automator/add_applescript.png)

<center><U>Applescript 블럭 추가</U></center>
&nbsp;

## 1-3 ) Applescript 코드 파악

지금 이 글을 보는 당신과 마찬가지로, 나도 Applescript 를 처음 본다. 이 프로그램 하나 만들자고 문법을 공부하는 것 보다는, 구글링을 선택했다.

그 중 가장 최근 문서가 https://stackoverflow.com/questions/48836306/using-applescript-to-click-web-button-via-id 였는데, 실행되는 코드는 아래와 같다.

<pre><code>tell application "Google Chrome"
    activate
    --  # Wait until page finishes loading.
    repeat until (loading of active tab of front window is false)
        delay 1
    end repeat
    tell active tab of front window
        --  # Click the "Vote now" button for "Ahuroa School".
        execute javascript "document.getElementsByClassName('flotediv col-md-3 col-xs-12 col-sm-6 flexbutton ')[77].click();"
        delay 0.5
        --  # Click the "Vote" button.
        execute javascript "document.getElementsByClassName('votebutton')[0].click();"
        delay 0.5
        --  # Click the "Close" button.
        execute javascript "document.getElementsByClassName('btn btn-default')[0].click();"
        delay 0.5
    end tell
    quit
end tell
</code></pre>

Chrome을 실행할 외부 프로그램으로 지정한 다음 javascript 로 UI를 다루는 모습인데, 이와 비슷하게 짜면 될 것 같다. 우선 우리가 출석체크하고 싶은 사이트를 들어가보자.

## 1-4 ) 타겟 사이트 조작용 jquery 코드 작성

나는 이 카페의 출석 체크를 하고 싶다. 크롬 개발자 도구를 이용해서 클릭하고 싶은 곳의 element를 확인하자. 보면 id가 menuLink381이 박혀있다.

![target site](/assets/attendance-check-automator/target_site.png)

<center><U>타겟 사이트</U></center>
&nbsp;

<pre><code>document.getElementById('menuLink381').click();</code></pre>

이런 코드로 당신은 저 dom element를 클릭할 수 있다. 이제, Applescript 에 저 코드를 담도록 작성해보자. 대충 요런 느낌이다. (코드가 개판인 점은 양해를 바란다....)

<pre><code>on run {input, parameters}
tell application "Google Chrome"
open location "https://cafe.naver.com/webcenter"
delay 5
execute front window's active tab javascript "document.getElementById('menuLink381').click();"
end tell
return input
end run
</code></pre>

![apple_script_write](/assets/attendance-check-automator/applescript_write.png)

<center><U>Applescript 작성</U></center>

&nbsp;

실행 버튼을 눌러보면, 사이트에 들어가서 출석체크 메뉴 버튼을 누른다! 정상 작동한다. 물론 출석체크를 하려면 input 칸을 채우고 출석체크 버튼을 한 번 더 눌러야겠지만, 그런 기본적인 돔 조작은 여러분들이 할 수 있을것이라 믿는다.

이제 프로그램을 저장하면, automator 아이콘을 가진 프로그램이 생긴다.

## 1-6 ) 유의해야 할 점

이 프로그램을 실행해보면, 다음과 같은 설정이 뜨는데, 허가해줘야 다른 앱을 제어할 수 있게 되므로 허가해주자.
&nbsp;

![app_admit](/assets/attendance-check-automator/app_admit.png)

<center><U>App의 Chrome 사용권한 설정</U></center>

그리고, 크롬에서도 마찬가지로 Apple Evenets의 자바스크립트를 허용해줘야 한다. 보기 -> 개발자 정보 -> Apple Events의 자바스크립트 허용을 누르면 된다.
&nbsp;

![app_admit](/assets/attendance-check-automator/chrome_applescript_admit.png)

&nbsp;

<center><U>Chrome의 applescript 사용권한 설정</U></center>

# 2. Google Calendar

자, 이제 만든 프로그램을 주기적으로 실행시켜주는 Scheduller 앱이 필요하다. 파일을 주기적으로 실행시키는 방법은, 놀랍게도 구글 캘린더에서 가능하다.

_필자는 crontab, launchctl 등 서비스 관리 프레임워크들을 사용해봤으나 외부 애플리케이션 사용 권한을 해당 프레임워크들에게 주는 법을 몰라 삽질하다 실패했다.... 누군가 성공하면 알려주시길._

## 2-1 ) 일정 설정

일정을 만들고, 알림을 눌러보자. 알림에 사용자화를 보면 파일열기 목록이 있다. 여기서 파일 열기를 주기적으로 실행해준다. 정말 간단하다.
&nbsp;

&nbsp;

![alarm_user_defiened](/assets/attendance-check-automator/alarm_user_defiened.png)

<center><U>알람 사용자화</U></center>

그리고 일정 설정 중 반복을 설정하면, 나는 이제 매일 오후 1:30에 출석체크하는 프로그램을 등록하게 된 것이다.

![daily_auto_check](/assets/attendance-check-automator/daily_auto_check.png)

<center><U>매일 자동 출석 체크</U></center>

# 3. 맥 자동 시동 / 잠자기

추가적으로, 이 프로그램 실행을 하기 위해 Mac을 혼자 키고 끄고 싶다. 그렇다면 맥의 시스템 환경설정 -> 에너지 절약 -> 일정 을 가보면 다음과 같이 굉장히 쉽게 설정할 수 있다.

&nbsp;
![auto_run](/assets/attendance-check-automator/auto_run.png)

<center><U>맥 자동 실행</U></center>

다들 즐 출첵하길 바라며 글을 마친다.
