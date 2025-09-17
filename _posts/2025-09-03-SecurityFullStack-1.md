---
title: "Security Full Stack(1) - 인프라/네트워크 보안"
# author: wrkholic84
date: 2025-09-03 00:06:00 +0900
categories: [Cloud]
tags: [cloud]
math: true
mermaid: true
published: true
# image:
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---
보안 풀스택(FullStack) 엔지니어를 위한 정리
- 네트워크 관점에서 보안 패러다임 진화 과정
- Kubernetes를 기반으로 한 소프트웨어에서 발생 가능한 침투 테스트 취약점

## 보안 패러다임 진화 과정 (네트워크)
1~3단계는 사용자와 네트워크 중심 보안,
4단계는 환경 통합 보안(멀티 클라우드),
5단계는 데이터·워크로드·자동화 중심 보안 으로 진화함.
### 1단계 경계 보안(Perimeter Security)
전통적인 데이터센터때의 모습으로 내부망은 안전하고, 외부는 위험하다는 단순한 구조. 보안은 네트워크 경계(Perimeter)를 중심으로 이루어짐.
#### 보안
방화벽(Firewall), IDS/IPS, VPN
#### 한계
- 내부 침입자가 생기면 내부망 침투 가능(Lateral Movement)
- 내부 사용자·내부 서버를 기본적으로 신뢰하여 내부 위협에 취약

### 2단계 네트워크 세분화(Network Segmentation)
경계보안의 한계를 보완하기 위해 내부망을 구간 단위로 나누어 보호.
(예, DMZ를 두고, DB는 내부망에서만 관리하여 안전을 확보하는 것)
#### 보안
VLAN, 서브넷, DMZ로 논리적 구간 분리
#### 한계
- 같은 서브넷 내에 침투 시 DB, 서버 등에 자유롭게 접속 가능
- Security Group/ACL로 구간을 나눠도, 허용된 서비스 간 통신은 제한 불가
- 여전히 내부 구간을 신뢰하고 있음

### 3단계 강화된 인증·접근 제어(Identity-Centric Security)
네트워크 세분화의 한계로 단일환경(onPremise 또는 단일 클라우드)에서 사용자·서비스의 정체성(Identity) 기반 접근 제어로 전환.  
사용자·서비스의 정체성을 검증한다는 점에서 강화된 인증·접근제어 단계에서 Zero Trust의 초기 형태가 등장.  
이 단계에서 Zero Trust 개념이 시작되고, 실제 구현은 이 다음 단계에서 확산됨.
#### 보안
IAM, MFA, RBAC, ABAC
#### 한계
단일환경(onPremise 또는 단일 클라우드)에선 효과적이지만, onPremise + 멀티 클라우드 환경으로 확장되면서 문제 발생
- 직원 퇴사 시, AD계정은 삭제되지만, 다른 클라우드 계정은 남아있을 수 있음
- 애플리케이션 권한 관리가 제각각인 문제(회사는 ERP, AWS는 IAM, GCP는 Role)
- 내부망은 DLP가 적용되어 있으나, 클라우드 환경으로 데이터 전송 시 평문 전송 가능성
- 서비스간 정체성 검증 미흡해서 악성 워크로드 위장 가능

### 4단계 하이브리드/멀티 클라우드 (Cloud Security Tools)
기업의 IT환경이 온프레미스 + 멀티 클라우드로 확장되어
환경마다 다른 보안 모델·IAM·정채으로 통합관리가 어려워 단순 Identity 통제가 아니라 환경 전체를 아우르는 보안체계(클라우드 전용 플랫폼) 등장
#### 보안
CSPM, CWPP, CIEM
#### 한계
- 환경 복잡성 증가
- 통합 Identity 관리의 한계
- 데이터 보안·규제 준수 문제
- 공격 표면 확대
- Shadow IT (개발자가 승인 없이 클라우드 리소스 생성) 같은 통제 불가능 영역도 한계
- 멀티 클라우드 환경에서 Observability/Telemetry 부족 (각 클라우드의 로그·이벤트 형식 불일치) 보안 가시성 문제

