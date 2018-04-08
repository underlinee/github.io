---
layout: post
title:  "Array에 invalid한 값이 포함된 경우 default 값으로 변경하기"
date:   2018-04-02 00:01:59
categories: DevLog
tags:   java8 stream 
---

## 문제
특정 배열에서 값이 빈 문자열인 경우 빈 문자열 대신 default 값을 사용해야하는 이슈가 있어 
해결 방법을 정리해봤다. 

아래의 파이썬의 defaultdict 처럼 사용할 수 있는 방법이 없을까 생각해보았으나 좋은 생각이 나지 않았다. 
Optional.orElse()는 값이 null인 경우에 사용할 수 있기 때문에 활용하기 어려워 보였다. 

```
from collections import defaultdict 
dict = defaultdict(lambda: 0) # Default 값을 0으로 설정
```

그렇다고 배열을 사용하는 시점에 null 체크 후 다른 값으로 변경해주는 방법도 있었으나,
배열의 요소들이 사용되는 시점마다 체크 해주어야 하기 때문에 좋지 않은 방법이라고 생각했다. 

<br/>

## 선택한 방법
결국은 미리 .isEmpty() 로 빈 문자열인지 확인 후 디폴트 값으로 변경해주기로 했다. 
for문은 번거로워 stream 처리하였으나 Index가 필요한 관계로 IntStream.range를 사용하고
결과값을 리턴 받을때 toArray에서 String 생성자를 generator로 받아 String 배열을 생성했다. 
더 좋은 방법은 없을까.

```
    private String[] defaultItems = {"1", "2", "3", "4", "5"};

    public String[] convertToDefaultWhenEmpty(String[] strArray) {
        return IntStream.range(0, strArray.length)
                .mapToObj(index -> (strArray[index].isEmpty()) ? defaultItems[index] : strArray[index])
                .toArray(String[]::new);
    }
```
