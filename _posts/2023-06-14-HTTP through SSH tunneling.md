---
title: "SSH 터널링을 통한 HTTP 통신"
author: wrkholic84
date: 2023-06-14 00:00:00 +0900
categories: [Security, web]
tags: [SSH, Tunneling, web]
math: true
mermaid: true
---
[참조 링크](https://blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=221364560794&parentCategoryNo=&categoryNo=22&viewDate=&isShowPopularPosts=false&from=postView)를 그대로 가져옴.   
자세하고 쉽게 설명되어 있어서, 직접 작성하는 것보다 아카이빙해두고 편집하는것이 더 의미가 있는 것 같음. 

## SSH 포트 포워딩이란?

많은 개발자들이 SSH를 단순히 원격 터미널 접속 용도로만 사용한다. 그렇지만 SSH는 기본적인 접속 기능 외에도 'SSH 포트 포워딩' 또는 'SSH 터널링 기능' 이라고 불리는 것을 제공한다 (두 용어는 같은 것을 의미한다). 터널링이라는 단어는 보통 VXLAN의 Overlay Network에서 물리적 토폴로지를 논리적으로 나눌 때에나 등장하는 단어일 터인데, 신기하게도 SSH 에서도 그와 비슷한 기능을 사용할 수 있다. 

단적으로 말하자면, SSH 터널링은 '프록시' 와 비슷한 역할을 하며, SSH의 특징 상 SSH 터널링을 통해 전달되는 데이터는 모두 암호화된다. 그렇다면 SSH 터널링은 무엇을 위한 프록시인가, 를 짚고 넘어가야 할 것 같다

![00](/assets/images/posts/20230614HTTPthroughSSHtunneling/00.jpg)

SSH 포트 포워딩을 사용할 수 있는 단적인 예를 들어 보자. 나는 A 서버에서 80 포트로 바인딩 된 서비스에 접근할 필요가 있다. 그러나 보안 상의 이유로 A 서버의 Firewall에서 SSH (22 포트) 외의 포트는 전부 차단된 상태이다. 당연히 Firewall은 80 포트를 개방하지 않았기 때문에 해당 서비스로 접근이 불가능하다. 

이 때, SSH 포트포워딩을 사용하면 SSH 터널링을 통해 웹 서버에 접근할 수 있다. 80 포트를 사용하는 서비스와 SSH 서버를 엮은 뒤 SSH 터널링을 생성하고, 사용자가 서비스를 요청하면 해당 요청은 SSH 서버로 전송된 뒤 A 서버 내부에서 다시 포워딩된다.

![01](/assets/images/posts/20230614HTTPthroughSSHtunneling/01.png)

일단 사용자와 서버 간의 SSH 터널링이 수립되고 나면 데이터의 요청 및 반환은 모두 SSH 서버를 통해 일어나므로 서버의 80 포트로 접근할 필요가 없다. 따라서 SSH 서버는 터널링을 통해 데이터를 주고받을 수 있게 해주는 일종의 프록시의 역할을 수행하게 된다. 

.... 설명은 이렇게 했지만 SSH 터널이라는 개념 자체는 원래 SSH 연결에서 사용되는 것이며, SSH 클라이언트와 SSH 서버 사이의 연결 통로 자체를 SSH 터널이라고 부른다. 즉, SSH 연결 수립 뒤 외부로부터 데이터를 보호할 수 있는 일종의 연결 통로를 터널이라고 부른다고 한다. SSH 터널링은 Secure Shell 상의 데이터뿐만 아니라 기타 애플리케이션의 데이터 또한 SSH 터널을 이용하게 함으로써 암호화 등과 같은 SSH의 장점을 활용할 수 있는 방법이라고 보면 된다. [4]

그러나 잘만 악용하면 나쁜 의도로 사용할 수도 있기 때문에 공격자의 관점에서 SSH 포트 포워딩을 바라볼 필요도 있을 것이다. 보안 측면에서도 SSH 터널링을 생각해 볼 필요가 있다는 이야기이다.


##  SSH 포트 포워딩 사용하기

SSH 포트 포워딩에는 크게 2가지 종류가 있다. Local, Remote 모드를 사용할 수 있고, Dynamic이라는 것도 있는 듯 하지만 지금은 다루지 않기로 했다.

### 서버 구성

당연하겠지만, SSH를 기반으로 하기 때문에 SSH Client와 SSH Server로 사용할 서버 각각 1대가 필요하다. VM을 사용해 아래와 같이 구성하였다. 

![02](/assets/images/posts/20230614HTTPthroughSSHtunneling/02.png)

테스트를 위해 SSH 서버에서는 Nginx 웹 서버를 80 포트와 바인딩해 컨테이너로 생성해 놓았다. SSH 포트 포워딩을 통해 SSH Client에서 해당 Nginx 웹 서버로 접근할 것이다.


### Local 포트 포워딩

SSH 포트 포워딩은 연결을 수립하는 주체가 누구냐에 따라 Local과 Remote로 구분할 수 있다. Local은 가장 이해하기 쉽고 직관적인 경우로, 늘 하던 것처럼 SSH Client -> SSH Server로 연결을 수립하는 경우이다. 이해를 위해 간단한 예시를 통해 Local 포트 포워딩을 사용해보자.

![03](/assets/images/posts/20230614HTTPthroughSSHtunneling/03.png)

SSH Client는 SSH Server에 SSH로 접속할 수 있다. 그러나 Nginx 서버는 127.0.0.1:80으로 바인딩되어 있어 외부에서 접근할 수 없는 상황이다. SSH Client 에서 Nginx에 접근할 수 있도록 SSH 터널링 연결을 생성한다.

![04](/assets/images/posts/20230614HTTPthroughSSHtunneling/04.png)

가장 많이 헷갈리는 것이 SSH 포트 포워딩 시 '무엇을 어떻게 입력할지' 인데, 위 그림으로 이해할 수 있다. SSH Client에서 ssh -L [로컬에서 사용할 포트]:[최종적으로 접근할 곳] [SSH Server 주소] 형식으로 입력한다. SSH 터널이 생성된 뒤에는 [로컬에서 사용할 포트] 를 이용해 [최종적으로 접근할 곳] 에 접근할 수 있다.

직접 해보자. 가장 먼저, SSH Server 에서는 Nginx 서버시 127.0.0.1:80 으로 바인딩되어 실행되고 있다.

```bash
[root@ssh-server ~] docker ps --format '{% raw %}{{.Names}}{% endraw %}\t{% raw %}{{.Image}}{% endraw %}\t{% raw %}{{.Ports}}{% endraw %}'
vigilant_khorana        nginx   127.0.0.1:80->80/tcp
```

SSH Client에서 아래의 명령어를 입력해 SSH 포트 포워딩을 실행한다. SSH Server의 주소는 192.168.1.201 이고, 해당 SSH Server에서 접근할 Endpoint는 127.0.0.1:80 (Nginx Server) 이며, SSH Client의 8585 포트로 해당 Endpoint에 접근할 것이다.

```bash
[root@ssh-client ~] ssh -L 8585:127.0.0.1:80 192.168.1.201
root@192.168.1.201's password:

Last login: Sun Sep 23 05:01:06 2018
[root@ssh-server ~]
```

평상시와 똑같이 SSH 접속에 성공하였지만, SSH 터널이 생성된 상태이다. SSH Client에서 새로운 터미널을 생성한 뒤, 로컬호스트의 8585로 요청을 전송해보자.

```bash
[root@ssh-client ~] curl localhost:8585 -v
...
< HTTP/1.1 200 OK
< Server: nginx/1.15.3
< Date: Sun, 23 Sep 2018 09:06:43 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 28 Aug 2018 13:32:13 GMT
< Connection: keep-alive
< ETag: "5b854edd-264"
< Accept-Ranges: bytes
```

요청이 제대로 전송되었고 응답 또한 수신하였다. SSH 터널링은 수립된 SSH 연결을 통해 구성되기 때문에 위에서 접속한 SSH 연결을 끊으면 SSH 터널 또한 종료된다.


### Remote 포트 포워딩

Remote 포트 포워딩은 SSH Server -> SSH Client 로 연결을 수립해 SSH 터널을 생성한다. 연결을 생성하는 주체가 SSH Server이기 때문에 SSH Server의 22 포트를 Firewall에서 개방해 둘 필요가 없다. 이는 역으로 말하자면 22 포트를 포함해 모든 포트가 Firewall에 의해 막혀 있는 상태이더라도, SSH 터널을 생성해 SSH Server에서 실행 중인 서비스, 또는 SSH Server가 접근 가능한 네트워크에 접속할 수 있다는 의미가 된다.

간단한 예시를 들어보자. 외부로 나가는 트래픽은 허용되지만 내부로 들어오는 트래픽은 Firewall에 의해 전부 차단되는 상황을 가정한다. 물론 SSH 또한 사용할 수 없는 상황이다.

![05](/assets/images/posts/20230614HTTPthroughSSHtunneling/05.png)

이 때 Remote 포트 포워딩을 SSH Server에서 사용한다면 내부 네트워크에 쉽게 접속할 수 있다. Outbound 트래픽은 허용되는 상황이므로 SSH Server -> SSH Client로 SSH 터널(SSH 연결)을 생성한 뒤, SSH Client는 SSH Server가 접근 가능한 네트워크에 접속해 데이터를 주고 받는 방식이다.

어찌 보면 보안상으로 취약하다고 말할 수 있다. Firewall 단에서 모든 포트로 들어오는 Inbound 트래픽을 차단해도 Outbound 트래픽을 통해 내부 네트워크에 자유롭게 접근할 수 있기 때문이다. 따라서 SSH 연결이라고 해서 무조건 Trust한 패킷만 오고 간다고 볼 수는 없다. 

어쨌든, 사용해보는 것이 가장 빠르게 이해할 수 있는 지름길이다.

![06](/assets/images/posts/20230614HTTPthroughSSHtunneling/06.png)

이번에는 SSH Server 에서 ssh -R [SSH Client가 사용할 포트]:[최종 목적지] [SSH Client 주소] 와 같은 형식으로 SSH 포트 포워딩을 실행한다. 당연하지만, SSH Client에서도 SSH 데몬이 실행 중이여야만 한다. SSH Server에서 SSH Client로 SSH 연결을 수립하기 때문이다.

```bash
[root@ssh-server ~] ssh -R 8585:127.0.0.1:80 192.168.1.200
root@192.168.1.200's password:

Last login: Sun Sep 23 05:30:37 2018 from ssh-client
[root@ssh-client ~]#
```

SSH Client 측에서 로컬 호스트의 8585로 접근하면 127.0.0.1:80에 접근할 수 있다.

```bash
[root@ssh-client ~] curl localhost:8585 -v
....
> User-Agent: curl/7.29.0
> Host: localhost:8585
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.15.3
< Date: Sun, 23 Sep 2018 09:40:40 GMT
....
```