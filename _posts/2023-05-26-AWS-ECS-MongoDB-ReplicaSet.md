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
```bash
[ec2-user@ip-0-0-0-0 ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme0n1       259:0    0  30G  0 disk 
├─nvme0n1p1   259:1    0  30G  0 part /
└─nvme0n1p128 259:2    0   1M  0 part 
nvme1n1       259:3    0  30G  0 disk 
```
nvme1n1 볼륨이 확인되었으니, 파일 시스템 유형을 확인해본다. data라고만 표시되면 파일 시스템이 없는 것이다.
```bash
[ec2-user@ip-0-0-0-0 ~]$ sudo file -s /dev/nvme1n1
/dev/nvme1n1: data
```
다음으로 볼륨에 파일 시스템을 생성해본다. 데이터베이스로 사용할 볼륨이라, xfs 포멧으로 생성한다.
```bash
[ec2-user@ip-0-0-0-0 ~]$ sudo mkfs -t xfs /dev/nvme1n1
```
그리고 다시 파일 시스템 유형을 확인해보면, 아래와 같이 바뀌어 표시된다.
```bash
[ec2-user@ip-0-0-0-0 ~]$ sudo file -s /dev/nvme1n1
/dev/nvme1n1: SGI XFS filesystem data (...)
```
이제 원하는 경로에 볼륨을 마운트 한다.
```bash
[ec2-user@ip-0-0-0-0 ~]$ sudo mount /dev/nvme1n1 /mnt/data
```
그리고 다시 블록 디바이스 정보를 확인하면 아래와 같이 /mnt/data 에 마운트 된 것을 알 수 있다.
```bash
[ec2-user@ip-0-0-0-0 ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme0n1       259:0    0  30G  0 disk 
├─nvme0n1p1   259:1    0  30G  0 part /
└─nvme0n1p128 259:2    0   1M  0 part 
nvme1n1       259:3    0  30G  0 disk /mnt/data
```

### 3. MongoDB Replica Set에서 사용할 디렉토리 및 파일 생성
위 그림과 같이 3개 DB를 운영하기 위한 데이터 디렉토리 3개와 Replica Set 인증키 파일 1개를 방금 마운트 한 볼륨에 생성해준다.
```bash
[ec2-user@ip-0-0-0-0 data]$ cd mongodb
[ec2-user@ip-0-0-0-0 mongodb]$ pwd
/mnt/data/mongodb
[ec2-user@ip-0-0-0-0 mongodb]$ mkdir primary
[ec2-user@ip-0-0-0-0 mongodb]$ mkdir secondary
[ec2-user@ip-0-0-0-0 mongodb]$ mkdir arbiter
[ec2-user@ip-0-0-0-0 mongodb]$ openssl rand -base64 756 > ./mongodb.key
[ec2-user@ip-0-0-0-0 mongodb]$ chmod 400 ./mongodb.key
```

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
