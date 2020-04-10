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