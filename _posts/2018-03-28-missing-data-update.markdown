---
layout: post
title:  "DB에 누락된 정보를 로그에서 추출하여 수동 업데이트 하기"
date:   2018-03-28 00:00:59
categories: DevLog
tags:   diff sort awk tr
---

### DB에 누락된 정보와 실제 작업이 수행된 데이터 준비하기 
물론 있어서는 안 되는 일이겠으나.. DB 장애 등으로 인해 데이터 업데이트를 실패하여 실제 어플리케이션 동작과 DB 데이터가 일치하지 않는 경우가 있었다. 어플리케이션 로그를 확인해서 DB 데이터를 수동으로 업데이트하면서 사용한 커맨드를 정리해봤다. 

로그를 분석해 실제 작업이 수행된 데이터를 real_executed.txt 파일에 추출하고 DB에 업데이트 되어 있는 데이터를 sql을 통해 추출하자. DB 클라이언트를 사용하고 있다면 csv 파일로 추출하기 어렵지 않다. 추출한 파일을 db_updated.csv 라고 하자. 두 파일은 아래와 같이 '\n' 으로 구분되어 있도록 추출했다.

```
// real_executed.txt
a
b
c
d
e
```


```
// db_updated.csv
a
b
c
...
```

<br/>

### 실제 실행된 데이터에서 DB 결과를 제외해서 업데이트 실패한 데이터를 추출
real_executed.txt 와 db_updated.csv 를 동일한 포맷으로 추출했다면 diff 명령어를 통해 누락된 내용을 쉽게 파악할 수 있을 것 같다. 겉으로 보면 log에서 추출한 txt 파일이나, DB에서 추출한 csv 파일은 포맷이 동일하다. 하지만 csv 파일은 delimiter가 달라서 diff가 제대로 동작하지 않는다. 때문에 정석적으로는 csv 파일을 txt로 변환한 후 비교해야 하지만 이번에는 sort를 사용했다. sort를 하면 정렬과 함께 csv의 delimiter를 txt와 동일하게 바꾸어 주는 것으로 보인다.

``` shell
cat real_executed.txt | sort -n > sorted_real_executed.txt
cat db_updated.csv | sort -n > sorted_db_updated.csv.txt
```

이제 diff를 통해 누락된 정보만 추출하자.
```
diff sorted_db_updated.csv.txt sorted_real_executed.txt > should_update.txt
```

<br/>


### 누락된 발송 결과 DB에 Update 하기 
누락된 정보를 추출한 후 DB에서 쿼리를 수행하기 위해서는 쿼리에 필요한 스트링으로 변환을 해야 한다. 사용하고 있던 업데이트문은 IN 절을 사용하고 있었기 때문에 "" 로 데이터를 감싸야 하고, '\n' 을 ',' 로 구분해야 한다. 
```sql
UPDATE alphabet_table
SET    status = "SUCCESS",
WHERE  element IN ("c", "d" ..... "z")
```

 
```
cat should_update.txt | awk '{print "\""$1"\""}' | tr '\n' , > sql_string.txt
```

awk 로 쌍따옴표를 추가하고, '\n'을 컴마로 변경해주면 결과는 다음과 같다. 결과 스트링을 IN 절에 붙여 넣고 SQL을 실행시키면 된다. 
```
"c","d","e", .... "z",
```


<br/>
