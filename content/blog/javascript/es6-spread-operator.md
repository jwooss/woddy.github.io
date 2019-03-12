---
title: '[Javascript] ES6 - 전개 연산자(Spread Operator)'
date: 2019-03-12 23:00:00
category: 'javascript'
---
전개 연산자는 객체, 배열의 값들을 각각 다른 객체, 배열
로 복제하거나 옮길 때 사용한다.

전개 연산자를 사용하는 방법은 `...`를 붙이면 된다.

```javascript{7,11}
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

모던 자바스크립트 개발에서는 많이 쓰이고 있으며 코드를 줄여주고 
더 명확하게 해주는 이점이 있다. 
 
여러 케이스가 있겠지만 매개변수로 사용될 때도 유용한데 곱셈을 연산하는 함수로
예를 들어보겠다. 

먼저 전개 연산자를 사용하지 않고 곱셈연산을 하는 함수를 작성해보면  

```javascript{2}
function multi() {
  let args = Array.from(arguments)// 유사배열 변환
  let result = args.reduce(function(acc, val) {
    return acc * val
  })
}

multi(1, 2, 3) // 6
multi(2, 4, 6) // 12
```

여기서 매개변수를 array로 변환하기 위한 코드가 있다.
이 부분을 전개 연산자를 이용하면 한 줄의 코드를 줄이고 더 명확하게 작성할 수 있다.

```javascript{1}
function multi(...args) {
  let result = args.reduce(function(acc, val) {
    return acc * val
  })
}

multi(1, 2, 3) // 6
multi(2, 4, 6) // 12
```
