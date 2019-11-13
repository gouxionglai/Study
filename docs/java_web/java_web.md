# java web



## Cookie & Session

先来一个概括

| 方法        | 信息量大小      | 保存时间                                | 应用范围 | 保存位置 |
| ----------- | --------------- | --------------------------------------- | -------- | -------- |
| Application | 任意大小        | 整个应用程序的生命期                    | 所有用户 | 服务器端 |
| Session     | 小量,简单的数据 | 用户活动时间+一段延迟时间(一般为20分钟) | 单个用户 | 服务器端 |
| Cookie      | 小量,简单的数据 | 可以根据需要设定                        | 单个用户 | 客户端   |



### Cookie: 客户端技术

#### HTTP协议的消息头

​	请求消息头：Cookie 客户端向服务器端传递信息

​	响应消息头：Set-Cookie 服务器端向客户端传递信息

#### Cookie详解

1. name：cookie的名称

   1. 必要的属性

2. value：cookie的取值

   1. 必要的属性
   2. **不能为中文**

3. path：cookie的路径

   1. 可选属性

   2. 默认值就是写cookie的那个资源的访问路径

      ```txt
      比如：http://localhost:8080/day09_00_cookie/servlet/CookieDemo1 
      path就是/day09_00_cookie/servlet/
      注意：
      	如果一个存在浏览器缓存中的cookie的路径是/day09/servlet/
      	当访问http://localhost:8080/day09/CookiePathDemo1时，
      	浏览器根本不带Cookie给服务器。浏览器比对的是cookie的路径和当前访问的资源的路径。
      	浏览器满足一下条件就会带cookie给服务器：
      		当前访问的地址的路径.startWith(已存cookie的路径)。即：如果一个Cookie的路径设置为了当前应用，说明访问该网站的任何资源时浏览器都带该cookie给服务器。（开发中经常做的）
      ```

      

4. maxAge：cookie最大生存时间

   1. 默认实在浏览器内存中

5. domain：cookie的域名（网址）

   1. 默认就是cookie的那个资源所属的网站

6. version：版本

7. comment：注释

### Session: 服务端技术

## Servlet