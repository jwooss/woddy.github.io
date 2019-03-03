---
title: '[PHP] CURL 파일 전송시 특수문자 이슈'
date: 2019-2-17 16:21:13
category: 'php'
---

PHP에서 CURL을 사용해 파일을 전송하는 할 일이 생겼다. 

이때, 파일 전송을 위해서는 curl 옵션을 설정하고 (Content-type 등..)

보내고자 하는 파일의 경로 앞에 '@' 를 붙이면 $_FILES 전역변수로 인식하여 정보가 들어가게 된다.

그런데 만약 파일이 아닌 데이터의 앞에 '@' 문자가 있으면 어떻게 될까?

코드로 간단하게 표현하면 다음과 같다. 

```php{4}
$request->post([
	'filePath' => '@' . $filePath
	'title' => 'title'
	'contents' => '@contents'
]);
```

filePath의 데이터는 실제 파일의 경로이기 때문에 문제가 없지만, contents의 데이터는 파일 경로가 아니므로 PHP에서는 파일이 아니라는 에러를 발생시키고 curl 전송이 실패하게 된다.

이런 이슈로 인해 5.5버전 이후로 curl\_file\_create라는 함수가 생겨서 문제가 해결되었다.

```php{4}
$request->post([
	'filePath' => curl_file_create($filePath, $fileType, $fileName)
	'title' => 'title'
	'contents' => '@contents'
]);
```

하지만 운영중인 서비스는 5.2버전으로 함수를 사용할 수 없어서 다른 방법을 찾아야 했다. 

하루 정도 시간을 투자하여 여러 방법으로 시도해 보다가 CURL 본문을 직접 작성하는 방법으로 성공하였다. 

참고한 코드는 다음과 같다.

```php
/**
* For safe multipart POST request for PHP5.3 ~ PHP 5.4.
* 
* @param resource $ch cURL resource
* @param array $assoc "name => value"
* @param array $files "name => path"
* @return bool
*/
function curl_custom_postfields($ch, array $assoc = array(), array $files = array()) {
    
    // invalid characters for "name" and "filename"
    static $disallow = array("\0", "\"", "\r", "\n");
    
    // build normal parameters
    foreach ($assoc as $k => $v) {
        $k = str_replace($disallow, "_", $k);
        $body[] = implode("\r\n", array(
            "Content-Disposition: form-data; name=\"{$k}\"",
            "",
            filter_var($v), 
        ));
    }
    
    // build file parameters
    foreach ($files as $k => $v) {
        switch (true) {
            case false === $v = realpath(filter_var($v)):
            case !is_file($v):
            case !is_readable($v):
                continue; // or return false, throw new InvalidArgumentException
        }
        $data = file_get_contents($v);
        $v = call_user_func("end", explode(DIRECTORY_SEPARATOR, $v));
        $k = str_replace($disallow, "_", $k);
        $v = str_replace($disallow, "_", $v);
        $body[] = implode("\r\n", array(
            "Content-Disposition: form-data; name=\"{$k}\"; filename=\"{$v}\"",
            "Content-Type: application/octet-stream",
            "",
            $data, 
        ));
    }
    
    // generate safe boundary 
    do {
        $boundary = "---------------------" . md5(mt_rand() . microtime());
    } while (preg_grep("/{$boundary}/", $body));
    
    // add boundary for each parameters
    array_walk($body, function (&$part) use ($boundary) {
        $part = "--{$boundary}\r\n{$part}";
    });
    
    // add final boundary
    $body[] = "--{$boundary}--";
    $body[] = "";
    
    // set options
    return @curl_setopt_array($ch, array(
        CURLOPT_POST       => true,
        CURLOPT_POSTFIELDS => implode("\r\n", $body),
        CURLOPT_HTTPHEADER => array(
            "Expect: 100-continue",
            "Content-Type: multipart/form-data; boundary={$boundary}", // change Content-Type
        ),
    ));
}
```

문제가 해결돼서 좋긴한데 PHP 버전을 올려야 하는 필요성을 다시 한 번 느낀다.


###### 참고
* [http://php.net/manual/en/class.curlfile.php](http://php.net/manual/en/class.curlfile.php)
