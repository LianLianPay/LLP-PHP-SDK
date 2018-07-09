# LLP-PHP-SDK

欢迎来到连连， 本仓库中包含使用```PHP```请求连连的服务器端API时的示例工程及使用说明， 示例工程中含有收款结果查询API的请求示例。

## 主要内容

* [前置要求](#前置要求)

* [使用说明](#使用说明)

* [签名说明](#签名说明)

* [延伸阅读](#延伸阅读)


## 前置要求

1. PHP SDK的版本在5.6或5.6以上

2. 需开启PHP的curl服务

## 使用说明

您的服务器可通过HTTP协议直接请求连连的服务器端API， 要求如下:

* HTTP请求的媒体类型应设置为:

```text
Content-type: application/json;charset=utf-8
```

* 请求方法应为```POST```。

* 需使用[HTTPS](https://baike.baidu.com/item/https/285356?fr=aladdin)协议。

* 在您与连连的交互中， 所有的信息须进行签名处理。

## 签名说明

首先生成签名原串， 如示例工程```lib```目录下```llpay_core.function.php```的```createLinkstring()```方法:

```php
/**
 * 把数组所有元素，按照“参数=参数值”的模式用“&”字符拼接成字符串
 * @param $para 需要拼接的数组
 * return 拼接完成以后的字符串
 */
function createLinkstring($para) {
	$arg  = "";
	while (list ($key, $val) = each ($para)) {
		$arg.=$key."=".$val."&";
	}
	//去掉最后一个&字符
	$arg = substr($arg,0,count($arg)-2);
	//file_put_contents("log.txt","转义前:".$arg."\n", FILE_APPEND);
	//如果存在转义字符，那么去掉转义
	if(get_magic_quotes_gpc()){$arg = stripslashes($arg);}
	//file_put_contents("log.txt","转义后:".$arg."\n", FILE_APPEND);
	return $arg;
}
```


加签时，使用示例工程```lib```目录下```llpay_rsa.function.php```的```Rsasign```方法进行加签，其中```priKey```即为您的私钥，```data```为签名原串：

```php
/********************************************************************************/

/**RSA签名
 * $data签名数据(需要先排序，然后拼接)
 * 签名用商户私钥，必须是没有经过pkcs8转换的私钥
 * 最后的签名，需要用base64编码
 * return Sign签名
 */

function Rsasign($data,$priKey) {
	//转换为openssl密钥，必须是没有经过pkcs8转换的私钥
    $res = openssl_get_privatekey($priKey);

	//调用openssl内置签名方法，生成签名$sign
    openssl_sign($data, $sign, $res,OPENSSL_ALGO_MD5);

	//释放资源
    openssl_free_key($res);
    
	//base64编码
	$sign = base64_encode($sign);
	//file_put_contents("log.txt","签名原串:".$data."\n", FILE_APPEND);
    return $sign;
}
```

验签时，使用示例工程```lib```目录下```llpay_rsa.function.php```的```Rsaverify```方法进行验签，其中```sign```连连向您发送的请求报文中的签名值，```data```为签名原串， ```pubKey```为连连向您提供的公钥：

```php
/********************************************************************************/

/**RSA验签
 * $data待签名数据(需要先排序，然后拼接)
 * $sign需要验签的签名,需要base64_decode解码
 * 验签用连连支付公钥
 * return 验签是否通过 bool值
 */
function Rsaverify($data, $sign)  {
	//读取连连支付公钥文件
	$pubKey = file_get_contents('key/llpay_public_key.pem');

	//转换为openssl格式密钥
    $res = openssl_get_publickey($pubKey);

	//调用openssl内置方法验签，返回bool值
    $result = (bool)openssl_verify($data, base64_decode($sign), $res,OPENSSL_ALGO_MD5);
	
	//释放资源
    openssl_free_key($res);

	//返回资源是否成功
    return $result;
}
```

## 延伸阅读

[连连开放平台 - API文档 - 新手指南](https://open.lianlianpay-inc.com/apis/get-started)

[连连开放平台 - 签名机制](https://open.lianlianpay-inc.com/docs/development/signature-overview)