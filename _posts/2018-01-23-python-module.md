---
layout: post
title:  "ModuleNotFoundError: No module named 해결하기"
date:   2018-01-22 20:13:59
group: 개발일지
tags:   python  
---

pycharm 으로 run 시에는 잘 동작하는 스크립트가 커맨드 라인으로 실행시 다음과 같은 에러와 함께 실행에 실패했다. 
```
Traceback (most recent call last):
  File "/Users/MyName/PycharmProjects/myproject/src/myproject/myscript.py", line 8, in <module>
    from myproject.package import script, script, script
ModuleNotFoundError: No module named 'myproject'
```
<br/> 

## sys.path 설정하기 

파이썬은 모듈을 읽어 들일 때 sys.path에 있는 모든 경로를 검색한다. 따라서 import에 문제가 있을 때는 sys.path부터 확인하자. 

```python
# import sys; print(sys.path)
['/Users/MyName/PycharmProjects/myproject/src/myproject'...]
```

동작하지 않던 스크립트에 다음 코드를 추가하자 커맨드 라인을 통해서도 스크립트가 잘 동작한다. 
```python
sys.path.append(os.path.join(os.path.dirname(__file__), '..'))
```

필요한 모듈을 import하기 위해서는 sys.path가 src 로 설정되어야하나 myproject로 설정되어 import에 실패했던 것. pycharm에서는 run 시에 디폴트로 content root와 source root를 PYTHONPATH에 추가하기 때문에 정상적으로 동작했다. 


<a href="//underlinee.github.io/assets/img/20180123-1.png" data-lightbox="falcon9-large">
  <img src="//underlinee.github.io/assets/img/20180123-1.png"/>
</a>


sys.path 를 확인해보면 다음과 같다. 

```python
# import sys; print(sys.path)
['/Users/MyName/PycharmProjects/myproject/src/myproject', ...'/Users/MyName/PycharmProjects/myproject/src/myproject/..']
```

<br/> 

## entry point를 프로젝트 루트로 변경하자

하지만 모든 실행파일에 PYTHONPATH를 지정할 수는 없다. main으로 실행되는 파일의 디렉터리는 자동으로 pythonpath에 저장되기 때문에 entry 포인트를 프로젝트의 루트에 두면 import 문제가 해결된다. 불가피하게 루트 아래에 있는 파일 외의 대해서 import가 필요하다면 아래와 같이 구성한다. 

```python
# setup.py
import sys

def load():
    paths = ['/path1/','/path2/','/path3/']
    for p in path:
        sys.path.insert(0, p)

# entrypoint.py
from setup import load
load()
```
<br/> 
