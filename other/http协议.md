> 资料来源：http://gityuan.com/2015/06/20/http-agreement/

### 3.请求方法

HTTP/1.1协议共定义八种请求方法：

| 序号 | 方法    | 描述                                                         |
| :--- | :------ | :----------------------------------------------------------- |
| 1    | GET     | 请求指定的页面信息，并返回实体主体。                         |
| 2    | POST    | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。 |
| 3    | PUT     | 从客户端向服务器传送的数据取代指定的文档的内容。             |
| 4    | DELETE  | 请求服务器删除指定的页面。                                   |
| 2    | HEAD    | 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头 |
| 6    | CONNECT | HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。     |
| 7    | OPTIONS | 允许客户端查看服务器的性能。                                 |
| 8    | TRACE   | 回显服务器收到的请求，主要用于测试或诊断。                   |

### 4.请求头

提供了关于请求，响应或者其他的发送实体的信息

| 头               | 描述                                                         | 示例                  |
| :--------------- | :----------------------------------------------------------- | :-------------------- |
| Allow            | 服务器支持哪些请求方法（如GET、POST等）                      |                       |
| Accept-Encoding  | 指定浏览器可以支持的web服务器返回内容压缩编码类型            | Accept-Encoding: gzip |
| Content-Encoding | 文档的**编码方法**，只有在解码后才得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间,但并非所有浏览器都支持gzip。 |                       |
| Content-Length   | 表示**实体内容长度**，只有当浏览器使用持久HTTP连接时才需要这个数据。 |                       |
| Content-Type     | 表示文档的**MIME类型**, Servlet默认为text/plain，但通常需要显式地指定为text/html，HttpServletResponse提供了一个专用的方法setContentType。 |                       |
| Date             | 当前的**GMT时间**，可以用setDateHeader来设置这个头以避免转换时间格式的麻烦。 |                       |
| Expires          | 文档的**有效期时长**，过期则不再缓存                         |                       |
| Last-Modified    | 文档的**最后改动时间**，Last-Modified也可用setDateHeader方法来设置。 |                       |
| Location         | 表示提取**文档位置**，Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。 |                       |
| Refresh          | 表示浏览器的文件的**刷新间隔**，N秒后刷新本页面或访问指定页面，此为扩展头，不属于HTTP 1.1正式规范。 |                       |
| Set-Cookie       | 设置和页面关联的Cookie，应使用HttpServletResponse提供的专用方法addCookie。 |                       |