---
layout: post
title:  "크롤링 해 온 html에 원하는 값이 없다면"
date:   2018-01-28 20:43:59
group: 개발일지
tags:   beautifulsoup crawling 
---

웹 사이트에서 잘 표시되는 컨텐츠를 크롤링 해왔을 때 파싱한 html에서 원하는 값을 찾을 수 없는 경우가 있다. 이런 상황에서 의심해 볼 수 있는 케이스를 정리했다. 

<br/>

### Ajax 를 통해 컨텐츠를 가져오는 경우
페이지에 특정 컨텐츠를 Ajax 를 통해 화면에 뿌려주는 경우 브라우저에서 접근하는 경로로는 해당 컨텐츠를 이용할 수 없다. 이런 경우의 첫번째 해결법은 <a href="http://selenium-python.readthedocs.io/">Selenium</a>을 사용해 브라우저로 해당 컨텐츠에 접근하는 방법이 있다. 편한 방법이지만 브라우저를 사용할 수 있는 환경에서만 가능한 방법이고 크롤링할때마다 브라우저가 켜지는 등 불편한 점이 많다. 두번째 방법은 직접 Ajax로 요청하는 Request를 찾아내서 다시 해당 url로 접근하는 방법이다. 해당 웹 사이트에서 막아 놓았을 가능성도 있지만 그렇지 않다면 Selenium을 사용하는 방법보다 깔끔하고 활용도가 높다. Ajax 콜을 확인하는 방법은 Chrome의 개발자 도구를 통해 네트워크 요청이 어떤 것이 있는지 확인한 후 하나하나 확인하는 방법이 그나마 가장 쉬운 방법인 것 같다. 타입이 xhr인 요청을 잘 살펴보자. 그리고 원하는 요청을 찾았다고 하더라도 request 헤더를 조작해야 요청이 가능한 경우도 있다. 

<a href="//underlinee.github.io/assets/img/20180129-1.png" data-lightbox="falcon9-large">
  <img src="//underlinee.github.io/assets/img/20180129-1.png"/>
</a>

<br/> 

### html이 유효하지 않은 경우 
브라우저는 생각보다 똑똑하다. 사용자가 많은 메이저한 웹 사이트라도 html이 깨지는 경우가 꽤 있고, 브라우저는 잘못된 html을 눈치채지 못하게 수정해서 올바른 형태로 보여준다. 이런 html을 파싱해서 컨텐츠를 가져오려고 하는 경우 파싱에 실패하면서 특정 데이터만 확인 못할 수 있다. 아래는 </div> 가 잘못 닫혀진 경우로 이런 경우 BeautifulSoup 의 html.parser를 통해 파싱하려고 하는 경우 값이 누락된다. 

```html
<div class="texts">
        <table border="0" cellpadding="0" cellspacing="0" width="100%"><tbody><tr><td valign="top">
<div>���簡 ������ �� �����ϴ�. ���� �������� �����ؾ� �ϴ� �ΰ��� ���԰� ���Կ� ���� ��� �����̴�. ���ӵ� �� ����� ������ �ߴ� ���� ��ﳭ��. �⸧������ �԰������ ���� ��¿ �� ���� ��û�� �� ���ε� �װ� �׸� ���� �˳İ�, �ᱹ ���ڵ��� ����ȸ�縦 ���ؼ� �ϴ� ûŹ���� �ƴϳĸ鼭 ������ �� �� �ִ� ����� ���� ����İ� ����� ���ߴ�. ������ ����� �����ڸ� �״� ��κ��� ����鿡 ���� �����ߴ�. ���̿��� ���� ��¡�� �Ķ����� �Ͼ���� �����ϴ� �������� �� �ڵ����� �� �뾿�̳� ������ �־���, ���� �޵� �� �ٴ� ���� ����Ʈ���� ��� �־���.
<br/>�ݸ� �� ������ �̿��ߴ� ������� ��¥ ���ε��̾���. �׵��� ������ �� ��� ���� ��ŷ ���̷� �극��ũ ������ ���� ���� �ִ�. �׷��� ���ӵ��ο��� �극��ũ�� ��Ҵµ� ����� �� ���⸸ �ϰ� ������ ���� �ʾ� ������ ���ָ� �ϴ� ��ü�� �Լ�ó�� ������ �������� �ƺ��� ����, �������� ������ �Ƴ��� �ٽô� �����鿡�� ���ư��� ������ ���� �ִ�. ������ �� ������ �׷� �� ���� �ƹ����� ������� ����̾���. ���� ��ȸ�� �������� �и��� �ڽ��� ���ӵǴ� ���� ���ո��ϰ� �Ұ����� ���̾���. �ڰ� ġ�� �ɷ��� ���� �� ���� �����̿� ������ �ִ� �÷��õ� �������̿�����, �ΰ��� ������ �ϴ� �ּ����� ������ å�Ӱ��� ������ ������� �׾߸��� ������ �������̿���.
<br/>_ ����¼�� �����, ������ �ɷ��� ���� �׵项 �߿���</div>
</td>
</tr></tbody></table>
```


이런 케이스라면 parsing library를 다른 것으로 바꿔보자. 엄격하지 않은 라이브러리로 교체하는 경우 문제가 있는 html이라도 가져올 수 있다. 
python Beutiful Soup을 사용한다면 html5lib가 엄격하지 않는 듯 하니 아래와 같이 적용해서 다시 시도해보자.

```python
soup = BeautifulSoup(response.data, 'html5lib')
```

<br/> 