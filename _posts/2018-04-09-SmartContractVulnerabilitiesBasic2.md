---
title: "Smart Contract Vulnerabilities Basic 2"
date: 2018-04-09 00:00:00 +0900
---

## 5. Forcing ether to a contract
솔리디티(Solidity)의 selfdestruct 함수는 두 가지 작업을 한다. 컨트랙트(Contract)에 해당하는 주소의 바이트 코드를 효과적으로 삭제하여 사용 불가능하게 만들고, 컨트랙트의 모든 자금은 지정된 주소로 보내준다. 즉, 블록체인에서 스마트 컨트랙트 코드를 제거하는 함수다. 하지만 특별한 경우가 있다. 지정된 주소가 컨트랙트라면, 해당 컨트랙트의 fallback함수를 실행하지 않는다.

### Fallback Function
솔리디티에서 컨트랙트는 인자(arguments), 리턴(return) 그리고 이름이 없는 함수를 하나 가지고 있다. 이 함수는 어떤 데이터나 함수 호출 없이 컨트랙트를 호출했을 때 실행된다. 그리고 이 함수가 payable 인 경우 컨트랙트는 이더(Ether)를 받을 수 있고, 컨트랙트에 이더를 보낼 경우에 이 함수가 실행된다. 이 함수를 Fallback Function(함수)이라고 부른다.

![00](/assets/images/posts/20180409SmartContractVulnerabilitiesBasic2/00.png)

컨트랙트의 함수가 해당 컨트랙트의 잔액이 일정 금액 이상인지 이하인지 확인하는 조건문이 있다면, 그 조건문은 잠재적으로 우회될 수 있다.
오른쪽 코드를 보자. Fallback 함수에 있는 revert<sup id="a1">[1](#footnote1)</sup>함수로 인해, 이 컨트랙트는 이더(Ether)를 수신할 수 없다. 하지만, 이 컨트랙트를 대상으로 selfdestruct 한다면, fallback 함수는 동작하지 않는다. 결과적으로, 잔액은 0보다 큰 상태가 될 수 있고, 공격자는 onlyNonZeroBalance 함수의 require 문을 우회할 수 있게 된다.

### Mitigation
컨트랙트의 잔고를 확인하는 것을 보호 조건으로 사용하면 안된다.

## 6. Call to the Unknown, Dos with Unexpected revert
이 취약점은 King of the Ether 스마트 컨트랙트에서 발견되었다.
공격자는 안전하지 않은 컨트랙트에 충분한 양의 이더(Ether)를 보내 자신이 리더임을 주장할 수 있다. 그리고나서, 리더임을 주장하려고 하는 다른 사람의 트랜젝션은 공격자의 컨트랙트에 있는 revert 함수에 의해 취소될 수 있다. 

단순한 공격이긴 하지만, 이 때문에 컨트랙트엔 영구적인 서비스 거부(Denial of Service)가 발생하여 쓸모없게 된다. 폰지 사기 컨트랙트(ponzi scheme contracts) 에서 동일한 패턴을 찾아볼 수 있다.

![01](/assets/images/posts/20180409SmartContractVulnerabilitiesBasic2/01.png)

## 7. Short Address Attack
이 공격은 Golem팀에 의해 발견되었고 다음 기사에 잘 소개되어 있다. 이 취약점으로 공격자는 transfer 기능을 악용하여 허용하는 것보다 많은 양의 ERC20 토큰을 출금할 수 있다.

이 버그를 설명하기 위해 먼저 10000개의 토큰이 들어있는 지갑을 가지고 있는 거래소와 이 거래소의 지갑에 32개의 토큰을 가지고 있는 사용자가 있다고 생각해보자. 그리고 사용자의 주소는 0x12345600(주소의 끝자리가 0으로 끝난다.)이라고 가정해보자. 

사용자(이하 공격자)는 잔고에 있는 토큰의 양보다 더 많은 양의 토큰을 출금하려고 한다. 공격자는 거래소에 가서 토큰 출금 버튼을 누른다. 그리고 공격자의 주소에서 0을 제외한 값을 입력한다. (공격자의 주소 길이가 잘못되었음에도 불구하고 거래소는 입력값 검증을 하지 않고 거래를 진행시킨다.)

그런 다음, EVM은 함수의 서명과 인자로 실행되는 트랜젝션의 입력값을 계산한다.
ERC20의 transfer 함수는 transfer(address to, uint256 amount)로 다음과 같이 실행된다.

![02](/assets/images/posts/20180409SmartContractVulnerabilitiesBasic2/02.png)

### The vulnerability
자세히 살펴보면, 트랜젝션의 길이가 2바이트 더 짧다. 이런 경우 EVM은 트랜젝션의 마지막에 0을 채워서 다음과 같이 실행한다.

![03](/assets/images/posts/20180409SmartContractVulnerabilitiesBasic2/03.png)

이렇게 하면 거래소에 32개의 토큰만 가지고 있는 공격자는 훨씬 더 많은 양의 토큰(8192개)을 완벽하게 정상적인 트랜젝션으로 만들어 실행할 수 있다. 이 공격은 보내는 사람의 계좌(거래소의 지갑 계좌)에 충분한 양의 토큰이 있다는 사실에 기초한다.

### Mitigation
* msg.data의 크기가 잘못되었다면 예외처리
* 거래소는 반드시 입력값 검증한다

## PLUS, CVE Details
CVE Details에서 이더리움 관련 취약점을 검색해보면 다음과 같이 나온다.

![04](/assets/images/posts/20180409SmartContractVulnerabilitiesBasic2/04.png)

모두 2018년 1월 CVE로 등록되었고, 4월 1일 기준으로 일곱개의 권한 우회(Authentication Bypass)와 두개의 DoS 취약점이 있다. 앞서 설명한 취약점들과 조금 다른 내용이지만, 요약하자면 cpp-ethereum의 JSON-RPC를 지원하는 몇가지 API에 공격 가능한 권한 관련 취약점이 있었다.

스마트 컨트랙트가 없거나 의미를 갖기 힘든 컨소티움(Consortium) 블록체인이나 프라이빗(Private) 블록체인(i.e. NexLedger) 모의해킹 시 JSON-RPC는 유심히 살펴봐야 할 부분이 될 것으로 생각된다. 잘 기억해두자.

><b id="footnote1">1</b> 공컨트랙트의 모든 상태 변화를 되돌린다. 그리고 함수를 호출한 사람(Caller)에게 이더(Ether)를 되돌려 준다.