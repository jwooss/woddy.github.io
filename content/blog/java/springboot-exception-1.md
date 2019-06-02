---
title: '[SpringBoot] @Controller 와 @RestController 분리에 대한 고민'
date: 2019-5-29 22:19:00
category: 'java'
---

PHP 기반의 서비스를 운영하다가 최근에 SpringBoot 기반의 프로젝트를 진행중이다.

새로운 서비스를 만드는 만큼 PHP 레거시에서 아쉬웠던 부분을 프로젝트 진행하면서 개선하려고 노력중인데 그중에서 한가지가 ``Exception에 대한 처리``이다.

```php{5,9}
<?php
    class UserController {
    	public function create() {
    		try {
    				// ReqeustValidException
    				$userId = $this->P("userId");
    				$userPwd = $this->P("userPwd");
    
    				// ServiceException, ModelException...
    				$userService->create($userId, $userPwd);
    				
    				...
    		} catch (ReqeustValidException $e) {
    				...
    		} catch (ServiceException $e) {
    			Log::log($e->getMessage());
    				...
    		} catch (ModelException $e) {
    			Log::info($e->getMessage());
    				...
    		} catch (Exception $e) {
    			Log::error($e->getMessage());
    				...
    		}
    	}
    }
 ```

PHP 서비스에서 Controller에 해당하는 부분은 대략적으로 위와 같은 코드가 계속 반복된다. 

발생할 수 있는 사용자 정의 Exception에 대해서는 Controller에 try ~ catch 구문으로 모두 처리하고 각 Exception에 해당하는 로그 레벨 설정 등 후속처리를 진행한다.

간단한 기능이라면 큰 불편함은 없겠지만, 기능이 많아지면 중복 코드가 점점 늘어날 것이고 누락된 코드라던지 ``human error``를 발생시키게 된다.
(~~실제로도 많음)~~

그중에서 잦은 실수가 Exception 처리로직을 복사해서 그대로 사용하다가 json을 반환하는 컨트롤러에서 view를 반환하여 사용자에게 적절치 못한 메시지를 보여주는 경우다.

이런 부분들을 개선하기 위해서는 Controller의 반복적인 try ~ catch를 제거하고 Exception에 대한 공통 Handler를 구현하면 개선할 수 있지 않을까 생각이 들었다.

아니나 다를까 이미 진행중인 SpringBoot 기반의 프로젝트는 생각했던 대로 처리되고 있었는데

```java
@Controller
public class ExceptionController {
	@ExceptionHandler(Exception.class)
	publick String exceptionHandler(Exception exception) {
		try {
			... // exception 처리 (알림발송 등)
		} catch (Exception e) {
			... // exception 처리중 실패
		}
		
		return "error";
	}
}

@Controller
publick class UserController extends ExceptionController {

	@PostMapping("/create")
	public String create(Model model) {
		... 
		
		return "view";
	}
}
 ```
 
User Controller 에서 ExceptionController 를 상속받고 exception이 발생했을 경우 공통으로 처리하고 있으므로
대략 생각했던 흐름대로 Controller 에서 반복적인 코드가 없어지고 공통으로 exception이 처리되고 있다.

그런데 만약 create() method에 @ResponseBody 어노테이션이 있다면 exception이 제대로 처리될까?

```java
@Controller
publick class UserController extends ExceptionController {

	@ResponseBody
	@PostMapping("/create")
	public JsonResult create(Model model) {
		... 
		
		return new JsonResult()";
	}
}
 ```
 
왜냐하면 exceptionHandler() 는 view를 반환하고 있으므로 개발자가 의도한 response를 못 내려 줄 것이다.

지금 프로젝트 구조라면 얼마든지 일어날 수 있다고 생각이 들어 내린 결론은 controller를 ```@RestController``` 와 ```@Controller``` 로 분리하고 그에 따라서 exception 처리를 하는 ``handler도 분리`` 하는 것이다.

코드로 표현하면 대략적으로 다음과 같다.

```java{1,2,12,13}
@RestController
publick class UserRestController extends ExceptionRestController {

	@PostMapping("/create")
	public JsonResult create(Model model) {
		... 
		
		return new JsonResult()";
	}
}

@Controller
publick class UserController extends ExceptionController {

	@PostMapping("/view")
	public String view(Model model) {
		... 
		
		return "view";
	}
}
 ```
 
프로젝트 구조는 이렇게 바꾸면 될 것 같다.

```
// 기존
controller
	└ UserController.java
	
// 변경
controller
	└ UserController.java
	api
		└ UserRestController.java
```

controller 가 분리됨에 따라 exceptionHandler도 분리되어 역할이 명확해지고 사용자에게 더 올바른 응답을 내려줄 수 있다.
 
근데 이렇게 하면 또 파일이 많아져서 복잡해지는건 아닌가 걱정이 들기도 하고\.\.\.

모든 개발이 트레이드 오프인 것처럼 정답은 없는 것 같다.

Java와 Spring을 접한지 3주 정도 됐기 때문에 고민에 대해서 확신이 없는데 프로젝트가 끝날 무렵에는 이런 고민에 대해서 확신이 생겼으면 좋겠다.




