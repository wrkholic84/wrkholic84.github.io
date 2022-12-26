---
title: Tor as a SOCKS Proxy
# author: wrkholic84
date: 2020-01-12 00:00:00 +0900
categories: [Security, Web]
tags: [proxy]
math: true
mermaid: true
# image:
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---
요즘 웹 사이트엔 정말 다양한 보안 정책들이 적용되어 있다. 그 중 IP 차단 정책은 모의해킹을 수행할 때 있어서 매우 골칫거리가 아닐 수 없다. 

IP 차단 정책을 우회할 수 있는 방법이 몇가지 있다. 예를 들자면, VPN을 사용하면 된다. 하지만 VPN은 사실 상 유료 서비스이기 때문에 사용하는데 부담이 된다. 그 대안으로 Tor가 있다. Tor는 무료이고 HTTP를 포함한 모든 프로토콜을 사용할 수 있는 장점이 있다.

## Tor
The Onion Router의 약자다. 한국에선 ‘토르’라고 부르는데 사실 ‘토어’라고 부르는게 맞다. 아무튼 Tor는 전달된 데이터를 익명으로 처리하도록 디자인된 인터넷 네트워크 프로토콜이다. Tor의 동작원리를 보면 알 수 있듯이 목적지까지 한 번에 통신하지 않고, 중간에 여러 노드들을 거쳐 보내기 때문에 최종 목적지에서는 내 IP를 숨길 수 있다.

그렇다 하더라도 IP 차단은 피할 수 없다. 여기서 Tor의 진가가 발휘되는데 Exit Node가 목적지로부터 차단 당한 경우 Tor는 새로운 Exit Node를 할당하여 접속할 수 있도록 만들어 준다.  그리고 차단 당한 Node는 Bridges<sup id="a1">[1](#footnote1)</sup> 로 다시금 역할을 하게 된다.

![00](/assets/images/posts/20200112TorAsASOCKSProxy/00.png)

## SOCKS Proxy
SOCKS는 서버와 클라이언트 사이의 TCP/UDP 통신을 프록시 서버를 거쳐 진행하도록 해주는 프로토콜이다. 프록시 서버는 클라이언트의 IP를 숨겨주는 기능을 한다. HTTP Proxy는 HTTP 프로토콜을 지원하는 반면 SOCKS Proxy는 더 낮은 수준에서 Proxy 역할을 한다. SOCKS 4/4a/5 버전이 있는데 앞으로 진행할 내용에선 SOCKS 5를 이용한다.

## Tor as a SOCKS5 Proxy
일반적으로 Tor는 9050번 포트로 SOCKS 연결을 지원한다.(참고로 Tor 브라우저는 9150번 포트로 지원한다) 클라이언트 PC에서 프로그램은 SOCKS5 설정을 통해 Tor의 SOCKS 로 접속할 수 있다. 간단히 단점을 언급하고 지나가자면 Tor 자체가 좀 느리다. 여러 노드를 지나가다보니 느릴 수 밖에 없다.

## Install Tor (for Mac)
Tor는 총 3가지 방법으로 설치할 수 있다. 그 중 Tor 브라우저를 다운로드 받는 것으로 쉽게 설치할 수 있지만, 여기서는 Mac의 brew 라는 패키지 관리자를 통해 Tor 를 설치해볼 것이다. (이렇게 설치하는게 깔끔하고, 원하는 브라우저를 사용할 수 있다는 장점이 있다) Tor를 설치하는 명령어는 아래와 같다.

```console
Mac:~ user$ brew install tor
```

```console
Mac:~ user$ brew install tor
Mac:~ user$ brew services start tor
tor     started spidey /Users/spidey/Library/LaunchAgents/homebrew.mxcl.tor.plist
```
위와 같이 나오면 Tor 네트워크를 이용할 준비가 된 것이다.

## Configure Firefox Browser
Firefox 브라우저를 이용해 Tor 네트워크를 이용해볼 것이다. 아래는 설정 화면이다.

![01](/assets/images/posts/20200112TorAsASOCKSProxy/01.png)

HTTP 또는 HTTPS 통신은 SOCKS 호스트를 목적지로 먼저 HTTP 프록시 (127.0.0.1:8080)로 요청을 보내게 된다.

## Configure Burp Suite
HTTP 프록시로 보내진 요청을 확인하기 위해 Burp Suite을 사용한다. 아래는 Burp suite의 설정 화면이다.
![02](/assets/images/posts/20200112TorAsASOCKSProxy/02.png)
![03](/assets/images/posts/20200112TorAsASOCKSProxy/03.png)

끝이다. 이제 Firefox 브라우저를 이용해 접속한 웹사이트는 Burp Suite을 통해 Tor 네트워크를 이용하게 된다.
Tor를 이용한 IP 차단 우회는 Tor를 건전한 목적으로 유용하게 사용할 수 있는 방법 중 하나다. 속도가 느리다는 단점이 있지만 유료 VPN을 사용할 수 없는 상황이라면 충분히 이용할만한 가치가 있다고 생각한다. 주의할 점은 Tor도 완벽하지 않다는 점이다. 하지만 Tor project 의 준수사항을 잘 지킨다면 안전하게 이용할 수 있다.

><b id="footnote1">1</b> 차단 당한 Exit Node는 Middle Node로 위치를 바꾸어 지속적으로 Tor 네트워크에서 역할을 하게된다.