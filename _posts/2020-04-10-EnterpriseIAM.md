---
title: "Enterprise IAM"
date: 2020-04-10 00:00:00 +0900
---

Identity and Access Management는 권한이 있는 사용자가 필요한 자원을 원하는 시기에 접근할 수 있게 하는 보안 원칙으로, 쉽게 생각되겠지만, 신경써야할 부분이 많다. 사용자가 누구인지 확실히 해야할 필요가 있고, 사용자의 접속 과정이 불편하진 않은지 생각해야하만 한다. 모든 보안 장치들을 통과해야만 권한을 얻을 수 있도록 만드는게 능사가 아니기 때문이다.

## Enterprise Identity and Access Management
일반적인 기업의 IAM 솔루션의 구성 요소를 살펴보자.
![00](/assets/images/posts/20200410EnterpriseIAM/00.png)
### AD/LDAP
사용자 정보 저장소가 필요하다. Active Directory를 종종 사용한다.(꼭 AD일 필요는 없다)
### Single Sign-On
꽤 일찍부터 기업들은 Single Sign-On(SSO)에 투자를 해왔다. 처음엔 회사 내에서 사용할 목적으로 비표준 방식의 SSO를 구현했다. 
### Identity Federation (Federated SSO)
외부 파트너와 같이 일하고, SaaS와 같은 서비스를 시작하면서 SSO와 같은 통합 계정이 필요해졌고, 표준 통합 계정으로 여러 공급 업체의 제품을 통합할 수 있다.
### Multi-factor Authentication
다단계 인증을 사용하면 사용자에 대한 더 높은 신뢰도를 가질 수 있기 때문에, 다단계(Multi-factor) 인증이 필요하다. 
### Automated Provisioning
SaaS의 사용자가 증가함에 따라, 많은 회사들이 임직원 관리 시스템에 사용자를 연결하기 시작했다. HRMS 기반의 사용자 관리, 요청/승인 과정, SaaS 또는 내부 시스템 할당(Provisioning) 등의 내용이 포함된다.
### Compliance - Audit, Reporting Analytics
IAM은 모니터링하고 감시하는 기능이 필요하다. 역할 분리, 자격 증명등의 내용이 포함된다.

## Authentication and Authorization
**Authentication(AuthN)** 은 사용자가 누구인지 증명하는 행위다. 일반적으로 username과 password를 사용하거나, 더 높은 신뢰도를 위해 인증서를 사용한다. MFA 같은 것을 사용할 수도 있다.
**Authorization(AuthZ)** 은 사용자가 접근할 수 있는 대상을 결정한다. 예를 들어, 로그인한 후에 공유 파일을 열고 내용을 확인할 때, 그 파일에 대한 권한이 사용자의 Authorization이다.

## Basic Local Identity and Access Management
개발자는 애플리케이션 개발 요청을 받으면 요구 사항 분석을 시작하고 비지니스 로직을 만들기 시작한다.
사용자별 설정, 환경설정 그리고 접근 권한 등을 관리하기 위해 서로 다른 사용자들을 나누기 위한 방법이 필요해진다. 그래서 개발자는 애플리케이션마다 사용자 저장소를 만들고 인증 로직을 만들어야 한다. 물론 코드는 재사용될 것이다.
![01](/assets/images/posts/20200410EnterpriseIAM/01.png)
무슨 일이 발생했을까? 애플리케이션들은 각각 변경될 것이고 상황은 매우 안좋아질 것이다. 비지니스 로직을 관리하는 개발자는 이제 사용자 저장소를 보호하는데 신경 써야할 것이다. 인증 과정에 문제가 생기는지, 해킹당하지 않는지 보장하는 일은 개발자에겐 매우 고통스러운 일이다.
사용자 입장에서도 반드시 아이디와 비밀번호를 입력해야 하고, 종종 취약한 비밀번호를 사용하거나 재사용하는 문제가 발생한다. 관리자 또한 매우 고통스러운데, 먼저, 각 애플리케이션에 사용자를 생성하고, 사용자가 탈퇴하면 위와 같은 문제를 해결하기 위해 각각의 애플리케이션에서 반드시 사용자를 비활성화 해야 한다.

