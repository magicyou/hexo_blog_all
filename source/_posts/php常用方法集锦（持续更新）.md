---
title: PHP常用的方法集锦（持续更新）
date: 2018-02-03 17:23:00
updated: 2018-02-03 15:23:00
tags: ['PHP']
categories: ['PHP']
---
开发中一个项目又一个项目，积累一下常用方法，方便查阅
<!--more-->


* 获取头部信息

```php
/**
 * 获取头部信息
 * @param  string $name [参数名字]
 * @return string       [返回对应参数的值]
 */
function getHeader($name=''){
	$header = [];
    if (function_exists('apache_request_headers') && $result = apache_request_headers()) {
        $header = $result;
    } else {
        if($name = 'Authorization'){
            $name = 'authorization';
        }
        $server = $_SERVER;
        foreach ($server as $key => $val) {
            if (0 === strpos($key, 'HTTP_')) {
                $key          = str_replace('_', '-', strtolower(substr($key, 5)));
                $header[$key] = $val;
            }
        }
        if (isset($server['CONTENT_TYPE'])) {
            $header['content-type'] = $server['CONTENT_TYPE'];
        }
        if (isset($server['CONTENT_LENGTH'])) {
            $header['content-length'] = $server['CONTENT_LENGTH'];
        }
    }
    return $header[$name]?$header[$name]:null;
}

```



* 二维数组排序

```php
/**
 * 二维数组排序
 * @param string $arr 二维数组
 * @param string $keys 排序键值
 * @param string $type 排序方式 asc正序 desc倒
*/
function array_sort($arr, $keys, $type = 'asc'){
    $keysvalue = $new_array = array();
    foreach ($arr as $k => $v) {
    	$keysvalue[$k] = $v[$keys];
    }
    if ($type == 'asc') {
   		asort($keysvalue);
    } else {
        arsort($keysvalue);
    }
    reset($keysvalue);
    foreach ($keysvalue as $k => $v) {
    	$new_array[$k] = $arr[$k];
    }
    return $new_array;
}
```




* 获取用户token

```php
/**
* 获取用户token
*/
function get_user_token($user_id) {
       $token = new \Org\Gamegos\jwt\src\Token();
       $token->setClaim('user_id', $user_id); // alternatively you can use $token->setSubject('someone@example.com') method
       $token->setClaim('exp', time() + C('jwt_time'));
       $encoder = new \Org\Gamegos\jwt\src\Encoder();
       $encoder->encode($token, C('jwt_key'), C('jwt_alg'));
       return $token->getJWT();
}
```



* 数组随机指定个数抽取元素

```php
/**
 * 数组随机指定个数抽取元素
 * @param string $arr 数组
 * @param string $keys 指定个数
 */
function array_rand_to($arr, $num){
    $ran_arr = array_rand($arr, $num);
    $arr_new = [];
    for ($i = 0; $i < count($ran_arr); $i++) {
    	array_push($arr_new, $arr[$ran_arr[$i]]);
    }
    return $arr_new;

}
```



 * php验证身份证号码是否正确函数

```php
/*
 * php验证身份证号码是否正确函数
 * @param string $id 身份证号
 */
function is_idcard($id)
{
    $id = strtoupper($id);
    $regx = "/(^\d{15}$)|(^\d{17}([0-9]|X)$)/";
    $arr_split = array();
    if(!preg_match($regx, $id))
    {
        return FALSE;
    }
    if(15==strlen($id)) //检查15位
    {
        $regx = "/^(\d{6})+(\d{2})+(\d{2})+(\d{2})+(\d{3})$/";

        @preg_match($regx, $id, $arr_split);
        //检查生日日期是否正确
        $dtm_birth = "19".$arr_split[2] . '/' . $arr_split[3]. '/' .$arr_split[4];
        if(!strtotime($dtm_birth))
        {
            return FALSE;
        } else {
            return TRUE;
        }
    }
    else      //检查18位
    {
        $regx = "/^(\d{6})+(\d{4})+(\d{2})+(\d{2})+(\d{3})([0-9]|X)$/";
        @preg_match($regx, $id, $arr_split);
        $dtm_birth = $arr_split[2] . '/' . $arr_split[3]. '/' .$arr_split[4];
        if(!strtotime($dtm_birth)) //检查生日日期是否正确
        {
            return FALSE;
        }
        else
        {
            //检验18位身份证的校验码是否正确。
            //校验位按照ISO 7064:1983.MOD 11-2的规定生成，X可以认为是数字10。
            $arr_int = array(7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2);
            $arr_ch = array('1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2');
            $sign = 0;
            for ( $i = 0; $i < 17; $i++ )
            {
                $b = (int) $id{$i};
                $w = $arr_int[$i];
                $sign += $b * $w;
            }
            $n = $sign % 11;
            $val_num = $arr_ch[$n];
            if ($val_num != substr($id,17, 1))
            {
                return FALSE;
            } //phpfensi.com
            else
            {
                return TRUE;
            }
        }
    }

}
```



