---
title: '[Javascript] ES6 - 향상된 객체 리터럴(Enhanced Object Literal)'
date: 2019-03-10 22:00:00
category: 'javascript'
---
###1. 프로퍼티 축약 문법
프로퍼티 값으로 변수를 사용하는 경우, 프로퍼티 이름을 생략할 수 있다. 
이때 값이 할당된 변수의 이름으로 자동 입력된다.

```javascript{4}
let name = 'wooddy'
let age = 28
    
const user = {name, age}
    
console.log(user) //{name: 'woody', age: 28}
```
<br/>

###2. 프로퍼티 키 동적 생성 (Computed property name)
ES5는 객체 리터럴로 먼저 선언후 이후에 프로퍼티 키로 변수를 사용할 수 있다.
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
