---
title: "Node JS Serialization Vulnerability"
date: 2018-05-24 00:00:00 +0900
---
SaaS와 MBaaS에 대해 다룬 적이 있다. 글로벌 IT 기업들이 제공하고 있는 가장 최신의 서비스 플랫폼이며, 현재 가장 많이 사용되고 있는 플랫폼이기도 하다. 하지만 여기서 그치지 않고 서비스 플랫폼은 서버리스(Serverless) 형태로 발전하고 있다. 말 그대로 서버가 없음을 의미하지만 사실 서버는 존재하고, 개발자들은 애플리케이션의 기능(Functions)에만 집중하는 형태다. FaaS(Functions as a Service)<sup id="a1">[1](#footnote1)</sup>로 불리는 이 서비스는 Google Cloud Functions가 대표적이다. 그리고 여기서 사용하는 대표적인 언어가 Javascript로 만들어진 Node.js다.
Node JS 실행 환경

사실 Javascript 런타임(프로그래밍 언어가 구동되는 환경)은 브라우저에만 있었다. 하지만 이 한계를 극복하고 Node.js가 만들어졌다. Node.js는 REPL(Read, Eval, Print, Loop)을 통해 런타임을 제공한다. 윈도우의 




><b id="footnote1">1</b> 서버 시스템에 대해 신경쓰지 않아도 된다는 점에서 PaaS와 헷갈릴 수 있는데, PaaS는 서비스가 24시간 동작하는 반면, FaaS는 특정 이벤트가 발생했을 때에만 실행되며, 작업을 마치면 종료되는 차이점이 있다.