### 5단계 데이터·워크로드·자동화 중심 보안 (Data & Workload-Centric, Automated Security)
클라우드 환경이 표준화되고, 기업의 IT 운영이 DevOps/클라우드 네이티브 방식으로 자리잡으면서 보안도 단순한 “환경 통합” 수준을 넘어 데이터 자체·워크로드 자체·자동화된 대응을 중심으로 진화함.
이 단계에서는 보안이 네트워크나 인프라에 종속되지 않고, 데이터 흐름·워크로드 동작·자동화된 정책 집행을 중심으로 이루어짐.
#### 보안
DSPM, CWPP 확장, DevSecOps & Policy as Code, SOAR 및 자동화된 대응, Zero Trust 확장
#### 한계
- 데이터 보호 규체 강화(GDPR, 금융권 규제 등) 글로벌 규제 준수 난이도
- DevSecOps 도입시 개발팀과 보안팀 협업 문제 (속도 vs 보안 균형)
- AI 기반 탐지 자동화 도입 시 오탐/과탐 문제
- 클라우드 네이티브 아키텍처의 복잡성 증가(MSA, 멀티 클러스터)

### 정리  
현재 3단계에서 5단계까지 모습으로 다양한 인프라 환경에서 서비스 제공 중. 서비스의 규모 및 중요성에 따라 5단계에 가까워지는 느낌.


|단계 |패러다임|주요 보안 수단|한계|
|----|:-----|:---------|:--|
|1단계|경계 보안<br>(Perimeter Security)|Firewall, IDS/IPS, VPN|- 내부 침입 시 Lateral Movement 가능<br>- 내부 사용자·서버 기본 신뢰 → 내부 위협 취약|
|2단계|네트워크 세분화<br>(Network Segmentation)|VLAN, Subnet, DMZ, Security Group, ACL|- 같은 구간 침투 시 내부 자원 자유 접근<br>- 허용된 서비스 간 통신 제한 불가<br>- 내부 구간 여전히 신뢰|
|3단계|강화된 인증·접근 제어<br>(Identity-Centric Security)|IAM, MFA, RBAC, ABAC, Zero Trust(초기)|- 단일환경에서는 효과적<br>- 멀티 클라우드 확장 시 계정 Lifecycle 불일치<br>- 권한/역할 관리 불일관성<br>- 서비스 간 정체성 검증 부족|
|4단계|하이브리드·멀티 클라우드<br>(Cloud Security Tools)|CSPM, CWPP, CIEM, CNAPP(통합)|- 환경 복잡성 증가<br>- 통합 Identity 관리 한계<br>- Shadow IT 문제<br>- 데이터 보안·규제 준수 어려움<br>- 클라우드 간 가시성 부족|
|5단계|데이터·워크로드·자동화 중심 보안<br>(Data & Workload-Centric, Automated Security)|DSPM, 확장된 CWPP(K8s/서버리스), DevSecOps,<br>Policy as Code, SOAR, AI/ML 기반 보안|- 글로벌 규제 준수 난이도↑<br>- DevSecOps 협업 어려움<br>- 자동화 시 오탐/과탐 문제<br>- 마이크로서비스·API·클러스터 복잡성 증가|



## Zero Trust
3단계 사용자·서비스의 정체성을 검증한다는 점에서 강화된 인증·접근제어 단계에서 Zero Trust의 초기 형태가 등장
멀티 클라우드 단계에서 내부/외부 구분이 모호해지면서 Zero Trust 필요성이 커짐. CSPM, CWPP, CIEM도 결국 Zero Trust를 구현하기 위한 기반 개념 중 일부
클라우드 네이티브 보안 단계가 본격적으로 Zero Trust가 구현되는 단계
- 내부도 신뢰하지 않는다
- 


### 클라우드 네이티브 보안 / CNAPP
#### 보안
- 서비스간 mTLS, 마이크로세그멘테이션, 컨텍스트 기반 접근 제어, 지속 검증 적용


우리가 점검하는 방향
Zero Trust 기반의 보안 점검을 해야 한다.
1. 내부/외부 구분 없이 모든 접근 검증
애플리케이션이 어느 플랫폼에 구축되더라도, 접근은 검증되어야 함

2. 최소 권한의 원칙
사용자·디바이스서비스에 최소한의 권한만 부여되었는지 확인

3. 침해 가정
이미 내부가 뚫렸다는 가정 하에 방어 체계 설계되었는지 확인

4. 지속 검증
1회 인증으로 끝나지 않고, 세션 중에도 지속적 평가 필요

5. 맥락 기반 접근
사용자·디바이스 상태·위치·행위 패턴 등 맥락에 따라 접근 허용 및 차단하고 있는지 확인

추가 할 일
위 일들을 AI등을 활용해 자동화 할 수 있는지 연구