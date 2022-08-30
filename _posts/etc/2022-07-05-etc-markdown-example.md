---
title: '[Markdown] 블로그 내 POST 작성 예제'
excerpt: '첫 글입니다. 축하해주세요.'

categories:
  - etc
tags:
  - [markdown, md, blog]

permalink: /etc/example/

toc: true
toc_sticky: true

date: 2022-07-05
last_modified_at: 2022-07-07
---

## 🗂 Header2

### 📌 Post 양식 (Header3)

| property         | desc                                             |
| ---------------- | ------------------------------------------------ |
| title            | post 제목                                        |
| categories       | post 카테고리                                    |
| tags             | post 태그 - 자유롭게 작성 가능한 듯              |
| toc              | table of contents                                |
| toc_label        | toc의 이름 -> default 값 변경 가능               |
| toc_icon         | toc의 아이콘                                     |
| toc_sticky       | toc 화면 고정 유무, 스크롤 여부에 관계 없이 고정 |
| last_modified_at | post 최근 수정일                                 |

### 📌 notice 박스 (Header3)

<div class="notice">저는 notice (default) 박스입니다.</div>

<div class="notice--warning">저는 notice--warning 박스입니다.</div>

<div class="notice--primary">저는 notice--primary 박스입니다.</div>

### 📌 Header3

> 정의나 강조 문구 표현하기 <br>
> 좋은 것 같습니다.

### 1. First:

일반 텍스트입니다. This is plain text. <br>
**e.g. orange, banana, apple, grape**

### 2. Second:

일반 텍스트입니다. This is plain text. <br>
**e.g. orange, banana, apple, grape**

### 3. Third:

일반 텍스트입니다. This is plain text. <br>
**e.g. orange, banana, apple, grape**

밑줄을 쫙 <u>그어보겠습</u>니다.

`code`를 사용해봅니다.

```js
const fruit = {
  name: 'banana',
  color: 'yellow',
};
```

[🏠 블로그 홈으로 가보기](https://uhjee.github.io)
