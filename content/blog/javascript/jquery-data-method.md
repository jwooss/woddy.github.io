---
title: '[Javascript] jQuery data() 메소드 흔한 실수'
date: 2018-12-23 16:21:13
category: 'javascript'
---
웹 개발시 jQuery를 사용하다 보면 data\() 메소드를 많이 사용하게 되는데,  이 때 메소드가 정확히 어떤식으로 동작하는지 모르고 사용하다 보면 다음과 같은 상황을 만나게 된다.

```html{7,9}
<html>
	<div id="search_list" data-key="userid"></div>
</html>

<script>
var $ele = $('#search_list')
console.log($ele.data('key')) // userid
$ele.attr('data-key', 'userno')
console.log($ele.data('key')) // userid
</script>
```

div element에 data-key attribute로 userid가 셋팅되어 있고, data\(\) 메소드로 key를 가져오면 userid라는 결과가 나온다. 

이 후 attr\() 메소드로 data-key atrribute의 값을 변경하고 data\(\) 메소드로 값을 가져오면 변경된 값이 아닌 처음에 셋팅해둔 userid라는 값이 나온다.

data\() 메소드가 html element에 있는 data-* attribute를 당연히 가져오는 건 줄만 알고 사용하다 보면 흔히 마주치는 문제이다. 

공식 문서에 살보면 다음과 같은 설명이 있다.

> Since jQuery 1.4.3, data-* attributes are used to initialize jQuery data. An element's data-* attributes are retrieved the first time the data\(\) method is invoked upon it, and then are no longer accessed or mutated \(all values are stored internally by jQuery).

data-* attributes는 최초 값을 초기화 할 때만 사용되고, 검색된 다음 더 이상 접근하거나 변경되지 않는다 그리고 값은 jQuery 내부적으로 저장된다는 설명이다. 

그럼 검색하기 전에 변경하면 어떻게 될까?

```html{8}
<html>
	<div id="search_list" data-key="userid"></div>
</html>

<script>
var $ele = $('#search_list')
$ele.attr('data-key', 'userno')
console.log($ele.data('key')) // userno
</script>
```

결과는 data\() 메소드가 호출되기 전에 data-* attributes가 변경한다면 변경된 후의 값을 가져오는 것을 볼 수 있다.

추가로 jQuery data\() 메소드에서 어떻게 값을 처리하는지 궁금해서 코드를 살펴보았다. 

이 부분은 data\(\) 메소드를 호출할 때의 일부분이며 arguments.length가 1일 때 retrun 하는 function이다. \(getter\)

```javascript
function dataAttr(elem, key, data) {
  // data가 없고 DOM 노드이면 data-* attributes를 확인
  if (data === undefined && elem.nodeType === 1) {
    var name = 'data-' + key.replace(rmultiDash, '-$1').toLowerCase()
    
    data = elem.getAttribute(name)
    
    if (typeof data === 'string') {
    	try { 
    	  data = data === 'true' ? true :
    	  	data === 'false' ? false :
    	  		data === 'null' ? null : 
    	  			+data + '' === data ? +data : rbrace.test(data) ? jQuery.parseJSON(data) : data
    	} catch 	 (e) {
    	}
    }
  	// 데이터를 캐쉬로 셋팅
  	// 이후 attr-* attributes를 변경해도 이전에 캐쉬되었던 값을 불러옴
  	jQuery.data(elem, key, data)
  			
  } else {
    data = undefined
  }
  
  return data
}
```

###### 참고
* [https://api.jquery.com/data/](https://api.jquery.com/data/)
