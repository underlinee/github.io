---
layout: post
title:  "구글 자연어 처리 API 사용하기"
date:   2018-01-20 18:43:59
categories: Devlog
tags:	python 자연어처리 
---


## CLOUD NATURAL LANGUAGE API
한국어 주제어 추출 오픈 소스를 찾다가 마땅한 걸 찾을 수가 없었다. 많이 사용하는 자연어 처리 라이브러리는 https://spacy.io 가 있으나 아직 한국어를 지원하지 않는다. konlpy를 사용하면 형태소 분석 정도는 문제 없으나 구문 분석에 사용할 만한 기능은 없는것 같다. 좀 더 찾아보니 구글에 한국어를 지원하는 자연어 처리 api가 있다. https://cloud.google.com/natural-language/ 무료로 0~5,000 단위/월 을 사용할 수 있고 생각보다 품질도 괜찮다.

<a href="//underlinee.github.io/assets/20180121-1.png" data-lightbox="falcon9-large">
  <img src="//underlinee.github.io/assets/20180121-1.png"/>
</a>

-------------
#### API key 발급
https://cloud.google.com/docs/authentication/getting-started 에서 Create service account key 를 클릭해서 발급 받자. 

<a href="//underlinee.github.io/assets/20180121-2.png" data-lightbox="falcon9-large">
  <img src="//underlinee.github.io/assets/20180121-2.png"/>
</a>

#### Python 으로 API 사용
필요한 google 패키지 설치 후 사용한다. 다운 받은 api key 경로를 환경 변수로 설정하여 사용할 수도 있지만 따로 환경 변수를 관리하기가 불편해서 google-auth에서 제공하는 credentials 객체를 사용하기로 했다. credentials 객체 사용은 환경변수와 마찬가지로 api key 파일 경로만 지정해주면 된다. 
http://google-auth.readthedocs.io/en/latest/reference/google.oauth2.service_account.html
LanguageServiceClient 는 구글 자연어 처리 api 용 클라이언트 라이브러리다.
https://cloud.google.com/natural-language/docs/reference/libraries


```
from google.cloudimportlanguage
from google.cloud.languageimportenums
from google.cloud.languageimporttypes
from google.oauth2importservice_account

credentials_path="/Users/account/Downloads/myservice.json"
credentials=service_account.Credentials.from_service_account_file(credentials_path)
client=language.LanguageServiceClient(credentials=credentials)

document=types.Document(
content=underline,
type=enums.Document.Type.PLAIN_TEXT)

response=client.analyze_entities(document=document,encoding_type=enums.EncodingType.UTF8)

```