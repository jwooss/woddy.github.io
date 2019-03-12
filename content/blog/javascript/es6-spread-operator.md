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

전개 연산자는 매개변수로 사용될 때 유용한데 전개 연산자를 사용하기 이전에는

```javascript{9,10}
var users = [
    {name: 'wooddy1', age: 28},
    {name: 'wooddy2', age: 28},
    {name: 'wooddy3', age: 28},
    {name: 'wooddy4', age: 28},
]
    
var obj = users.map((v, k) => {
	var user = {}
	obj[v.name] = v.age
	return obj
})
    
console.log(obj) //[{wooddy1: 28}, {wooddy2: 28}, {wooddy3: 28}, {wooddy4: 28}] 
```

ES6에서는 객체 리터럴 선언시 동적으로 생성 가능하다.

```javascript{10}
let users = [
	{name: 'wooddy1', age: 28},
	{name: 'wooddy2', age: 28},
	{name: 'wooddy3', age: 28},
	{name: 'wooddy4', age: 28},
]

let obj = users.map((v, k) => {
	return {
		[v.name]: v.age
	}
})
console.log(obj) //[{wooddy1: 28}, {wooddy2: 28}, {wooddy3: 28}, {wooddy4: 28}]
```
<br/>

###3. 메소드 축약 (function 예약어 생략)
ES5에서 프로퍼티 값으로 함수 선언식을 사용한다.
```javascript{3}
var user = {
	name: 'wooddy',
	getName: function() {
		return this.name
	}
}

user.getName() // 'woody'
```

ES6에서는  function 예약어 생략하여 사용할 수 있다.
```javascript{3}
const obj = {
    name: 'wooddy',
   	getName() {
  		return this.name
   	}
}
    
user.getName() // 'woody'
```
