---
title: "Smart Contract Vulnerabilities Basic 1"
date: 2018-04-02 00:00:00 +0900
---
블록체인 기술을 이용한 시스템이 활발하게 만들어지고 있다. 자연스럽게 스마트 컨트랙트(Smart Contract)의 수요가 늘어나고 있고, 현재까지는 이더리움(Ethereum)을 이용하는 경우가 가장 많아보인다. 넥스레저(NexLedger)도 이더리움을 기반으로 하고 있다. 넥스레저 혹은 다른 이름의 블록체인 취약점 점검을 위해 이더리움 스마트 컨트랙트의 기본적인 취약점과 대응방법을 알아보자.

스마트 컨트랙트를 작성할 때 가장 많이 사용하는 솔리디티의 대표적인 취약점 7가지와 대응 방법에 대해 알아보자.

## 1. Overflow and Underflow
오버플로우(Overflow)는 사전적인 의미로 넘쳐 흐른다는 뜻을 가진다. 오버플로우의 종류엔 여러가지가 있는데, 이해를 돕기 위해 정수 오버플로우(Integer Overflow)로 예를 들자면, 정수형 변수가 담을 수 있는 최대값을 넘어 증가할 때 발생한다. 솔리디티는 256비트 숫자(2<sup>256</sup>-1)까지 다루고 있고, 최대값보다 1 증가한 값은 0이 된다.

언더플로우(Underflow)는 반대로 unsigned(자연수 범위) 변수의 값이 0보다 작아질 때 발생하며, 0보다 작아진 변수의 값은 변수의 최대값을 갖게 된다. 두 가지 모두 위험하지만 언더플로우가 일어날 가능성이 좀 더 크다. 

![00](/assets/images/posts/20180402SmartContractVulnerabilitiesBasic1/00.png)

