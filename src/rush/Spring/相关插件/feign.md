## feign

#### 相关tips：

1. feign可以通过：

   ``````````````java
   @FeignClient(name = "weixin-externalcontact", url = "https://qyapi.weixin.qq.com/cgi-bin/externalcontact")
   ``````````````

   的方式，调用**非本地注册**的服务。



#### 相关问题：

1. feign能否通过代理服务器进行服务的转发？
2. feign的原理是什么？和http有什么区别？