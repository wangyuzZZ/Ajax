#Javascript Ajax请求

## 相关知识点

### 1.同步异步

- 同步：主线程执行，客户端发起请求->服务器响应->页面加载，阻塞线程。
- 异步：后台执行，客户端发起请求->服务器响应->页面加载同时执行。

### 2.GET & POST

- GET：一般用于信息获取，请求参数拼接在地址后，不安全，速度快。
- POST：一般用于修改服务器上的资源，请求参数打包在请求报文中，安全，速度慢。

### 3.HTTP请求

http 是计算机通过网络进行通信的规则，http是一种无状态协议。

> 一个完整的http请求，通常有下面7个步骤：

1）、建立TCP连接

2）、Web浏览器向Web服务器发送请求命令

3）、Web浏览器发送请求头信息

4）、Web服务器应答

5）、Web服务器发送应答头信息

6）、Web服务器向浏览器发送数据

7）、Web服务器关闭TCP连接

> 一个HTTP请求一般由四部分组成：

1）、HTTP请求的方法或动作，比如是GET还是POST请求

2）、URL 请求地址

3）、请求头，包含一些客户端环境信息，身份验证信息等

4）、请求体，也就是请求正文，请求正文中可以包含客户提交的查询字符串信息，表单信息等等。

> 一个HTTP响应一般由三部分组成：

1）、一个数字和文字组成的状态码，用来显示请求是成功还是失败；

2）、响应头，响应头也和请求头一样包含许多有用的信息，例如数据类型、日期时间、内容类型和长度等。

3）、响应体，也就是响应正文

HTTP状态码由3位数字构成，其中首位数字定义了状态码的类型

1）、1XX：信息类，表示收到Web浏览器请求，正在进一步的处理中；

2）、2XX：成功，表示用户请求被正确接收、理解和处理。例如 200 OK；

3）、3XX：重定向，表示请求没有成功，客户必须采取进一步的动作；

4）、4XX：客户端错误，表示客户端提交的请求有错误。例如 404 Not Found：请求中引用的文档不存在；

5）、5XX：服务器错误，表示服务器不能完成对请求的处理。例如 500

### 4.Ajax请求实例

```javascript
function sendAjax() {
  // 1. 创建xhr对象 
  var xhr = new XMLHttpRequest();
  // 2. 设置xhr请求的超时时间
  xhr.timeout = 3000;
  // 3. 设置响应返回的数据格式
  xhr.responseType = "json";
// 4. 创建一个 GET 请求，采用异步
  xhr.open('GET', 'json/goods.json', true);
  // 5. 注册相关事件回调处理函数
  xhr.onload = function(e) { 
    if(this.status == 200|| this.status == 304){
        alert(this.response);
    }
  };
  xhr.ontimeout = function(e) { ... };
  xhr.onerror = function(e) { ... };
  xhr.upload.onprogress = function(e) { ... };
  
  // 6. 发送数据
  xhr.send(null);
}
```

#### 4.1 设置request header