예를 들어 토큰(Token) 10개를 가지고 있는 사용자 A는 악의적으로 11개의 토큰을 사용하려고 할 수 있다. 만약 코드에서 이 취약점을 확인하지 않는다면, 공격자는 자신이 가지고 있는 토큰(10개)보다 더 많은 토큰을 소비<sup id="a1">[1](#footnote1)</sup>할 수 있다. 그리고 공격자의 토큰은 가질 수 있는 최대한의 개수(2256-1)만큼 늘어난다.

### Mitigation
직접 입력값 검증(Validation)을 해주면 된다. 하지만 요즘엔 OpenZeppelin의  SafeMath를 많이 사용한다.

## 2. Visibility
함수의 가시성<sup id="a2">[2](#footnote2)</sup>(Visibility)에 대해서 모르는 사람은 없을 것이다. 솔리디티에도 동일한 개념이 있다. 취약점과 관련이 있는 중요한 접근 제어자를 먼저 알아보자. Public 함수는 컨트랙트 내부의 함수, 상속된(inherited) 컨트랙트의 함수 또는 외부 사용자 등 누구나 호출할 수 있다. External 함수는 외부에서만 호출할 수 있다. 컨트랙트 내부의 다른 함수들은 이 함수를 호출할 수 없다. 
2017년 7월 Visibility 취약점으로 인한 Parity Wallet 해킹으로 3천만 달러 손실

오른쪽의 코드를 보면 cannotBeCalled 함수는 External 함수고, canBeCalled함수는 Public 함수다. cannotBeCalled함수는 외부 가시성(External Visibility)때문에 컨트랙트 내 다른 함수(testExternal)에 의해 호출될 수 없다. 따라서 컴파일되지 않는다. 하지만 cannotBeCalled 함수는 다른 컨트랙트에 의해 호출될 수 있다.

![01](/assets/images/posts/20180402SmartContractVulnerabilitiesBasic1/01.png)

### Mitigation
용도에 맞추어 Public 대신 Private 함수와 Internal 함수를 사용해야 한다. Private 함수는  오직 컨트랙트 내부에서만 호출할 수 있다. Internal 함수는 조금 덜 제한적인데, 부모로부터 상속된 컨트랙트는 이 함수를 사용할 수 있다. 외부 컨트랙트에서 호출할 일이 없다면, Private 또는 Internal 함수를 사용하는 것이 좋다.

## 3. Delegatecall
솔리디티는 메시지 호출<sup id="a3">[3](#footnote3)</sup>(Message Call)을 통해 컨트랙트를 제어한다. Delegatecall는 그 중 하나로, 다른점은 함수를 호출하고 있는 컨트랙트의 컨텍스트(Context)에서 대상 주소의 코드가 실행된다는 점과 msg.sender와 msg.value의 값이 바뀌지 않는다는 점이다. 컨트랙트가 런타임(runtime)에 다른 주소의 코드를 동적으로 불러올 수 있음을 의미한다. 스토리지, 현재 주소 그리고 잔액은 함수를 호출하는 컨트랙트에서 가져오지만, 코드는 호출되고 있는 주소에서 가져온다.
코드 인젝션(Code Injection)과 비슷한 개념이라 생각하면 조금 쉬울 수 있다.
더 나은 이해를 위해 직접 솔리디티로 스마트 컨트랙트를 구현해보는 것이 가장 좋다.

DelegateCall 함수는 라이브러리를 구현하고 코드를 모듈화(modularization)하기 위한 것이기 때문에 매우 유용하다. 하지만 근본적으로, 누구나 원하는 코드를 컨트랙트에서 실행시킬 수 있기때문에 문제가 될 수 있다.
다음 코드를 보자. 공격자는 Delegate 컨트랙트의 함수 pwn을 호출할 수 있는데, 이 컨트랙트는 Delegation 컨트랙트에 속해 있기 때문에 이 컨트랙트의 소유권을 주장할 수 있다.
Parity wallet 해킹은 안전하지 않은 visibility와 비정상적인 데이터에 대한 delegatecall 이 모두 포함되어 있었다. Delegatecall로 구현된 취약한 컨트랙트의 함수와 소유권을 수정할 수 있는 다른 컨트랙트가 공개되어 있었다. 공격자는 msg.data 필드를 작성해 취약한 함수를 호출할 수 있었다.
msg.data 필드에 포함될 내용은 호출하고자 하는 함수의 서명이다. 여기서 서명은 함수 프로토타입의 SHA3 해시 값의 처음 8 바이트를 의미한다.

![02](/assets/images/posts/20180402SmartContractVulnerabilitiesBasic1/02.png)

## 4. Re-entrancy, DAO 해킹
솔리디티의 call 함수는 value와 함께 호출될 때, _amount 만큼 msg.sender에게 이더(Ether)와 가스<sup id="a4">[4](#footnote4)</sup>(gas)를 전달한다. 다음 코드 스니펫(snippet)에선, sender의 잔액을 실제로 줄이기 위해 함수가 호출된다.

![03](/assets/images/posts/20180402SmartContractVulnerabilitiesBasic1/03.png)

간단히 말하자면, 내가 내 은행 계좌에 있는 돈의 출금을 요청한 상태에서, 은행원이 내 계좌의 잔고를 줄이기 전에 다시 출금 요청을 한다. 은행원은 내 계좌의 잔고를 줄이지 못한 채 계속해서 돈을 출금해준다. 이 과정을 반복해 계속해서 돈을 받는 것이다.
“500원을 출금할 수 있을까요? 잠시만요, 그 전에.. 500원을 출금할 수 있을까요?”
계좌의 잔고는 바닥나지 않기 때문에 출금 요청을 할 에너지(Gas)가 없을 때까지 계속해서 500원을 반복해서 출금할 수 있다. 시작 단계에서 단 한번 계좌 잔고에 500원 이상의 잔액이 남아 있는지 확인하도록 설계되어 있었기 때문에 이 공격이 가능했다.

### Mitigation
돈이 전달되기 전에 sender의 계좌를 줄이면 된다. 병렬 프로그래밍을 다루는 사람들에게 익숙한 또 다른 해결방법은 뮤텍스<sup id="a5">[5](#footnote5)</sup>(mutex)를 사용하는 것이다. 모든 종류의 레이스 컨디션(race condition) 문제를 해결할 수 있다.
현재, require(msg.sender.transfer(_value)를 사용하는 것이 이런 종류의 상황을 해결하는 가장 좋은 방법이다.

><b id="footnote1">1</b> 공격자는 언더플로우(Underflow) 취약점으로 토큰의 개수가 0개인 상태에서 소비할 경우, 토큰이 2256-1개로 늘어나는 것을 알고있다.

><b id="footnote2">2</b> 클래스와 클래스 맴버(변수 또는 함수)의 사용범위를 결정. 접근 제어자.

><b id="footnote3">3</b> 컨트랙트가 다른 컨트랙트를 호출하거나 EOA로 이더(Ether)를 보낼 때 사용.

><b id="footnote4">4</b> 트랜젝션을 실행하기 위한 수수료 단위. EVM에서 Bytecode를 실행할 때 가스(Gas)를 지불해야 코드를 실행할 수 있음.

><b id="footnote5">5</b> 공유된 자원을 여러 스레드(Thread)가 동시에 접근하는 것을 막음. 여러 풀 노드가 동시에 스마트 컨트랙트 바이트 코드(Bytecode)를 실행하는 것을 막을 수 있다.