## Modern Identity Management and Access
현대에 쓰이는 인증 모델은 클레임 기반 모델로써, 개발자는 클레임을 받아주는 더 단순한 로직으로 애플리케이션의 인증 과정을 바꾸었다.
![02](/assets/images/posts/20200410EnterpriseIAM/02.png)
신뢰성(TRUST)은 애플리케이션과 Identity Provider(IdP) 사이에서 시작된다. IdP는 인증이나 권한을 확인해주는 서비스로 보면 쉽다. 애플리케이션은 단순히 IdP에서 전송된 클레임만 받으면 된다.
애플리케이션은 사용자 정보 설정과 적절한 권한을 위해 로컬 사용자 저장소를 가지고 있지만, 비밀번호를 관리하진 않는다. 사용자가 애플리케이션에 직접 인증을 시도하지 않기 때문이다. 대신 사용자는 IdP에서 인증을 받아야 한다.
사용자가 애플리케이션에 접속하고자 하면, 클레임이나 접속 토큰이 IdP에 의해 생성되고 애플리케이션에 보내진다. IdP는 모든 애플리케이션에 클레임을 발행할 수 있다.
클레임 기반 접속 방식은 개발자, 사용자, 관리자 모두에게 좋다. 개발자들은 강력한 인증 방법을 만들 필요가 없고 사용자 비밀번호를 보호해야할 필요가 없다. 만약 인증 방법이 변경되야 한다면 IdP에서 바꾸고 애플리케이션은 바꾸지 않고 남겨두면 된다. 사용자들은 IdP에서 한번 인증을 받으면 모든 애플리케이션에 간편하게 접속할 수 있다. 관리자들은 사용자가 퇴사할 경우 IdP에서 사용자를 비활성화하는 즉시 모든 애플리케이션에 적용할 수 있다.

## Real world example
좀 더 깊게 클레임에 대한 개념을 이해하기 위해 일반적인 상황에 맞추어 보자.
![03](/assets/images/posts/20200410EnterpriseIAM/03.png)
여행을 간다고 상상해보자. 목적지를 정하고 공항에 도착했다. 
1. 체크인 카운터(IdP)에 가서 여권을 제시하고 티켓 구입 여부를 확인한다. 
2. 체크인 직원은 여권을 확인하고 탑승권(Claim)을 발급해준다.
3. 보안체크를 통과하고 탑승할 차례다. 탑승권을 게이트(SP) 직원에게 보여준다.게이트는 체크인 카운터를 신뢰(Trust)한다.
4. 탑승권은 서명(Signed)되어 있고, 게이트는 조작여부를 알 수 있다.
탑승권(Claim)에는 이름, 비행편 등의 정보(attribute)가 들어있다. 

## Creating Trust
클레임 기반 인증 모델에선 신뢰(Trust)가 핵심이다. 제대로 동작하기 위해선 애플리케이션이 IdP를 반드시 신뢰해야 한다. 신뢰 관계를 구축하는 방법은 표준에 따라 다양한데, 인증서를 교환하는 방식이 많이 쓰인다. 신뢰 관계를 만들 때 종종 파일이나 파일에 걸린 링크와 같은 일반적인 메타데이터를 사용해야 한다. 애플리케이션은 다른 애플리케이션과 이 정보를 공유하는데, 메타 데이터 파일을 사용하면 신뢰 설정이 간단해진다.  메타 데이터는 보통 인증서를 포함하고 있고, 로그인/로그아웃 URL 그리고 시스템 사용에 필요한 값 들을 가지고 있다. 메타 데이터 파일을 사용하지 않는 경우 이런 값들을 직접 입력해줘야 한다. 어떤 경우, 인증서 대신 시스템 ID와   비밀번호를 사용하는 경우도 있다.

## Identity Management
이제 클레임 기반 인증에 대한 이해가 생겼지만 대부분의 애플리케이션에는 여전히 사용자 정보를 관리해야 한다.  오늘날 많은 회사에선 사용자를 생성하고 관리하기 위해 중앙 집중식 인적 자원 시스템을 사용하고 있다. 이 시스템은 IdP에 사용자를 보내주고, IdP는 애플리케이션에 사용자를 전달한다.
![04](/assets/images/posts/20200410EnterpriseIAM/04.png)

## Metadirectory
하지만, 대기업의 경우 더 많은 중앙 집중식 시스템이 필요하고, 다양한 사용자 저장소가 있다. 따라서 메타 디렉토리를 중앙 집중식 단일 지점으로 통합한다. 이 메타 디렉토리는 인적 자원 시스템에 의해 제공된다. 메타 디렉토리에서 사용자는 다른 시스템 중 하나와 동기화된다. 이 방법으로 사용자의 전체 수명주기는 메타 디렉토리를 기반으로 동작한다.
![05](/assets/images/posts/20200410EnterpriseIAM/05.png)

## Realm/Security Domain
클레임 기반 인증에 대해 이야기 할 때 영역(Realm) 또는 보안 도메인(Security Domain)이란 단어를 자주 들어봤을 것이다. 영역이나 보안 도메인은 기본적으로 신뢰(Trust)의 범위다. 영역(Realm) 안에서 각 항목들은 서로를 신뢰한다. 그리고 클레임은 시스템에 대한 접속을 위해 사용된다.
![06](/assets/images/posts/20200410EnterpriseIAM/06.png)

