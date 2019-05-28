---
title: '[MyBatis] insert query 수행 후 key 반환하기'
date: 2019-5-28 23:14:00
category: 'java'
---

요청이 들어왔을 때 특정 테이블에 데이터를 등록 후 생성되는 key 값으로 

추가 작업이 필요하다면 쿼리 수행 후 해당 key를 반환하도록 해줘야 한다.

현재 Spring Boot 기반에 MyBatis를 사용하고 있어서 방법을 찾아보았다.
  
<br/>

###1. useGeneratedKeys, keyProperty 속성

데이터베이스 내부적으로 ``자동 생성한 키``를 받는 경우 사용한다.

```oracle
<insert id="insertUser" useGeneratedKeys="true" keyProperty="user_no" parameterType="UserDto">
  insert into user_main ( user_id, user_pwd )
  values ( #{userId}, #{userPwd} )
</insert>
```

<br/>

###2. selectKey 구문

데이터베이스에서 ``auto_increment``를 지원하지 않는 경우 사용하는 방법이다.

selectKey 구문이 먼저 실행되고 ``user_no`` 프로퍼티에 셋팅된다. 

이후 insert 쿼리가 실행된다.

```oracle
<insert id="insertUser" useGeneratedKeys="true" keyProperty="user_no" parameterType="UserDto">
   	<selectKey keyProperty="user_no" resultType="int" order="BEFORE">
      select SEQ_USER_NO.nexyval FROM DUAL
    </selectKey>
    insert into user_main ( user_id, user_pwd )
  values ( #{userId}, #{userPwd} )
</insert>
```

###### 참고
* [http://www.mybatis.org/mybatis-3/ko/sqlmap-xml.html](http://www.mybatis.org/mybatis-3/ko/sqlmap-xml.html)
