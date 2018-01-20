---
layout: post
title:  "logstash 6.1.2 모니터링을 위한 x-pack 세팅"
date:   2018-01-19 16:43:59
categories: Devlog
tags:	logstash x-pack
---


## elasticsearch 세팅

-------------
#### elasticsearch 설치 & x-pack 설치
```
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.2.tar.gz

// 압축을 푼다

bin/elasticsearch-plugin install x-pack
```


#### elasticsearch.yml 설정
x-pack security는 적용 안하고 바로 사용하기 위해 xpack.security.enabled 는 false로 설정
```
network.host: "example-monitoring.net"
http.port: 9200
xpack.security.enabled: false
```

#### elasticsearch 실행
```
bin/elasticsearch
```
<br/> 
## kibana 세팅
-------------

#### kibana 설치 & x-p
```
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-6.1.2-darwin-x86_64.tar.gz

// 압축을 푼다

bin/kibana-plugin install x-pack

```

#### kibana.yml 설정
```
server.host: example-monitoring.net
elasticsearch.url: "example-monitoring.net:9200"
xpack.security.enabled: false
```

#### kibana 실행
```
bin/kibana
```
<br/> 

## logstash 세팅
-------------

#### logstash 설치 & x-pack 설치
```
curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-6.1.2.tar.gz

\\ 압축을 푼다

bin/logstash-plugin install x-pack

```

#### logstash.yml 설정
```
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.url: example-monitoring.net:9200
```

#### logstash 실행
```
bin/logstash -f config/logstash.conf
```








