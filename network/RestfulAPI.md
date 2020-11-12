## Restful API规范

### URI
>URI 表示资源，资源一般对应服务器端领域模型中的实体类。

>1,URI规范            
>2,不用大写；      
>3,用中杠-不用下杠_；          
>4,参数列表要encode；         
>5,URI中的名词表示资源集合，使用复数形式           

### 安全性和幂等性
>1,安全性：不会改变资源状态，可以理解为只读的；
>2,幂等性：执行1次和执行N次，对资源状态改变的效果是等价的。


Method | 安全性 | 幂等性
:-:|:-:|:-:
 GET | √ | √
 POST| × | ×
 PUT | ×| √
 DELETE |× | √



>3,安全性和幂等性均不保证反复请求能拿到相同的response。以 DELETE 为例，第一次DELETE返回200表示删除成功，第二次返回404提示资源不存在，这是允许的。


### Request


### Response
```
{
    code
    msg
    data
}
```

### 错误处理
>1,不要发生了错误但给2xx响应，客户端可能会缓存成功的http请求；   
>2,正确设置http状态码，不要自定义；      
>3,Response body 提供 1) 错误的代码（日志/问题追查）；2) 错误的描述文本（展示给用户）。       


* [Restful API规范](https://novoland.github.io/%E8%AE%BE%E8%AE%A1/2015/08/17/Restful%20API%20%E7%9A%84%E8%AE%BE%E8%AE%A1%E8%A7%84%E8%8C%83.html)