* 获取当前页面完整URL地址

```php
/**
 * 获取当前页面完整URL地址
 */
function get_url(){
    //$sys_protocal = isset($_SERVER['SERVER_PORT']) && $_SERVER['SERVER_PORT'] == '443' ? 'https://' : 'http://';
    $php_self   = $_SERVER['PHP_SELF'] ? $_SERVER['PHP_SELF'] : $_SERVER['SCRIPT_NAME'];
    $path_info  = isset($_SERVER['PATH_INFO']) ? $_SERVER['PATH_INFO'] : '';
    $relate_url = isset($_SERVER['REQUEST_URI']) ? $_SERVER['REQUEST_URI'] : $php_self . (isset($_SERVER['QUERY_STRING']) ? '?' . $_SERVER['QUERY_STRING'] : $path_info);
    return $relate_url;
    //return $sys_protocal . (isset($_SERVER['HTTP_HOST']) ? $_SERVER['HTTP_HOST'] : '') . $relate_url;
}
```



* 请求接口两种方式

```php
/**
 * GET方式请求接口
 * @param  [String] $url [curl地址]
 * @return [返回结果数据]
 */
function curl_get($url)
{
    // 初始化
    $curl = curl_init();
    // 设置抓取的url
    curl_setopt($curl, CURLOPT_URL, $url);
    // 设置头文件的信息作为数据流输出
    curl_setopt($curl, CURLOPT_HEADER, 0);
    // 设置获取的信息以文件流的形式返回，而不是直接输出。
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    // 执行命令
    $data = curl_exec($curl);
    // 关闭URL请求
    curl_close($curl);
    // 返回获得的数据
    return $data;
}

/**
 * POST方式请求接口
 * @param  [String] $url   [curl地址]
 * @param  [array]  $param [curl参数数组]
 * @return [返回结果数据]
 */
function curl_post($url, $param)
{
    // 初始化
    $curl = curl_init();
    // 设置抓取的url
    curl_setopt($curl, CURLOPT_URL, $url);
    // 设置头文件的信息作为数据流输出
    curl_setopt($curl, CURLOPT_HEADER, 1);
    // 设置获取的信息以文件流的形式返回，而不是直接输出。
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    // 设置post方式提交
    curl_setopt($curl, CURLOPT_POST, 1);
    // 设置post数据
    curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($param));
    // 执行命令
    $data = curl_exec($curl);
    // 关闭URL请求
    curl_close($curl);
    // 返回获得的数据
    return $data;
}
```



* 获取图片的Base64编码(不支持url)

```php
/**
 * 获取图片的Base64编码(不支持url)
 * @param $img_file [传入本地图片地址]
 * @return string
 */
function img_to_base64($file) {
    $code = '';
    if (file_exists($file)) {
        $info = getimagesize($file); // 取得图片的大小，类型等
        //echo '<pre>' . print_r($info, true) . '</pre><br>';

        $fp = fopen($file, "r"); // 图片是否可读权限
        if ($fp) {
            $type = '';
            $size = filesize($file);
            $content = fread($fp, $size);
            $encode = chunk_split(base64_encode($content)); // base64编码
            switch ($info[2]) {
                //判读图片类型
                case 1:
                    $type = "gif";
                    break;
                case 2:
                    $type = "jpg";
                    break;
                case 3:
                    $type = "png";
                    break;
            }
            $code = 'data:image/' . $type . ';base64,' . $encode; //合成图片的base64编码
        }
        fclose($fp);
    }

    return $code; //返回图片的base64
}
```

