---
title: "How to smell out in-house coin miners"
date: 2018-03-12 00:00:00 +0900
---
대다수 일본인들은 회사에서 휴대폰을 충전하는 행위를 ‘전기를 훔치는 일’이라고 생각하여 이를 엄격히 제재한다고 한다. 다만, 일본의 경우 한국과 달리 업무용 스마트폰을 지급하는 것이 보편적이다. 암호화폐를 생각해보자. 직원이 회사의 자원을 이용하여 암호화폐 채굴(Mining)을 한다면 이는 용인될 수 있을까? 인정 문화를 공유하는 한국에서 휴대폰 충전 정도는 너그러이 넘어갈 수 있지만, 상당한 양의 전력을 필요로 하는 채굴은 이야기가 다를 것이다.

최근 암호화폐 채굴 악성코드가 유포되어 큰 문제가 되었다. 이 문제는 직원이 직접 채굴을 하는 문제보다 더 심각한 문제다. 많은 수의 서버와 PC를 감염시킬 수 있고, 상당한 양의 전기를 사용할 뿐만 아니라 회사에서 운영하고 있는 서비스에 영향을 미칠 수 있다. 이는 곧 회사의 이익(Profit)에 적지 않은 문제를 가져온다.

> 사내 PC 성능은 대부분 문서 작업에 적합한 수준이기 때문에 사내 암호화폐 채굴은 CPU Mining 으로 진행된다. GPU 또는 ASIC을 이용한 채굴에 비해 효율이 무척 낮은 편이지만, 노느니 염불 외는게 낫지 않은가.

## CPU Mining
회사에서 암호화폐 채굴을 하는 경우 대부분 CPU를 이용한 채굴 방식을 사용한다. 고성능의 GPU나 ASIC 장비를 이용한 채굴이 일반적이지만 직원 PC나 서버에 고성능의 GPU가 없기 때문이다. CPU를 이용해 채굴할 수 있는 코인은 디지바이트(Digibyte-Skein, DGB), 모네로(XMR), ZCash(ZEC), ZCoin(XZC) 등이 있다.

## Stratum Mining Protocol
채굴 프로그램 사용법을 보면 stratum+tcp 라는 스킴(scheme)을 사용하는 것을 알 수 있다.
cpuminer 프로그램을 이용한 Zcoin 채굴 명령

```console
cpuminer.exe -a lyra2z -o stratum+tcp://miningpooladdress.com:1234 -u username.workername -p x
```

stratum은 일반적인 TCP 소켓에 JSON-RPC 메시지로 인코딩된 페이로드를 담아 사용하는 채굴 프로토콜이다. 암호화폐의 종류와 상관없이 사용할 수 있다.

![00](/assets/images/posts/20180312HowToSmellOutInHouseCoinMiners/00.png)

JSON-RPC 메시지에 담기는 내용은 다음과 같다.
* method: 클라이언트와 서버 사이에 호출되는 함수 명
* params: 호출되는 함수에서 사용되는 파라미터 

통신하는 내용의 대부분은 “method”와 “params"를 확인하는 것으로 분석할 수 있다.

## Communication
실제로 통신하는 내용을 살펴보면 다음과 같다.
![01](/assets/images/posts/20180312HowToSmellOutInHouseCoinMiners/01.png)

## Mitigation
### 패킷 탐지 규칙(Rule) 설정
채굴 프로그램은 서버(채굴 풀)와 통신할 때 stratum+tcp 라는 특이한 프로토콜을 사용하고 있다. 앞서 알아본 바와 같이, mining.subscribe 라는 함수를 사용하여 서버(채굴 풀)와 통신을 시작한다.

IDS 또는 IPS의 탐지 규칙(Rule)에 mining.subscribe, mining.authorize 그리고 mining.notify 등을 추가한다면 회사 내에서 채굴하는 행위를 막을 수 있다. 
![02](/assets/images/posts/20180312HowToSmellOutInHouseCoinMiners/02.png)
악성코드에 의해 사용자(또는 직원) 모르게 PC나 서버에서 채굴 프로그램이 동작하는 것도 차단할 수 있다. 
> 서버와 연결되지 못한 채굴 프로그램은 CPU를 이용한 채굴을 시작하지 않는다. - CPU 점유율을 높이지 않는다.

### 포트 그리고 서버(채굴 풀) 차단
서버(채굴 풀)는 채굴을 위해 다양한 포트를 지원한다. 한국에서 가장 유명한 서버(채굴 풀)인 마이닝풀허브(https://miningpoolhub.com)는 12xxx, 17xxx 그리고 20xxx번 포트를 사용한다. 회사 정책에 의해 80 또는 443번 포트를 제외한 아웃바운드(outbound) 트래픽은 차단되기 때문에 이 서버를 이용해 채굴할 수 없다.

채굴 악성코드는 이를 우회하여 80 또는 443번 포트를 이용한 채굴을 시도한다. 크립토 풀(https://monero.crypto-pool.fr)은 다양한 채굴 포트를 지원하고 있고, 모네로(XMR) 채굴 풀 중 가장 유명한 서버다.

![03](/assets/images/posts/20180312HowToSmellOutInHouseCoinMiners/03.png)

크립토 풀(crypto-pool)과 같은 서버(채굴 풀)에 대한 통신을 차단하여 사내 채굴 행위를 막을 수 있다. 물론, 악성코드에 의해 동작하는 채굴 프로그램도 차단할 수 있다.

마지막으로 SSL 프로토콜을 이용해 채굴하는 경우도 있는데, 마찬가지로 채굴을 위해 사용하는 프로토콜인 stratum+tcp를 사용할 수 밖에 없기 때문에 대응 방법은 동일하다.


> 참고 : 전세계 채굴 풀 서버에 대한 실시간 정보 알림 사이트 [https://investoon.com/](https://investoon.com/)
