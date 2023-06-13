---
title: "AWS ECS에 MongoDB ReplicaSet 구축"
author: wrkholic84
date: 2023-05-24 00:00:00 +0900
categories: [Cloud, AWS, MongoDB, ReplicaSet]
tags: [AWS, ECS, DB]
math: true
mermaid: true
---
## AWS ECS에 MongoDB ReplicaSet 구축하기
MongoDB 운영 중 장애 발생에 대응하기 위해 Replica Set 구성을 해줘야 한다.

기존에 운영중인 컨테이너 서비스용 서버(ECS등)가 있다면, 여기에 MongoDB Replica Set 구성을 추가하여 사용하면 간편하다.

### 1. 아키텍처 구성
![00](/assets/images/posts/20230526AWSECSMongoDBReplicaSet/00.png)

기존에 사용하던 ECS 컨테이너에 DB Service를 추가하는 방식으로 구성한다.

### 2. 볼륨 생성
EC2 > 볼륨 > 볼륨 생성
범용 SSD, 30GiB, ap-northeast-2a 으로 볼륨 생성

만들어진 볼륨을 ECS 인스턴스에 연결
ECS 인스턴스 선택, 디바이스 이름 (/dev/sdf 확인)
EC2에 연결되는 볼륨은 sdf부터 알파벳순으로 sdg, sdh과 같이 이름이 붙여진다.
EC2 내부적으로 표시되는 이름은 /dev/xvdf ~ /dev/xvdp 로 바뀐다.

### 3. 볼륨 마운트
ECS 인스턴스에서 (논리적으로)연결된 볼륨을 마운트한다.
먼저, lsblk 명령으로 사용 가능한 디스크 디바이스 및 마운트 포인트를 확인
Nitro 시스템 기반 인스턴스의 경우 아래와 같이 NVMe 블록 디바이스로 표시되고, 그 외(T2인스턴스 등)의 경우 /dev/xvda 등으로 표시된다.

### 4. 작업 정의
ECS에서 사용할 작업을 만들어준다.
이 작업에는 
* 총 3개의 MongoDB 컨테이너가 동작하며,
* 앞서 마운트한 볼륨을 컨테이너가 공유하여 사용한다
* 각각의 DB 컨테이너는 앞에서 생성한 3개 디렉토리를 각자의 역할에 맞추어 데이터 디렉토리로 사용하며,
* Replica Set 인증키를 공유하여 Replica Set의 구성원이 된다
* DB 접속을 위한 계정을 만든다
* 클라우드 와치에 연결하여 로그를 수집한다.

<작업 정의 생성>  
네트워크 모드 : default  
호환성 요구 : EC2  
볼륨추가  
#1.   
이름 : mongodb-arbiter  
볼륨 유형 : Bind Mount  
소스 경로 : /mnt/data/mongodb/arbiter  
#2.  
이름 : mongodb-primary  
볼륨 유형 : Bind Mount  
소스 경로 : /mnt/data/mongodb/primary  
#3.  
이름 : mongodb-secondary  
볼륨 유형 : Bind Mount  
소스 경로 : /mnt/data/mongodb/secondary  
#4.  
이름 : mongodb-replica-key  
볼륨 유형 : Bind Mount  
소스 경로 : /mnt/data/mongodb/mongodb.key  
  
<컨테이너 정의>  
#1.  
컨테이너 이름 : mongodb-primary  
이미지 : mongo:latest  
하드제한 : 512MB  
포트 매핑 : 27017 / 27017 / tcp  
환경>   
명령 :   
--replSet,< Replica Set Name >,--keyFile,/etc/mongodb.key,--bind_ip_all,--auth
환경변수 :   
MONGO_INITDB_ROOT_PASSWORD / <DB비밀번호>
MONGO_INITDB_ROOT_USERNAME / <DB아이디>  
스토리지 및 로깅>  
탑재지점>  
#1. mongodb-primary > /data/db  
#2. mongodb-replica-key > /etc/mongodb.key  
