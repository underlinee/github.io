---
layout: post
title:  "java 스레드 덤프 분석 - 병목 지점 확인하기"
date:   2018-03-26 00:43:59
categories: DevLog
tags:   java thread dump bottleneck 
---


## 스레드 덤프하기 
batch 어플리케이션을 실행하는데 실행 속도가 점점 느려져서 스레드 덤프를 통해 병목 구간을 확인해봤다. 어플리케이션은 배치 작업으로 DB에서 데이터를 가져온 후 Queue에 넣는다. 어플리케이션 실행 후 로그 확인시 어플리케이션의 동작이 이루어지지 않은 것으로 보이는 (병목으로 추측되는!) 기간이 2분 가량 지속되었다. 

#### pid 확인하기 
```
ps -ef | grep java 
```

<br/>

#### jstack 이용하여 스레드 덤프
병목 구간으로 추측되는 부분에서 스레드 덤프를 한다. 한 번으로 정확히 알 수 없기 때문에 여러번 스레드 덤프를 통해 결과를 비교하자. 
```
jstack -l pid >> threaddumps.log
```

<br/>

#### 스레드 덤프 내용 분석
어플리케이션이 Queue에서 데이터를 소비하는 Worker Thread는  WAITING 상태였고, DB에서 데이터를 가져오는 Repository 객체는 RUNNABLE인 상태였다. 
ExampleRepository.updateData() 메서드에서 병목인 것을 확인할 수 있다. 

자세한 분석 방법을 알고 싶다면 <a href="http://d2.naver.com/helloworld/10963">스레드 덤프 분석하기</a>를 참고하자.

```
// Worker Thread
"ajp-bio-8009-exec-77" #597 daemon prio=5 os_prio=0 tid=0x00007f901c00e000 nid=0x7965 waiting on condition [0x00007f90c91aa000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000abdc3cd0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)
```

```
// RUNNABLE인 Repository 객체
"ses-quartz-scheduler_Worker-5" #16 prio=5 os_prio=0 tid=0x00007faaec6ff800 nid=0x1dcc runnable [0x00007faaa55de000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
        at java.net.SocketInputStream.read(SocketInputStream.java:171)
        at java.net.SocketInputStream.read(SocketInputStream.java:141)
        at com.mysql.jdbc.util.ReadAheadInputStream.fill(ReadAheadInputStream.java:112)
        at com.mysql.jdbc.util.ReadAheadInputStream.readFromUnderlyingStreamIfNecessary(ReadAheadInputStream.java:159)
        at com.mysql.jdbc.util.ReadAheadInputStream.read(ReadAheadInputStream.java:187)
        - locked <0x00000006ee4400e0> (a com.mysql.jdbc.util.ReadAheadInputStream)
        at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:3158)
        at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3615)
        at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3604)
        at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:4149)
        at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2615)
        at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2776)
        at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2840)
        - locked <0x00000006ee437d78> (a com.mysql.jdbc.JDBC4Connection)
        at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:2082)
        - locked <0x00000006ee437d78> (a com.mysql.jdbc.JDBC4Connection)
        at com.mysql.jdbc.PreparedStatement.execute(PreparedStatement.java:1302)
        - locked <0x00000006ee437d78> (a com.mysql.jdbc.JDBC4Connection)
        at com.mchange.v2.c3p0.impl.NewProxyPreparedStatement.execute(NewProxyPreparedStatement.java:989)
        at com.ibatis.sqlmap.engine.execution.SqlExecutor.executeUpdate(SqlExecutor.java:80)
        at com.ibatis.sqlmap.engine.mapping.statement.MappedStatement.sqlExecuteUpdate(MappedStatement.java:216)
        at com.ibatis.sqlmap.engine.mapping.statement.MappedStatement.executeUpdate(MappedStatement.java:94)
        at com.ibatis.sqlmap.engine.impl.SqlMapExecutorDelegate.update(SqlMapExecutorDelegate.java:457)
        at com.ibatis.sqlmap.engine.impl.SqlMapSessionImpl.update(SqlMapSessionImpl.java:90)
        at org.springframework.orm.ibatis.SqlMapClientTemplate$9.doInSqlMapClient(SqlMapClientTemplate.java:383)
        at org.springframework.orm.ibatis.SqlMapClientTemplate$9.doInSqlMapClient(SqlMapClientTemplate.java:381)
        at org.springframework.orm.ibatis.SqlMapClientTemplate.execute(SqlMapClientTemplate.java:203)
        at org.springframework.orm.ibatis.SqlMapClientTemplate.update(SqlMapClientTemplate.java:381)
        at com.examples.application.repository.ExampleRepository.updateData(ExampleRepository.java:48)
```

<br/>

#### 병목 구간 없애기
ExampleRepository.updateData() 는 배치 작업을 시작하기 전 데이터의 플래그를 변경하는 메서드인데, WHERE IN 절에 크기가 너무 큰 리스트가 들어가면서 성능상 문제가 생긴 것으로 확인했다. 
```
// dataIds는 size가 10000인 list
repository.updateData(dataIds, STATUS.WORKING);
```

dataIds를 나누어 처리하기 위해 Guava 의 Iterables를 사용했다. 
```
Iterable<List<String>> patitionedDataIds = Iterables.partition(dataIds, 500);
for (List<String> dataIds : patitionedDataIds) {
    repository.updateData(dataIds, STATUS.WORKING);
}
```







