---
title: 微信小程序---PHP获取微信小程序码
date: 2019-07-03 21:33:00
updated: 2019-07-03 21:33:00
tags: ['PHP','微信小程序','JavaScript']
categories: ['前端']
---
记录一下折腾小程序遇到的问题，PHP获取微信小程序码。省的以后再次踩坑

<!--more-->

小程序端要合成海报，上面带小程序码，所以有了这个博文过程。

#### 先提供封装的工具类部分源码

```
  /**
     *
     * 获取小程序码
     * @param string $access_token 接口凭证
     * @param string $urlPath 小程序路径
     * @return array 应答
     */
    public function createQRCode($access_token, $urlPath)
    {

        $url = "https://api.weixin.qq.com/wxa/getwxacode?access_token={$access_token}";
        $post_data = [
            "path"          => $urlPath,
            "auto_color"    => false,
            "width"         => 240,
        ];

        $re_string = $this->curlPost($url, $post_data);

        $re_data = json_decode($re_string, true);

        if ($re_data['errcode']) {
            return $re_data;
        }

        $path = 'Uploads/recode/' . date('Ymd') . '/';

        if (!is_dir($path)) {
            mkdir($path, 0755, true);
        }
        $filepath = $path . 'recode_' . time() . ".png";
        $file = fopen($filepath, "w");
        fwrite($file, $re_string);
        fclose($file);

        return ['file_path'=>$filepath];
    }
    
    /**
     *
     * 登录凭证校验。通过 wx.login 接口获得临时登录凭证 code 后传到开发者服务器调用此接口完成登录流程
     * @return array 应答
     */
    public function getAccessToken()
    {
        $appid = $this->appid; // 小程序APPID
        $secret = $this->secret; // 小程序secret

        $url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid={$appid}&secret={$secret}";
        return $this->curlGet($url);
    }
    
    /**
     * 发送请求
     * @param string $url   请求地址
     * @return string 应答json字符串
     */
    public function curlPost($url, $dataObj)
    {

        $curl = curl_init();
        curl_setopt($curl, CURLOPT_URL, $url);
        curl_setopt($curl, CURLOPT_HEADER, 0);
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($curl, CURLOPT_POST, 1);
        curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($dataObj));
        curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, 0);
        curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, 0);
        curl_setopt($curl, CURLOPT_HTTPHEADER, array("Content-type: application/json") );
        $ret = curl_exec($curl);

        curl_close($curl);
        return $ret;
    }
    
    /**
     * GET方式请求接口
     * @param  [String] $url [curl地址]
     * @return [返回结果数据]
     */
    private function curlGet($url)
    {
        $curl = curl_init();
        curl_setopt($curl, CURLOPT_URL, $url);
        curl_setopt($curl, CURLOPT_HEADER, 0);
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
        $data = curl_exec($curl);
        curl_close($curl);
        return json_decode($data, true);
    }
```

不想啰嗦的直接拿走用，写的简单，直接用就可以。



#### 一步一步阐述遇到的问题

##### * 官方文档

先来看看官方文档，传送门[https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/qr-code/wxacode.createQRCode.html]

官方提供了三种获取小程序码Api，对应不同的业务场景。这里我选用第二个，

##### * 报错 47001

完整错误信息如下：

```
string(69) "{"errcode":47001,"errmsg":"data format error hint: [21WqGa05154711]"}"
```

死活找不到问题，post修了好几次，后来找到一篇博文，说要用post请求body要用raw，格式为json

```
"Content-type: application/json"
```

改完后依旧报同样的错，无奈，再后来找到博文说，获取获取小程序码第二三个接口调用时，post参数不需要放access_token，试了试，额，，，，果然好了。

先来看看官方的文档证据：

![image-20190913234905295](/Users/magicyou/Desktop/image-20190913234905295.png)

难道是我不够心细，没看到官方说明。仔细从上到下，没发现说post参数不需要access_token，官方文档有点坑X。