## Multiple Realms
여러 영역(Realm) 이나 보안 도메인(Security Doamin)이 있다면, 기본적으로 이들은 서로를 신뢰할 수 없다. 영역(Realm) A의 애플리케이션에 접속할 수 있도록 IdP(A) 에서 발급받은 클레임은 영역(Realm) B에서 사용할 수 없다.
![07](/assets/images/posts/20200410EnterpriseIAM/07.png)

## Federation
위와 같은 이유로 페더레이션(Federation, 통합)의 개념이 등장했다. 페더레이션을 사용하면 서로 다른 두 영역간 신뢰 관계를 설정할 수 있기 때문에 영역(Realm) B의 사용자가 영역(Realm) A의 애플리케이션에 접근할 수 있다. 
![08](/assets/images/posts/20200410EnterpriseIAM/08.png)
사용자는 자신의 로컬 IdP를 이용하여 인증한 다음 다른 영역(Realm)의 애플리케이션에 쉽게 접속할 수 있다.  사용자 관리는 각 영역(Realm) 내에서 처리된다. 즉, 영역(Realm) B의 사용자 관리자가 사용자에게 권한을 부여하지 않으면 사용자는 더 이상 접속 권한을 갖지 못한다. 영역(Realm) A의 관리자에게 알릴 필요도 없다. 사용자는 영역(Realm) B에서 한 번 권한을 잃게 되면 모든 곳에서 권한을 잃는다.

## Level of Assurance
영역(Realm)간 통합 시, 인증 수준에 대한 부분을 빼놓을 수 없는데, 다른 영역(Realm)의 프로세스, 시스템, 사용자, 관리자를 얼마나 신뢰하는지에 대한 것이다. 영역(Realm) B에서 생성된 사용자를 인증하기 위한 적절한 식별 수단을 제공해야 한다. 그리고 인증을 위해 다단계 인증이 제공된다. 내 보증(Assurance) 수준은 상당히 높지만 사용자가 단순히 익명의 양식을 채우고 계정을 만들 수 있고 오직 사용자명과 비밀번호만으로 인증한다면 내 보증(Assurance) 수준은 당연히 꽤 낮을 것이다. 많은 클레임에 사용자 인증 방법에 대한 정보가 포함되어 있다는 사실에 이를 연결시킬 수 있다. 서로 다른 두 영역간에 신뢰가 설정되어 있지만 사용자가 너무 약한 인증 방법을 사용하여 인증된 경우, 접근을 허용하지 않을 것이다. 사용자를 사용자의 영역으로 다시 돌려보내고 더 높은 수준의 인증의 메시지를 요청할 수 있다. 

## The Claim
클레임(Claim)과 관련하여 다양한 표준이 있는데, 이것은 클레임(Claim)에 다양한 내용이 포함될 수 있음을 의미한다. 하지만 최소한 고유한 사용자 식별자가 포함되어 있어야 한다. 클레임(Claim)은 서명(Sign)되는 경우도 있다. 이 경우 엔티티는 클레임(Claim)이 조작되지 않았는지 검증할 수 있다. 요즘엔 추가 보호를 위해 전체 클레임(Claim)을 암호화 하는 것이 일반화되어 있다. 어떤 클레임(Claim)에는 사용자가 인증 관리를 하는지 여부에 대한 정보가 들어있다. 이상하게 들릴 수 있는데, 사용자가 특정 인증 방법을 수행할 수 있는지 여부를 확인하기 위해 클레임(Claim)에 전달하는 경우도 있다. 인증 방법이 약하다고 판단되면 더 강력한 인증 방법을 사용하도록 사용자를 돌려보낼 수 있다. 어떤 표준은 다른 표준이 매우 엄격하게 다루는 클레임(Claim)에 매우 유연하다. 이메일 주소, 이름같은 것들을 확인하기도 한다. 클레임(Claim)에 역할, 그룹, 멤버십과 같은 것들을 전달한다.

## Claim Transformation
클레임 (Claim)
Sometimes you hear the word claim transforamtion so here's two example of claim transformation. In alternative one, I use one type of claim to gain access to an identity provider. in this case I use Kerberos once authenticated the identity provider sends me forward with a new kind of claim. this time is saml-based claim. another way of viewing claim transformation is if you change the content of the claim as an example the user may authenticate using username and password and thereby authenticate into the identity privider as an individual. But the application may not need individual users separation. so the only thing that may be needed in the claim is group membership or the role employee of a certain company.
![09](/assets/images/posts/20200410EnterpriseIAM/09.png)
![10](/assets/images/posts/20200410EnterpriseIAM/10.png)

## Chained Federation
you can build chains of federating entities with each hop the 