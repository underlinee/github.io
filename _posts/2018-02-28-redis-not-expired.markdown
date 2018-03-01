---
layout: post
title:  "Redis TTL -1 인 값이 생기는 버그 해결"
date:   2018-02-28 18:43:59
categories: Devlog
tags:	Redis 버그 expire ttl 
---


## Redis expire 되지 않는 값
Redis 에 incrBy 으로 새 값을 추가 후 expire 시간을 세팅하고, 해당 값들에 대해 incrby를 통해 카운트 집계하는 로직에서 expire 되지 않는 값들이 발견되었다. expire 되지 않는 값들은 ttl 조회시 -1로 조회가 된다. 모든 값을 생성 후 expire 를 set 했지만 expire가 없는 케이스가 발견된 원인은, expire 된 시점에 incryby 했기 때문이다. redis의 incryby는 key가 없는 경우 해당 키를 자동으로 생성해서 incrby를 한다. 이런 경우 expire가 없는 채로 새 값이 생성된 것이다. 

<br/> 

## 해결하기 
- WATCH command
Redis 트랜잭션 관련 커맨드인 WATCH는 해당 값을 WATCH하고 있어도 expire 된 부분에 대해서 트랜잭션을 보장해주지 않는다. 따라서 expire에 대해서 트랜잭션을 적용할 수 있는 명령어를 찾지 못했다. 
- INCRBY를 한 값이 신규로 set 되었다면 expire를 새로 지정
expire 된 값에 대해 incrby를 실행했더라도 그 값을 보고 신규로 set 된 값이 것을 확인한 경우 expire를 새로 지정해준다. 이미 존재하는 key에 대해 incrby를 수행했다면 따로 처리가 필요하지 않다. 

```
jedis = redisPool.getResource();

String value= jedis.get(key);
if( null == value) {
	result = jedis.incrBy(key, increment);
    jedis.expire(key, expireSeconds);
} else {
    result = jedis.incrBy(key, increment);
    if( isNewValue(result)) {
        jedis.expire(key, expireSeconds);
	}
}
```

<br/> 

## 참고
https://redis.io/topics/transactions
<br/> 
https://redis.io/commands/incrby