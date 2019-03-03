---
title: '[PHP] Session Lock 문제'
date: 2019-2-10 16:21:13
category: 'php'
---
PHP는 기본적으로 세션 데이터를 파일에 저장한다. (일반적으로 /tmp 에 저장됨)

PHP에서 세션이 처리되는 순서를 나타내면 다음과 같다.

    1. session_start() - 세션파일 생성 및 잠금
    2. SQL, API 호출 등 기능 수행 - 세션의 파일 잠금 유지
    3. 스크립트 종료 - 세션파일 잠금 제거

이때 파일은 PHP가 세션을 시작할 때 잠금을 설정하여 다른 프로세스가 데이터를 변경하는 `race condition`을  방지한다.

그런데 여기서 문제점이 발생한다. 동시에 요청이 들어왔을 경우 세션에 대한 접근은 동시에 불가능 하기 때문에 최초 요청의 스크립트가 종료될 때 까지 다른 요청들은 대기해야된다.

PHP 세션에서 2개의 요청을 처리하는 순서를 다시 나타내보면 다음과 같다.

    1. A-session_start() - 세션파일 생성 및 잠금
    2. B-session_start() - 잠금이 해제될 때 까지 대기
    3. A-SQL, API 호출 등 기능 수행 - 세션의 파일 잠금 유지
    4. A-스크립트 종료 - 세션파일 잠금 제거
    5. B-SQL API 호출 등 기능 수행 - 세션의 파일 잠금 유지
    6. B-스크립트 종료 - 세션파일 잠금 제거

요청에 대한 수행시간이 짧다면 크게 문제될 건 없지만 만약 길다면 사용자는 페이지 새로고침을 시도할 것이고 반복적인 요청으로 서버의 부하로 이어질 수 있다.

그래서 PHP에는 session_write_close()라는 함수를 제공한다. 함수 이름과 같이 데이터를 쓰고 파일을 닫음으로써 잠금을 제거하는 함수이다.

```php{9}
...
session_start ();

$ _SESSION['userid'] = 'DANDI';
$ _SESSION['userno'] = '232321';

session_write_close ();

이후에 $_SESSION이 조작되면 변경사항이 저장되지 않는다.
```

PHP 7 부터는 세션을 시작할 때 옵션을 지정할 수 있다.


```php{5}
session_start ([
        'read_and_close'=> true,
]));

세션 데이터를 읽고 다른 스크립트가 차단되지 않도록 즉시 잠금을 해제한다.
```

참고로 해당 사이트에 접속하면 session lock을 간단히 확인할 수 있다.

[https://demo.ma.ttias.be/demo-php-blocking-sessions/](https://demo.ma.ttias.be/demo-php-blocking-sessions/)
