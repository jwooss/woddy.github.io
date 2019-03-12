---
title: '[Javascript] ES6 - 전개 연산자'
date: 2019-03-12 23:00:00
category: 'javascript'
---
전개 연산자는 객체, 배열의 값들을 각각 다른 객체, 배열
로 복제하거나 옮길 때 사용한다

전개 연산자를 사용하는 방법은 `...`를 붙이면 된다.

```javascript
// object
let user = {
  name: 'wooddy',
  age: 28,
}

var newUser = {...user} // {name: 'wooddy', age: 28}

// array
let users = ['wooddy1', 'wooddy2', 'wooddy3']
let newUsers = [...users] // ['wooddy1', 'wooddy2', 'wooddy3']
```
<br/>

보통 매개변수로 사용될 때 유용한데 
전개 연산자를 사용하지 않고 곱셈연산을 하는 함수를 작성해보면  

```javascript
function multi() {
  let args = Array.from(arguments)// 유사배열 변환
  let result = args.reduce(function(acc, val) {
    return acc * val
  })
}

multi(1, 2, 3) // 6
multi(2, 4, 6) // 12
```

여기서 전개 연산자를 이용하면 코드를 줄이고 더 쉽게 작성할 수 있다.

```javascript
function multi(...args) {
  let result = args.reduce(function(acc, val) {
    return acc * val
  })
}

multi(1, 2, 3) // 6
multi(2, 4, 6) // 12
```

한 줄의 코드가 제거되었고 읽기 쉽고 명확한 코드로 바뀌었다.
