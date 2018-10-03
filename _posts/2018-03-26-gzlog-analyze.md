---
layout: post
title:  "gz 로그 파일 커맨드 라인으로 분석하기"
date:   2018-03-26 00:00:59
group: 개발일지
tags:   linux 
---

## 로그 파일 분당 특정 로그가 나타나는 횟수 분석
어플리케이션에서 특정 동작이 단위 시간당 얼마나 일어나는지 확인할 필요가 있어서 사용했던 커맨드 라인을 정리했다.
로그 내용은 아래와 같고 분당 Working 갯수가 얼마나 되는지 확인해야 했다.

```
// application.log
2018-03-27 00:01:04,636 INFO  c.d.e.a.i.ExampleService - worker id=1724
2018-03-27 00:01:04,636 INFO  c.d.e.b.jobs.worker.ExampleWorker - Working
2018-03-27 00:01:04,636 INFO  c.d.e.b.jobs.worker.ExampleWorker - Working
```

<br/>

### gz 아닌 log 분석
Working을 grep 한 후 시간/분에 대해서 카운트 한다. 

```
tail -n 1000000 application.log | grep "Working" | cut -d ':' -f1,2 | uniq -c
   7252 2018-03-27 20:03
  10000 2018-03-27 20:04
  10000 2018-03-27 20:05
  10000 2018-03-27 20:06
  10000 2018-03-27 20:07
  10000 2018-03-27 20:08
  10000 2018-03-27 20:09
  10000 2018-03-27 20:10
  10000 2018-03-27 20:11
  10000 2018-03-27 20:12
  10000 2018-03-27 20:13
  10000 2018-03-27 20:14
  10000 2018-03-27 20:15
```

<br/>

### gz으로 압축된 log 분석
날짜가 지나면 압축되도록 설정해 두었다면 gz 파일을 열어봐야 하는데 귀찮은 일이다. gz 파일에 대해서 tail은 사용할 수 없고 zcat으로 일부만 확인할 수도 없댜. 
다행히 gzip에 좋은 옵션이 많았다. tail 앞에 gzip -cd 를 추가해 동일한 결과를 확인할 수 있다.
```
// man gzip
... 
      -c --stdout --to-stdout
              Write  output  on standard output; keep original files unchanged.  If there are several input files, the output consists of a sequence of independently compressed
              members. To obtain better compression, concatenate all input files before compressing them.

      -d --decompress --uncompress
              Decompress.
...
```


```shell
gzip -cd 2018-03-26.application.log.gz | head -n 1000000 application.log | grep "Working" | cut -d ':' -f1,2 | uniq -c
  10000 2018-03-26 00:00
  10000 2018-03-26 00:10
  10000 2018-03-26 00:13
   3261 2018-03-26 00:14
   6739 2018-03-26 00:15
   9275 2018-03-26 00:16
    725 2018-03-26 00:17
  10000 2018-03-26 00:18
  10000 2018-03-26 00:20
   2314 2018-03-26 00:21
   7686 2018-03-26 00:22
   8487 2018-03-26 00:23
   1513 2018-03-26 00:24
```

다만 gzip으로 head는 빠르게 확인 가능하지만, tail 만 확인할 수는 없다. 압축 알고리즘은 스트림에서 작동하며 스트림의 내용이 특정 지점 이전에 무엇인지 알지 못하면 해당 지점에서 압축 해제를 할 수 없다고 한다. <br/>
(참고 : https://stackoverflow.com/questions/1183001/how-can-i-tail-a-zipped-file-without-reading-its-entire-contents)

<br/>
