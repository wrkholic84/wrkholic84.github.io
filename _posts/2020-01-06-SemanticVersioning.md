---
title: Semantic Versioning(SemVer)
# author: wrkholic84
date: 2020-01-06 00:00:00 +0900
categories: [Development, General]
tags: [versioning]
math: true
mermaid: true
# image:
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---
어느 정도 규모가 있는 소프트웨어를 개발할 때면 외부 라이브러리를 사용하게된다. 그리고 자연스럽게 라이브러리 의존성 문제를 접하게 된다. 의존성 문제는 라이브러리가 버전 문제로 서로 충돌하여 사용할 수 없게 되는 것이라 이해하면 쉽다.

이 문제를 해결하고자 나온 라이브러리 버전 명시 규칙이 있다.

라이브러리의 버전은 보통 a.b.c로 표기된다.

* 첫 번째 a(Major) 버전이 오르면 기존 버전과 호환되지 않음
* 두 번째 b(Minor) 버전이 오르면 새로운 기능이 추가된 경우
* 세 번째 c(Patch) 버전이 오르면 자잘한 버그 수정이 있는 경우

예를 들면,

libC 1.2.1은 새로운 버전인 libC 2.1.1과 서로 호환되지 않는 API로 구성되어 있을 것이다.

libC 1.2.1 버전을 사용하는 A 라는 프로그램이 있다고 가정해보자.
A 프로그램은 libC의 버전이 2.1.1로 오르더라도 계속해서 libC 1.2.1 버전을 사용하면서 프로그램의 안정성을 유지할 수 있다.

## NPM package.json 의 SemVer
NodeJS에서 관리하는 라이브러리(패키지)의 버전 관리를 어떻게 하는지 알아보자.

* ^2.2.1 : 2.2.1 이상 3.0.0 미만 (2.2.1 이상이면서 같은 Major 버전)
* ~2.2.1 : 2.2.1 이상 2.3.0 미만 (2.2.1 이상이면서 같은 Major 버전)
* \>2.2.1 : 2.2.1 초과 (2.2.2 이상의 버전 사용)
* 2.2.1 - 3.0.0 : 2.2.1 이상 3.0.0 이하

## Tild(~)와 Caret(^)의 차이점
Tild(~)가 쓰였을 경우 버그가 수정된 최신 버전으로 업데이트 한다.
~2.2.1 버전이라면 2.2.* 의 버전을 가져온다.

Caret(^)이 쓰였을 경우 호환 가능한 최신 버전으로 업데이트 한다.
^2.2.1 버전이라면 2.*.* 의 버전을 가져온다.

> [npm semver calculator](https://semver.npmjs.com/)