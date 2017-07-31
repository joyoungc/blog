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

## 유용한 예제
#### 1. 앞에서 n자리 뒤의 문자들을 마스킹 처리
```java
// Masking 정규식
String regexMasking = "(?<=.{4}).";

String example = "mask1234";

// lookbehind 정규식을 사용하여 앞의 4자리 문자를 제외한 나머지 문자들을 * 처리한다.
example = example.replaceAll(regexMasking, "*");

System.out.println(example); // output : mask****
```

#### 2. 전화 번호 유효성 체크
```java
//  dash 포함 번호에 대한 정규식
String regexMobilePhoneNumberWithDash = "^(01[016789]{1})-([0-9]{3,4})-([0-9]{4})$";

// dash 미포함 번호에 대한
String regexMobilePhoneNumberNoDash = "^(01[016789]{1})([0-9]{3,4})([0-9]{4})$"; 정규식

String test1 = "010-1234-5678";
String test2 = "01056781234";

// 1. 대쉬가 포함된 핸드폰 번호 유효성 검증
Pattern r1 = Pattern.compile(regexMobilePhoneNumberWithDash);
Matcher m1 = r1.matcher(test1);

if (m1.find()) {
  for (int i = 0; i < (m1.groupCount() + 1); i++) {
    System.out.println("Group " + i + " : " + m1.group(i));
    /* output :
      Group 0 : 010-1234-1234
      Group 1 : 010
      Group 2 : 1234
      Group 3 : 5678
    */
  }
}

System.out.println("## MobilePhoneNumberWithDash is :" + m1.matches()); // output : true

// 2. 대쉬가 포함되지 않은 핸드폰 번호 유효성 검증
Pattern r2 = Pattern.compile(regexMobilePhoneNumberNoDash);
Matcher m2 = r2.matcher(test2);

if (m2.find()) {
  for (int i = 0; i < (m2.groupCount() + 1); i++) {
    System.out.println("Group " + i + " : " + m2.group(i));
    /* output :
      Group 0 : 01056789999
      Group 1 : 010
      Group 2 : 5678
      Group 3 : 1234
    */
  }
}

System.out.println("## MobilePhoneNumberNoDash is : " + m2.matches()); // output : true
```

## 용어정리
###### Predefined character classes
| Regex Code | Description |
| ----- | ----- |
| . | Any character |
| \d | A digit : [0-9]  |
| \D | A non-digit : [^0-9]  |
| \s | A whitespace character : [ \t\n\x0B\f\r] |
| \S | A non-whitespace character : [^\s] |
| \w | A word character : [a-zA-Z_0-9] |
| \W | A non-word character : [^\w] |

<br/>
###### Logical operators

| Regex Code | Description |
| ----- | ----- |
| XY | X followed by Y |
| X|Y | Either X or Y |
| (X) | X, as a capturing group^⑴^ |
> (1) 괄호 안의 정규식을 Group화 할 수 있다. Group 0 은 언제나 전체 표현식을 나타낸다.

<br/>
###### Special constructs (named-capturing and non-capturing)
| Regex Code | Description |
| ----- | ----- |
| (?:X) | X 정규식이 일치하는 것에 대해  capturing group에서 제외(as a non-capturing group) |
| (?<name>X) | X 정규식이 일치하는 capturing group에 name을 부여할 수 있음.^⑴^  (X, as a named-capturing group) |
| (?<=X)Y | X 정규식이 일치한다면 뒤에 나오는 Y 정규식에 일치하는 데이터를 반환 (X, via zero-width positive lookbehind) |
| (?<!X)Y | X 정규식이 일치하지 않는다면 뒤에 나오는 Y 정규식에 일치하는 데이터를 반환 (X, via zero-width negative lookbehind) |
 > (1)  Java7 부터 지원

<br/>
###### Quantifiers
http://docs.oracle.com/javase/tutorial/essential/regex/quant.html

| Greedy | Reluctant | Possessive | Meaning |
| ----- | ----- | ----- | ----- |
| X? | X?? | X?+ | X가 1개이거나 존재하지 않을때 (X, once or not at all) |
| X* | X*? | X*+ | X가 0개이거나 그 이상이 존재할때 (X, zero or more times) |
| X+ | X+? |X++ | X가 1개이거나 그 이상이 존재할때 (X, one or more times) |
| X{n} | X{n}? | X{n}+ | X가 정확이 n번 반복된 것 (X, exactly n times) |
| X{n,} | X{n,}? | X{n,}+ | X가 최소한 n번 반복된 것 (X, at least n times) |
| X{n,m} | X{n,m}? | X{n,m}+ | X가 n 이상 m 이하 |

- A greedy quantifier first matches as much as possible.
- A reluctant or "non-greedy" quantifier first matches as little as possible.
- A possessive quantifier is just like the greedy quantifier, but it doesn't backtrack.


## 유용한 사이트
http://www.regexplanet.com (각 언어에 대한 정규식 테스트 사이트)
https://www.slideshare.net/ibare/ss-39274621
