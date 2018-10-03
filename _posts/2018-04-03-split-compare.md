---
layout: post
title:  "StringUtils.split vs String.split"
date:   2018-04-02 00:00:59
group: 개발일지
tags:   java 
---

아래 코드에서 out of index 익셉션이 발생하는 버그가 있어서 원인을 확인해봤다. 
```java
String[] items = StringUtils.split(str, DELIMITER);
itemChar = items[i].charAt(1); 
```

Apache의 StringUtils.spilt은 delimiter 사이에 값이 없으면 배열에 빈 값을 추가하지 않고
다음 값을 그 자리에 할당하다. 길이가 1보다 작은 String에 대해 charAt(1) 을 호출하면서 익셉션이 발생한 것. 

StringUtils 도큐먼트에는 다음과 같이 설명되어 있다. 
```
The separator is not included in the returned String array.
     * Adjacent separators are treated as one separator.
     * For more control over the split use the StrTokenizer class.
```

<br/>

## 예제

```java
import org.apache.commons.lang3.StringUtils;

import java.util.Arrays;

public class SplitCompare {

    private static final String DELIMITER = ":";

    public static void compare(){
        String str = "1:2:3::5";
        String[] splitedByStringUtils = StringUtils.split(str, DELIMITER);
        String[] splitedByString = str.split(DELIMITER);

        System.out.println(Arrays.toString(splitedByString));
        System.out.println(Arrays.toString(splitedByStringUtils));
    }

    public static void main(String args[]){
        compare();
    }
}
```

결과는 다음과 같다. 
```java
[1, 2, 3, , 5]
[1, 2, 3, 5]
```
