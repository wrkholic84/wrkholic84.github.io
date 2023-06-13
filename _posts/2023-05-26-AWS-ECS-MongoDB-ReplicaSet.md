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