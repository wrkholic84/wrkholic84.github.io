---
title: "X-Forwarded-For 스푸핑과 방어 전략"
# author: wrkholic84
date: 2026-03-15 00:00:00 +0900
categories: [Security]
tags: [security, network, nodejs, express, aws]
math: false
mermaid: true
published: true
---

Node.js(Express) + AWS ALB 환경에서 실제 클라이언트 IP를 로그에 기록하는 과정에서 마주치는 X-Forwarded-For 스푸핑 이슈와 방어 전략을 정리한다.

## X-Forwarded-For 헤더란

HTTP 요청이 프록시나 로드밸런서를 경유할 때, 원래 클라이언트 IP를 전달하기 위한 헤더다.

```
X-Forwarded-For: client, proxy1, proxy2
```

- **왼쪽**: 원본 클라이언트 IP (또는 가장 처음 프록시가 추가한 값)
- **오른쪽**: 가장 마지막에 경유한 프록시가 추가한 값

각 프록시는 자신이 받은 요청의 출발 IP를 헤더 **끝(오른쪽)에 append** 한다.

## 스푸핑 시나리오

공격자가 헤더를 직접 조작해서 보내면 어떻게 될까?

```
공격자 요청: X-Forwarded-For: fake-ip

AWS ALB이 실제 IP를 append:
X-Forwarded-For: fake-ip, real-client-ip
```

공격자는 앞쪽(왼쪽)에 값을 삽입할 수 있지만, AWS ALB은 항상 실제 연결 IP를 맨 끝에 붙인다.

```
공격자 요청: X-Forwarded-For: fake1, fake2, fake3

AWS ALB이 실제 IP를 append:
X-Forwarded-For: fake1, fake2, fake3, real-client-ip
```

몇 개를 삽입해도 ALB이 추가한 값은 항상 **맨 오른쪽**에 위치한다.

## 방어 방법

### 방법 1: 마지막 IP 직접 파싱

```typescript
req.headers['x-forwarded-for']
  ? String(req.headers['x-forwarded-for']).split(',').at(-1)?.trim()
  : req.ip
```

`.at(-1)`로 맨 오른쪽 값(ALB이 추가한 실제 IP)을 가져온다. 공격자가 앞에 아무리 많이 삽입해도 영향 없다.

### 방법 2: Express trust proxy 설정 (권장)

```typescript
app.set('trust proxy', 1); // ALB 1개를 신뢰
```

`trust proxy: 1`의 의미: 서버 바로 앞의 프록시 1개(ALB)를 신뢰하겠다.

Express 내부적으로 [`proxy-addr`](https://github.com/jshttp/proxy-addr) 패키지를 사용해 동일한 로직을 처리한다.

| trust proxy 값 | 동작 |
|---|---|
| 설정 안 함 | `req.ip` = socket remote address (ALB IP) |
| `1` | 오른쪽 1개(ALB)를 신뢰, `req.ip` = 실제 클라이언트 IP |
| `2` | 오른쪽 2개(CDN + ALB)를 신뢰, `req.ip` = 실제 클라이언트 IP |

설정 후에는 `req.ip`만으로 올바른 클라이언트 IP를 얻을 수 있어, 수동 파싱이 필요 없다.

```typescript
// trust proxy: 1 설정 후
app.set('trust proxy', 1);

// 아래 두 코드는 동일한 결과
req.ip
String(req.headers['x-forwarded-for']).split(',').at(-1)?.trim()
```

## 프록시 구성별 trust proxy 설정

```mermaid
graph LR
    A[클라이언트] --> B[AWS ALB]
    B --> C[Express 서버]
```
→ `trust proxy: 1`

```mermaid
graph LR
    A[클라이언트] --> B[CDN]
    B --> C[AWS ALB]
    C --> D[Express 서버]
```
→ `trust proxy: 2`

## 앱 레벨의 한계 — 인프라 레벨 방어 필요

앱 레벨 방어의 전제 조건: **모든 요청이 반드시 ALB을 경유해야 한다.**

공격자가 ALB을 우회하여 Express 서버에 직접 접속하는 경우:

```
공격자 → (ALB 우회) → Express 서버 직접 연결
X-Forwarded-For: fake-ip  (ALB이 append하지 않음)

req.ip = fake-ip  ✗
```

이 경우 어떤 앱 레벨 방어도 소용없다. **AWS 보안 그룹**으로 막아야 한다.

```
서버 인바운드 규칙:
ALB의 보안 그룹에서 오는 트래픽만 허용
외부 직접 접근 차단
```

## 정리

| 공격 방법 | 방어 수단 | 레벨 |
|---|---|---|
| XFF 헤더에 fake IP 삽입 | `.at(-1)` 또는 `trust proxy` + `req.ip` | 앱 |
| ALB 우회 직접 접속 | AWS 보안 그룹으로 외부 직접 접근 차단 | 인프라 |

앱과 인프라 두 레벨 모두 올바르게 설정되어야 IP 기반 로깅 및 접근 제어가 신뢰할 수 있는 값을 사용한다.
