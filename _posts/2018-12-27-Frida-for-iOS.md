---
title: "Frida for iOS"
date: 2018-12-27 00:00:00 +0900
---

2018년 12월 iOS 버전이 12.1.2까지 배포되었다. 반면 iOS Jailbreak(탈옥)툴은 iOS 11.4 Beta3 까지 지원된다.
(iOS 11.4 Beta3 다운그레이드 불가) 현재 iOS 11.4.1 버전에 대한 Exploit을 Ian Beer가 가지고 있는 것으로 알려져 있고 Electra를 통해 배포될 것으로 예상하고 있지만 언제 배포될지 확실하지 않다.
새로운 버전의 iOS가 배포되면 이전 버전 iOS를 위한 Jailbreak 툴이 배포되었던 흐름에 맞추어 볼 때, 최근 iOS Jailbreak가 쉽지 않아졌음을 짐작할 수 있다. 
이 문서에서 설명하는 Frida는 Jailbreak 여부와 상관없이 iOS 애플리케이션을 분석하는데 사용할 수 있지만, 제한없는 사용을 위해 탈옥된 아이폰을 대상으로 한다.
따라서 iOS 11.4 Beta3 이하 버전의 탈옥된 아이폰을 사용하는 것이 좋다. 탈온된 아이폰에서 Frida를 이용한 애플리케이션(이하 앱) 분석 및 공격 방법을 알아보자.

# Jailbreak iPhone
iOS 버전에 따라 다양한 Jailbreak 툴을 사용할 수 있다. 가장 최신 버전의 iOS를 위한 Jailbreak 툴을 소개하자면, Coolstar의 Electra가 있다. 11.2 부터 11.4 Beta3 까지 지원한다. 가지고 있는 아이폰의 iOS 버전을 확인하고 그에 맞는 Jailbreak 툴을 이용하면 된다. 자신의 아이폰이 iOS 11.4.1 이상 버전이라면 다른 아이폰을 구해보도록 하자.

# Cydia
Cydia는 탈옥된 iOS 디바이스를 위한 앱 스토어다. 애플의 앱 스토어에서 설치할 수 없는 앱 또는 기능 개선을 위한 트윅(tweak)을 설치할 수 있다. iOS 앱 해킹을 위해 기본적으로 BigBoss Recommended Tools, OpenSSH, Frida와 같은 트윅을 설치한다. Jailbreak 툴을 사용해 탈옥을 하면 보통 자동으로 설치 되지만 간혹 함께 설치 되지 않는 경우도 있어 직접 설치해야 할 때도 있다. (Cydia를 만든 Jay Freeman이 곧 Cydia를 폐쇄 한다고 한다.)

# Install Frida Server on iPhone
Cydia에서 Frida를 설치해보자. frida.re에서 간단한 설치 방법을 확인할 수 있다. 
Sources(소스) 탭 -> Edit(편집) -> Add(추가) -> https://build.frida.re 

![00](./assets/images/posts/20181227FridaForiOS/00.png)

그리고 나면 Search(검색) 탭에서 Frida를 찾을 수 있다. 설치하면 된다.

현재 버전은 12.2.27이다. 이어서 설치할 Frida CLI Tools의 Major 버전이 일치해야 제대로 동작한다. Frida tools의 버전은 12.2.x 여야 한다.

# Install Frida’s CLI Tools on your Mac

Frida 서버에 스크립트를 전달하기 위해 Python을 사용한다.(Command 1) 이 때 사용할 Python용 Frida 라이브러리를 설치한다. Python 3 를 사용하여 전달하기로 한다.(Command 2)
아이폰에 설치된 Frida 서버와 통신하고 간단한 명령을 전달하기위해 사용자의 컴퓨터에 Frida 툴(Frida’s CLI tools)을 설치해야 한다. 설치를 위해 Python이 설치되어 있어야 하는데, Python 2.7을 사용하는 것을 추천한다. (Command 3,4)
## Commands:
1. brew install python3 (Python 3 설치(brew for Mac))
2. pip3 install frida (Python3 용 Frida 라이브러리 설치)
3. sudo easy_install pip (Python 2.7 패키지 관리자 설치 - 기본적으로 설치되어있지 않다.)
4. sudo -H pip install frida-tools (Python 2.7 버전의 Frida-tools 설치)
* pip3로 설치하지 않는 이유는 pip3의 frida-tools는 인코딩 버그가 있어 한글이 제대로 표시되지 않는다.
* Module six 관련 에러가 날 경우 sudo -H pip install frida-tools --ignore-installed six 로 설치

# Test Frida

여기까지 Frida를 실행할 준비를 마쳤다. 아이폰을 컴퓨터에 연결하고 frida-ps -U 명령을 실행해보자.
위 명령이 정상적으로 실행되었다면 현재 아이폰에서 동작중인 프로세스의 목록을 볼 수 있을 것이다. 앞으로 자주 사용하게 될 명령 frida-ps -Uai 도 실행해보자. 프로세스의 Identifier를 출력해준다. 주로 앱의 Identifier를 이용해 앱에 접근해 공격을 실행할 것이다.
