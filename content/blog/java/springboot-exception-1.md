---
title: '[SpringBoot] 예외처리를 어떻게 할것인가 (1)'
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

이런 부분들을 개선하기 위해서는 Controller의 반복적인 try ~ catch를 제거하고 Exception에 대한 Handler를 구현하여 예외처리를 공통화 시키면 개선할 수 있지 않을까 생각이 든다.
