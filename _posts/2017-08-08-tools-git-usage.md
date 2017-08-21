---
layout: post
title:  "Git 기초사용법"
date:   2017-08-08 11:28:00
categories: tools
tags: git
author: Joyoungc
---

## 명령어

#### git log
히스토리를 조회하는 명령어
```
$ git log {fileName} : 해당 파일의 커밋 히스토리를 시간순으로 보여준다.
```
```
$ git log -p  {fileName} : 해당 파일에 대한 diff 결과를 보여준다.
```
```
$ git log -p -2  {fileName} : 해당 파일에 대한 최근 두 개의 결과를 보여준다.
```
<br/>
#### git commit
변경한 내용을 확정
```
$ git commit -m "Edit source"  : 메세지를 즉시 입력한다.
```
<br/>
#### git checkout
```
$ git checkout -- {fileName}  : 해당 파일을 현재 브랜치의 마지막 버전으로 replace한다.
```

<br/>
#### git rm
```
$ git rm {fileName}  : 해당 파일을 삭제한다. (삭제와 staging을 한 번에 처리)
```
