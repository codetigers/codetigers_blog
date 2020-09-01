

## 为什么会发生ajax跨域问题

- 浏览器限制
- 跨域(输入的url请求和实际的url请求不同)
- XHR(XMLHttpRequest)请求(如果不是这种格式那么就算发生跨域也不会报错)

## 解决思路

1.浏览器

2.XHR使用Jsonp(少用)

3.跨域 

> 被调用方: 使用相关方法来允许不同的url的请求访问(支持跨域)
>
> 调用方:通过代理，将A域名转到B域名(隐藏跨域)

## JSONP

> 原理:在url请求的后面加上了callback字段，因此在后台也需要用一个callback来解决。
>
> json代码的内容就是callback的值作为函数名，返回的数据作为函数的参数

![](https://gitee.com//codetigers/images/raw/master/img/20200714091023.png)

> 弊端:
>
> 1. 服务器需要改动代码支持
> 2. 只支持GET
> 3. 发送的不是XHR请求

## 被调用方允许跨域

跨域请求头和普通请求头的区别:有一个Origin,当返回后浏览器判断当前返回头中有没有跨域域名的信息，没有就报错

![](https://gitee.com//codetigers/images/raw/master/img/20200714093532.png)

### 解决方法:filter解决方法

![](https://gitee.com//codetigers/images/raw/master/img/20200714093942.png)

![](https://gitee.com//codetigers/images/raw/master/img/20200714094100.png)

## 简单请求和非简单请求

![](https://gitee.com//codetigers/images/raw/master/img/20200714094211.png)

![](https://gitee.com//codetigers/images/raw/master/img/20200714100655.png)

如果这样设置那么其他的跨域请求就无法得到相应，因此通过判断origin来设置响应头来解决

![](https://gitee.com//codetigers/images/raw/master/img/20200714101017.png)

隐藏跨域(通过反向代理，使得调用的都是相对地址，而不是绝对地址，也就不会出现跨域)

**apache解决然后通过访问/test地址**

![](https://gitee.com//codetigers/images/raw/master/img/20200714104445.png)

**nginx调用**

![](https://gitee.com//codetigers/images/raw/master/img/20200714104824.png)