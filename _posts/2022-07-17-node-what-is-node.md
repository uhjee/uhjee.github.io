---
title: '[Node.js] Node.js란?'
excerpt: 'Javascript 런타임, Node.js에 대해서 알아봅시다.'

categories:
  - Node
tags:
  - [Node.js, Node, node, nodejs, runtime]

permalink: /node/what-is-node/

toc: true
toc_sticky: true

date: 2022-07-17
last_modified_at: 2022-07-17
---

## 📌 Node.js란?

Node.js는 크롬 V8 javascript 엔진으로 빌드된 **javascript 런타임**입니다.

## 📌 Javascript 런타임이란?

Javascript 런타임이란 <u>javascript를 실행시킬 수 있는 환경</u>을 말합니다.

기존에는 javascript 엔진이 있는 브라우저에서만 javascript 실행이 가능했었습니다. 하지만 2008년에 구글이 V8 엔진을 개발하고 오픈 소스로 코드를 공개했습니다. 이를 기반으로 라이언 달 선생님이 Node.js를 개발했습니다.

## 📌 Node.js 내부 구조

![eventloop_001.png](/assets/images/posts_img/node/eventloop_001.png){: .align-center}

위 그림과 같이 Node.js는 V8 javascsript 엔진과 Libuv 라이브러리를 기반으로 구성되어 있습니다. 그리고 노드는 우리가 작성한 코드를 V8 엔진, Libuv 라이브러리와 연결시켜 줍니다.

Libuv 라이브러리의 역할은 다음과 같고, 이 역할들은 곧 Node.js의 중요한 특성이 됩니다.

1. 이벤트 기반
2. 논 블로킹 I/O 모델

## 📌 Node.js 특징

### 특징 1. 이벤트 기반

이벤트가 발생할 때 미리 지정해둔 작업을 수행하는 방식

특정 이벤트가 발생했을 때, 어떤 처리를 할지 미리 등록해야함

- eventListener - callback function

#### 이벤트 루프

- 태스크 큐 task queue

  이벤트 발생한 경우, 백그라운드에서 태스크 큐로 타이머나 이벤트 리스너의 콜백 함수를 보냄

  순서대로 콜백들이 대기하고 있으므로 콜백 큐라고도 불림

  특수한 경우에 순서사 변경되기도 함

- 백그라운드 background

  setTimeout 같은 타이머, 이벤트 리스너들이 대기하는 곳

  시스템 커널 등 javascript가 아닌 언어로 작성된 별도의 프로그램이라고 봐도 됨

![eventloop_000.png](/assets/images/posts_img/node/eventloop_000.png){: .align-center}

이벤트 발생 시 호출할 콜백함 수들을 관리하고, 호출된 콜백 함수의 실행 순서를 결정하는 역할 담당

javascript engine으로 분류되는 힙 메모리와 콜 스택에 대해서는 이야기하지 않고 넘어가겠음

노드가 종료될 때까지 이벤트 처리를 위한 작업을 반복하므로 Loop

콜 스택이 비었을 때에만 태스크 큐에서 콜백을 올림

### 특징 2. 논 블로킹 I/O

이전 작업이 완료될 때까지 대기하지 않고 다음 작업을 수행하는 것

Node.js는 I/O 작업에 대해서 논 블로킹 처리

동시에 처리될 수 있는 작업은 백그라운드로 넘겨서 동시에 처리 → 대기 시간을 줄일 수 있음(동시 처리되는 것이 아님)

- 동시성: 동시 처리가 가능한 작업을 논 블로킹 처리해야 얻을 수 있는 속성

### 특징 3. 싱글 스레드

- Node.js 자체는 싱글 스레드로 동작하지 않지만 개발자가 직접 제어할 수 있는 스레드는 하나
- 스레드가 하나이기 때문에 논 블로킹 모델로 최대한 대기시간을 줄이는 방식으로 동작
- 멀티 스레드가 싱글 스레드보다 우월한 것이 아님
  - 자원 관리 측면에서 더 많은 비용 발생
- I/O 작업 시 멀티 프로세싱으로 많이 처리

⚠️ Node.js가 싱글 스레드로 동작하지 않는 경우

- 스레드 풀

  암호화, 파일 입출력, 압축 등의 경우 스스로 멀티 스레드 사용

- 워커 스레드

  Node 12부터 안정화된 기능으로 개발자가 직접 멀티 스레드를 다룰 수 있게 됨

## 📌  서버로서의 노드

Node.js에는 Web Server가 내장되어 있음

### 노드 서버의 장단점

싱글 스레드, 논 블로킹 모델의 특징 때문에 개수는 많지만 크기는 작은 데이터를 실시간으로 주고받는 서비스를 제공하는 서버들로 활용하기에 적합함

- 네트워크, 데이터베이스, 디스크 작업 같은 I/O에 특화
- 실시간 채팅, 주식 차트, JSON 데이터 제공하는 API 서버

싱글 스레드이기 때문에 CPU 연산을 많이 요구하는 서비스는 적합하지 않음

- 이미지, 비디오 처리, 대규모 데이터 처리