##### * 其他报错

其他也遇到了错误，不过报错翻译就明白怎么解决，比如：

* access_token失效；access_token有效期7200秒，做一下存储，处理一下就行
* path参数缺失；第三个Api的url路径参数名是page，额，，，我搞岔劈了



#### 进一步优化

##### 存储access_token

access_token有效期为两小时，且每天获取次数有限制，最好做一下缓存，方法很多，文件存储，radis存储等。我这里用Mysql存一下，还能记录获取历史记录。

创建数据表：

```
CREATE TABLE `tp_wx_access_token` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `access_token` varchar(255) DEFAULT NULL COMMENT 'access_token',
  `appid` varchar(100) DEFAULT NULL COMMENT '微信小程序appid',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `create_timestamp` bigint(20) DEFAULT NULL COMMENT '创建时间戳',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8mb4;
```

##### 存储小程序码

省的每次都要重新获取，既然每次都要获取后都会存下来，索性获取记录也做个存储。获取前参数作为条件查询是否存在，有了直接返回，没的话获取再存储再返回

创建数据表：

```
CREATE TABLE `tp_wx_acode` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `appid` varchar(50) DEFAULT NULL COMMENT '小程序appid',
  `file_path` varchar(255) DEFAULT NULL COMMENT '小程序图片保存',
  `url_path` varchar(255) DEFAULT NULL COMMENT '关联的url',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=22 DEFAULT CHARSET=utf8mb4 COMMENT='小程序码生成记录';
```

这样子下来，保证了查询速度，都做了存储，自我感觉良好



#### 使用

```
   /**
     * 获取小程序码
     * @param $param
     * @return array
     */
    public function getAcode($param)
    {

        $data = $this
            ->where(['appid'=>C('WX_APPID'), 'url_path' =>$param['url'] ])
            ->find();

        if ($data) {
            $data['file_path'] = 'https://' . $_SERVER['HTTP_HOST'] . '/' . $data['file_path'];
            $array = [
                'code' => 1,
                'msg'  => '获取成功',
                'data' => $data,
            ];
            return $array;
        }

        // 检查token是否过期
        $access_token_info = M('wx_access_token')
            ->where(['appid' => C('WX_APPID')])
            ->order('create_time desc')
            ->find();

        $applet = new Applet();
        if ($access_token_info && $access_token_info['create_timestamp'] + 7200 > time()) {
            $access_token = $access_token_info['access_token'];
        } else {

            $re_token = $applet->getAccessToken();
            if (!$re_token['access_token']) {
                $array = [
                    'code' => 0,
                    'msg'  => 'access_token获取失败',
                ];
                return $array;
            }

            $access_token = $re_token['access_token'];

            $data = [
                'access_token'          => $access_token,
                'appid'                 => C('WX_APPID'),
                'create_time'           => date('Y-m-d H:i:s'),
                'create_timestamp'      => time(),
            ];
            M('wx_access_token')->add($data);
        }

        $re_create =  $applet->createQRCode($access_token, $param['url']);
        if ($re_create['errcode']) {
            $array = [
                'code' => 0,
                'msg'  => '二维码获取失败',
                'data' => $re_create
            ];
            return $array;
        }

        $data = [
            'appid'        => C('WX_APPID'),
            'file_path'    => $re_create['file_path'],
            'url_path'     => $param['url'],
            'create_time'  => date('Y-m-d H:i:s'),
        ];
        $add_result = $this->add($data);
        if (!$add_result) {
            $array = [
                'code' => 0,
                'msg'  => '获取失败'
            ];
            return $array;
        }


        $array = [
            'code' => 1,
            'msg'  => '获取成功',
            'data' =>[
                'file_path'    => 'https://'. $_SERVER['HTTP_HOST'] .'/'. $re_create['file_path'],
            ],
        ];

        return $array;
    }

```





收工