在发送 Ajax 请求（实质是一个[HTTP](http://www.tutorialspoint.com/http/http_header_fields.htm)请求）时，我们可能需要设置一些请求头部信息，比如 `content-type`、`connection`、`cookie`、`accept-xxx` 等。xhr 提供了`setRequestHeader`来允许我们修改请求 header。

>```c
>void setRequestHeader(DOMString header, DOMString value);
>```

- 方法的第一个参数 header 大小写不敏感，即可以写成 *content-type* ，也可以写成*Content-Type*，甚至写成*content-Type*;
- *Content-Type* 的默认值与具体发送的数据类型有关
- *setRequestHeader* 必须在*open()*方法之后，*send()* 方法之前调用，否则会抛错；
- *setRequestHeader* 可以调用多次，最终的值不会采用覆盖 `override` 的方式，而是采用追加 `append` 的方式。例：

```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', 'demo.json');
xhr.setRequestHeader('X-Test', 'one');
xhr.setRequestHeader('X-Test', 'two');
// 最终request header中"X-Test"为: one, two
xhr.send();
```

#### 4.2 获取response header

>`DOMString getAllResponseHeaders();`
>
>`DOMString getResponseHeader(DOMString header);`

前者是获取 response 中的所有header 字段，后者只是获取某个指定 header 字段的值。

[W3C在 xhr 标准中的限制](https://www.w3.org/TR/XMLHttpRequest/) 规定客户端无法获取 response 中的 `Set-Cookie`、`Set-Cookie2` 这2个字段，无论是同域还是跨域请求；

[W3C在 cors 标准中对于跨域请求的限制](https://www.w3.org/TR/cors/#access-control-allow-credentials-response-header) 规定对于跨域请求，客户端允许获取的response header字段只限于 

“`simple response header`”

“`Access-Control-Expose-Headers`” 

> a. *simple response header* 包括的 header 字段有：`Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified`, `Pragma`;
>
> b. *Access-Control-Expose-Headers*：首先得注意是"*Access-Control-Expose-Headers*"进行**跨域请求**时响应头部中的一个字段，对于同域请求，响应头部是没有这个字段的。这个字段中列举的 header 字段就是服务器允许暴露给客户端访问的字段。

所以 *getAllResponseHeaders()*  只能拿到**限制以外**（即被视为`safe`）的header字段，而不是全部字段；而调用 *getResponseHeader(header)* 方法时，`header` 参数必须是**限制以外**的header字段，否则调用就会报`Refused to get unsafe header` 的错误。

#### 4.3 指定xhr.response数据类型

| 值              | `xhr.response` 数据类型 | 说明                             |
| --------------- | ----------------------- | -------------------------------- |
| `""`            | `String` 字符串         | 默认值(在不设置`responseType`时) |
| `"text"`        | `String` 字符串         |                                  |
| `"document"`    | `Document` 对象         | 希望返回 `XML` 格式数据时使用    |
| `"json"`        | `javascript` 对象       | 存在兼容性问题，IE10/IE11不支持  |
| `"blob"`        | `Blob` 对象             |                                  |
| `"arrayBuffer"` | `ArrayBuffer` 对象      |                                  |

例

```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', 'json/sale.json', true);
xhr.responseType = 'json';
xhr.onload = function(e) {
    if (this.status == 200) {
        var obj = this.response;
        console.log(obj);
    }
};
xhr.send();
```

#### 4.4 获取response

- `xhr.response`

  - 默认值：空字符串 `""`
  - 当请求完成时，此属性才有正确的值
  - 请求未完成时，此属性的值可能是`""`或者 `null`，具体与 *xhr.responseType*有关：当*responseType* 为`""`或`"text"`时，值为`""`；`responseType`为其他值时，值为 `null`

- `xhr.responseText`

  - 默认值为空字符串`""`
  - 只有当 *responseType*  为 `"text"`、`""` 时，xhr 对象上才有此属性，此时调用 *xhr.responseText*，否则抛错
  - 只有当请求成功时，才能拿到正确值。以下2种情况下值都为空字符串`""`：请求未完成、请求失败

- `xhr.responseXML`

  - 默认值为 `null`
  - 只有当 *responseType*  为`"text"`、`""`、`"document"`时，xhr 对象上才有此属性，此时才能调用 *xhr.responseXML*，否则抛错
  - 只有当请求成功且返回数据被正确解析时，才能拿到正确值。以下3种情况下值都为`null`：请求未完成、请求失败、请求成功但返回数据无法被正确解析时

  #### 4.5 追踪ajax请求当前状态(xhr.readyState)

  ```javascript
  xhr.onreadystatechange = function () {
      switch(xhr.readyState){
          case 1://OPENED
              //do something
              break;
          case 2://HEADERS_RECEIVED
              //do something
              break;
          case 3://LOADING
              //do something
              break;
          case 4://DONE
              //do something
              break;
      }
  };
  ```

  

| 值   | 状态                             | 描述                                                         |
| ---- | -------------------------------- | ------------------------------------------------------------ |
| `0`  | `UNSENT` (初始状态，未打开)      | 此时`xhr`对象被成功构造，`open()`方法还未被调用              |
| `1`  | `OPENED` (已打开，未发送)        | `open()`方法已被成功调用，`send()`方法还未被调用。注意：只有`xhr`处于`OPENED`状态，才能调用 *xhr.setRequestHeader()* 和*xhr.send()* , 否则会报错 |
| `2`  | `HEADERS_RECEIVED`(已获取响应头) | `send()`方法已经被调用, 响应头和响应状态已经返回             |
| `3`  | `LOADING` (正在下载响应体)       | 响应体(`response entity body`)正在下载中，此状态下通过`xhr.response`可能已经有了响应数据 |
| `4`  | `DONE` (整个数据传输过程结束)    | 整个数据传输过程结束，不管本次请求是成功还是失败             |

#### 4.6 请求超时时间(xhr.timeout)

>xhr.timeout`
>
>单位：milliseconds 毫秒
>
>默认值：`0`，即不设置超时

从**请求开始** 算起，若超过 `timeout` 时间请求还没有结束（包括成功/失败），则会触发 *ontimeout* 事件，主动结束该请求。

>xhr.onloadstart* 事件触发的时候，也就是你调用 *xhr.send()* 方法的时候。
>
>因为 *xhr.open()* 只是创建了一个连接，但并没有真正开始数据的传输，而*xhr.send()*才是真正开始了数据的传输过程。只有调用了`xhr.send()`，才会触发`xhr.onloadstart` 。

请求结束

>xhr.loadend` 事件触发的时候 。

>1. 可以在 `send()`之后再设置此`xhr.timeout`，但计时起始点仍为调用`xhr.send()`方法的时刻。
>2. 当 xhr为一个 `sync` 同步请求时，`xhr.timeout`必须置为`0`，否则会抛错。原因可以参考本文的如“何发一个同步请求”一节。

#### 4.7 发送一个同步请求

xhr  默认发的是异步请求，但也支持发同步请求（当然实际开发中应该尽量避免使用）。到底是异步还是同步请求，由 *xhr.open()* 传入的 *async* 参数决定。

> `open(method, url [, async = true [, username = null [, password = null]]])`

- `method`: 请求的方式，如`GET/POST/HEADER`等，这个参数不区分大小写
- `url`: 请求的地址，可以是相对地址如`example.php`，这个**相对**是相对于当前网页的`url`路径；也可以是绝对地址如`http://www.example.com/example.php`
- `async`: 默认值为`true`，即为异步请求，若`async=false`，则为同步请求

同步请求需要注意以下几点：

- xhr.timeout`必须为`0`
- `xhr.withCredentials`必须为 `false`
- `xhr.responseType`必须为`""`（注意置为`"text"`也不允许）

#### 4.8 获取上传、下载进度

在上传或者下载比较大的文件时，实时显示当前的上传、下载进度是很普遍的产品需求。
我们可以通过`onprogress`事件来实时显示进度，默认情况下这个事件每50ms触发一次。需要注意的是，上传过程和下载过程触发的是不同对象的 *onprogress* 事件：

- 上传触发的是`xhr.upload`对象的 `onprogress`事件
- 下载触发的是`xhr`对象的`onprogress`事件

```javascript
xhr.onprogress = updateProgress;
xhr.upload.onprogress = updateProgress;
function updateProgress(event) {
    if (event.lengthComputable) {
      var completedPercent = event.loaded / event.total;
    }
 }
```

#### 4.9 可以发送什么类型的数据

*xhr.send(data)*  的参数data可以是以下几种类型：

- `ArrayBuffer`
- `Blob`
- `Document`
- `DOMString`
- `FormData`
- `null`

如果是 GET/HEAD请求，*send()* 方法一般不传参或传 `null`。不过即使你真传入了参数，参数也最终被忽略，*xhr.send(data)* 中的data会被置为 `null`.

*xhr.send(data)*  中data参数的数据类型会影响请求头部`content-type`的默认值：

- 如果data是 “Document” 类型，同时也是“HTML Document”类型，则 “content-type” 默认值为 *text/html;charset=UTF-8* ;否则为*application/xml;charset=UTF-8*；
- 如果data是 “DOMString” 类型，“content-type”默认值为“text/plain;charset=UTF-8”；
- 如果data是 “FormData” 类型，“content-type”默认值为“multipart/form-data; boundary=[xxx]”
- 如果data是其他类型，则不会设置“content-type”的默认值

# Jquery Ajax请求

```javascript
$.ajax({
    url:"",
    responseType:"数据类型"
    success(res){
        
    }
})
```

