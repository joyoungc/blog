---
layout: post
title:  "유용한 정규식(Regular expression) 정리"
date:   2015-12-21 17:57:00
categories: java
tags: java regex 마스킹
author: Joyoungc
---

- 언어는 Java 를 기준으로 작성 되었습니다. 
- JavaScript에서는 [lookbehind](https://stackoverflow.com/questions/2973436/regex-lookahead-lookbehind-and-atomic-groups) 기능을 지원하지 않기 때문에 동작하지 않을 수 있습니다.

1. 앞에서 n자리 뒤의 문자들을 마스킹 처리

```java
String example = "mask1234";
example = example.replaceAll("(?<=.{4}).", "*");
System.out.println(example); // output : mask****
```