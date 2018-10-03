---
layout: post
title:  "서버 부팅시 자동 실행 스크립트 등록 - chkconfig"
date:   2018-04-20 00:00:59
group: 개발일지
tags:   linux 
---

chkconfig는 /etc/rc[0-6].d 디렉토리를 유지보수하기 위한 도구다.
chkconfig를 이용하면 runlevel에 따른 데몬의 자동실행 유무를 빠르게 설정할 수 있다.

<br/>

## chkconfig 확인
각 필드별 숫자의 의미는 runlevel이며 on, off의 의미는 해당 runlevel에서 실행될지 여부이다.
chkconfig 버전에 따라 --list 옵션이 없어도 동작하기도 한다.

```shell
hello@MacBook:/ [18:14:59]$ chkconfig --list | grep servicectl
servicectl      0:off   1:off   2:off   3:off   4:off   5:off   6:off
```

<br/>

## chkconfig 등록
먼저 실행될 스크립트의 2번째 라인부터 아래와 같이 주석을 추가 하자
```shell
#!/bin/sh
# chkconfig: 345 20 80 // <runlevel> <실행우선순위> <중지우선순위>
# description: this is servicectl
```

/etc/init.d 디렉토리에 해당 스크립트를 복사한 후에 아래 명령어로 등록하자
```shell
chkconfig --add <스크립트명>
```

<br/>

## chkconfig 해제

```shell
chkconfig --del <스크립트명>
``` 

위 명령어로 해제가 안 될때가 있는데 이럴때는 /etc/init.d 아래에 있는 스크립트를 직접 삭제하자.
버그인지 모르겠으나 스크립트를 삭제하니 chkconfig에서도 제거가 되었다.

<br/>
