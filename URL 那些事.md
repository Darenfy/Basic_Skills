### URL 那些事  
Duang～ 网络上有道常见题目：从输入 URL 到网页显示，其间发生了什么？  


正好最近在整理笔记。准备从【安全】技术角度梳理 URL 相关知识 
仅仅记录自己所知，如果有错误，请指出
更细节的东西请各位大佬自行处理
## 产生
### 域名生成  
域名这种东西放在平时都是挑自己喜欢的去买就好
如果放在安全领域，需要考虑的就是域名生成算法  

DGA（domain generate algorithm）是一种伪随机域名生成算法，可批量生成大量的伪随机域名，作为C&C（command and control server）域名。

域名生成算法经常被用作恶意软件连接 C2 中控。恶意软件定期使用 ``DGA`` 算法生成为随机域名，有效绕过黑名单检测，尝试连接，寻找 ``C2`` 中控。

通常域名生成算法分为四类：
- 基于算数 -> 大部分 
- 基于Hash
- 基于单词表  
- 基于置换

比如几次恶意软件感染事件均使用了DGA域名：
- ``Wannacry``(iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com)
- ``xshell``(xmponmzmxkxkh.com)
- ``ccleaner``(ab3d685a0c37.com)

传统的检测方法：黑名单库
现有的检测方法：长短期记忆网络 LSTM

参考资料：
- https://mp.weixin.qq.com/s/GlWqTWQzBfoXt8J8uJAPRQ
- https://www.sohu.com/a/204200100_354899
- [Predicting Domain Generation Algorithms
with Long Short-Term Memory Networks](https://arxiv.org/pdf/1611.00791.pdf)

## 连接
### URL 解析  
想要访问特定资源，首先要判断相应资源所处目录（路径）、所处服务器（域名 + 端口）、通信方式（协议）

使用 [urllib.parse](https://docs.python.org/3/library/urllib.parse.html) 进行解析最为方便 

URL 一般结构  
``scheme://netloc/path;parameters?query#fragment``

本文主要考虑
- scheme -> 网络协议（http/https/ftp...）  
- netloc -> 网络位置（域名 + 端口）  
- path -> 分层路径  

### 获取 IP 地址  
域名是为了让人们更方便的记住所需要的资源位置，而便于计算机记录的是 IP 地址。这时就需要对 DNS 服务器进行访问，通过 ARP 协议获取对应 IP

参考链接：
https://blog.csdn.net/weixin_44604541/article/details/115909130?spm=1001.2014.3001.5501

### TCP 连接 【附 Socket 通信】
TCP 会使用三次握手，这里我们只需知道三次握手在流量包中的显示特征即可  

主要关注 Socket 客户端的 Java 实现  
- Step 1：创建Socket对象，指明需要链接的服务器的地址和端号
- Step 2：链接建立后，通过输出流向服务器发送请求信息
- Step 3：通过输出流获取服务器响应的信息
- Step 4：关闭相关资源
```java
import java.net.Socket;

// 获取 IP 端口号 
mSocket = new Socket(mIpAddress,mClientPort);

// 获取输入输出流  
mOutStream = mSocket.getOutputStream();
mInStream = mSocket.getInputStream();

// 发送获取相应信息  
mOutStream.write(msg.getBytes());
mOutStream.flush();

byte[] buffer = new byte[1024];
int count = mInStream.read(buffer);

// 关闭通信连接
mOutStream.close();
mInStream.close();
mSocket.close();
```
参考链接：
https://developer.android.com/reference/java/net/Socket

### HTTP/HTTPS 连接传输
使用 HTTP 协议的通信量占大头
其中 HTTPS 中 TLS 会使用四次握手，这里我们只需知道四次握手在流量包中的显示特征即可  

客户端传输内容主要使用系统内置 API 和 HTTP 框架
- HttpURLConnection
```java
// Create a neat value object to hold the URL
URL url = new URL("https://bbs.pediy.com");
HttpURLConnection connection = (HttpURLConnection) url.openConnection();
connection.setRequestProperty("accept", "application/json");
InputStream responseStream = connection.getInputStream();
ObjectMapper mapper = new ObjectMapper();
APOD apod = mapper.readValue(responseStream, APOD.class);
System.out.println(apod.title);
```
- HTTPClient  
```java
var client = HttpClient.newHttpClient();
var request = HttpRequest.newBuilder(
       URI.create("https://bbs.pediy.com"))
   .header("accept", "application/json")
   .build();
var response = client.send(request, new JsonBodyHandler<>(APOD.class));
System.out.println(response.body().get().title);
```
- Apache HttpClient  
```java
ObjectMapper mapper = new ObjectMapper();

try (CloseableHttpClient client = HttpClients.createDefault()) {
   HttpGet request = new HttpGet("https://bbs.pediy.com");
   APOD response = client.execute(request, httpResponse ->
       mapper.readValue(httpResponse.getEntity().getContent(), APOD.class));
   System.out.println(response.title);
}
```
- OkHttp  
```java
ObjectMapper mapper = new ObjectMapper();
OkHttpClient client = new OkHttpClient();
Request request = new Request.Builder()
   .url("https://bbs.pediy.com")
   .build();
Response response = client.newCall(request).execute();
APOD apod = mapper.readValue(response.body().byteStream(), APOD.class);
System.out.println(apod.title);
```
- Retrofit  
```java
Retrofit retrofit = new Retrofit.Builder()
   .baseUrl("https://bbs.pediy.com")
   .addConverterFactory(JacksonConverterFactory.create())
   .build();
APODClient apodClient = retrofit.create(APODClient.class);
CompletableFuture<APOD> response = apodClient.getApod("DEMO_KEY");
APOD apod = response.get();
System.out.println(apod.title);
```

参考链接：
https://www.twilio.com/blog/5-ways-to-make-http-requests-in-java

### 验证证书  
举个🌰 okkttp 验证服务器证书的调用栈
```java
com.android.org.conscrypt.TrustManagerImpl.checkServerTrusted()
android.security.net.config.NetworkSecurityTrustManager.checkServerTrusted()
android.security.net.config.NetworkSecurityTrustManager.checkServerTrusted()
android.security.net.config.RootTrustManager.checkServerTrusted()
com.android.org.conscrypt.Platform.checkServerTrusted()
com.android.org.conscrypt.OpenSSLSocketImpl.verifyCertificateChain()
com.android.org.conscrypt.NativeCrypto.SSL_do_handshake()
com.android.org.conscrypt.OpenSSLSocketImpl.startHandshake()
com.android.okhttp.Connection.connectTls()
```

参考链接：
https://developer.android.com/training/articles/security-ssl?hl=zh-cn
https://www.huaweicloud.com/articles/35fe55ce37013d706188fadb8644ef73.html

## 资源获取  
### 下载  
下载流程一般使用 ``Requests`` 库  

进行一般过滤的单线程 demo
```py
def downloadTask(url):

    log.info("{}/processing: {}".format(url))

    try:
        r = requests.get(url=url, verify=False, stream=True)
    except:
        log.error("[0] requests.get error: {}".format(url))
        return

    try:
        if int(r.headers['Content-length']) > 1024 * 1024 * 2:
            log.info("[1] exceeding quota: {}".format(url))
            return
    except:
        pass

    if r.status_code != 200:
        log.error("[2] status error: {}".format(url))
        return

    return
```

### 爬虫  
使用 ``wget`` 实现多层级网页内容爬取  

仅爬取一层的 demo
```py
def scarpy_web(url, storage_path):
    log.info("start crawling: {}".format(url))
    try:
        proc = subprocess.Popen("wget -r -l 1 --no-check-certificate -k --quota=20M {} -P '{}'".format(url, storage_path), shell=True, stdin=subprocess.PIPE,
                            stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        out, err = proc.communicate(timeout=100)  
        r = err.decode()
        o = out.decode()
        if r.find("200") == -1:
            log.error("server down.")
        else:
            pass
    except subprocess.TimeoutExpired:
        pass
    log.info("finished")

    return True
```

### 网页截图  
使用 [Selenium WebDriver](https://www.selenium.dev/) 实现网页截图  

基于 Chrome 的 demo
```py
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument('--start-maximized')
chrome_options.add_argument('--user-agent=Mozilla/5.0 (iPhone; CPU iPhone OS 14_0_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.0 Mobile/15E148 Safari/604.1')

def webScshot(url, urlHash, Opts):
    print("starting web screenshot...")
    try:
        driver = webdriver.Chrome(executable_path='/Users/darenfy/Downloads/chromedriver', options=Opts)
        driver.get(url)
        # Get the actual page dimensions using javascript
        width = driver.execute_script("return Math.max(document.body.scrollWidth,document.body.offsetWidth, document.documentElement.clientWidth, document.documentElement.scrollWidth, document.documentElement.offsetWidth);")	 
        height = driver.execute_script("return Math.max(document.body.scrollHeight, document.body.offsetHeight,document.documentElement.clientHeight, document.documentElement.scrollHeight, document.documentElement.offsetHeight);")
        #resize
        driver.set_window_size(width,height)
        time.sleep(5)
        driver.save_screenshot(urlHash + '.png')

        driver.quit()
    except WebDriverException:
        pass
```

参考资源：
- http://blog.codingcoder.cn/2019/07/14/html-page-to-jpg/
- https://donaldhan.github.io/util/2020/01/31/Selenium(ChromeDriver)%E7%9A%84%E4%B8%80%E6%AC%A1%E8%B6%9F%E5%9D%91.html

### 抓包  
在移动应用分析过程中，往往都是脱壳抓包分析一条龙，抓包是必不可少的内容
可用工具：
- tcpdump
- Wireshark  
- burpsuite  
- Charles  
- Fiddler  

一般网页流量均可以使用这些工具，主要讨论移动端应用，原理基本可以理解为中间人攻击  
以下统称为代理

根据通信方式
- 基于 Socket  
Socket 处于传输层与应用层之间，所以通过应用层协议进行抓包的代理就不能正常工作  
这时能够采用直接抓取网络层包的方式  
- 基于 WebSocket  
WebSocket 基于 HTTP，在 HTTP 一次通信之后便不再使用，所以也可以采用网络层抓包  
- 基于 HTTP  
可以使用代理直接抓取  
- 基于 HTTPS  
同 HTTP，只不过解析流量信息需要密钥

HTTPS 认证
- 仅服务器端证书校验
将代理的根证书置于手机中，便可以正常抓包  
注：根据系统版本的不同可能需要将证书置于用户层或系统层  

- 客户端服务器双向认证  
获取 App 客户端证书，导入代理中，便可正常使用

## 后话  
**其他用处**
我们平时用到域名和端口的机会还是很多的，不提一般资源的获取，平时远程访问也会用到  
- [nc](https://www.runoob.com/linux/linux-comm-nc.html)   
- [ssh](https://phoenixnap.com/kb/ssh-to-connect-to-remote-server-linux-or-windows)

**CTF** 
像在 CTF 中，有时会用到自构造 HTTP 首部，所以搞清楚常用首部内容还是可以加速解题的

举个小🌰：
重点关注内容：Cookie & Session

常用媒体类型
| 媒体类型 | 实体主体描述 |  
| ---| ---|  
| text/html | HTML  
| text/plain | 纯文本  
| image/gif | GIF 格式图像  
| image/jpeg | JPEG 格式图像  
| audio/x-wav | WAV 格式声音数据  
| multipart/form-data | 表单中文件上传
| message/http | 完整 HTTP 报文

内容编码类型
| Content-Encoding 值 | 实体格式描述 |
| --- | --- |
| gzip | GNU zip 编码
| compress | Unix 文件压缩程序  
| deflate | zlib 压缩  
| identify | 没有任何编码（默认）

由于对 URL 解析之后参数格式防御不充分，则可能会使得各种姿势的 SQL 注入等攻击手段奏效

**其他场景需求**
单独使用 URL 分析作防御手段的场景
- URL 相似度/相关性分析  
- 恶意 URL 检测  


**抱住大佬**
不同的环境对应不同的需求，工具也需要更新换代

目前针对移动应用抓包的最好用的办法就是肉丝大佬开发的
[r0capture](https://github.com/r0ysue/r0capture)  
优点众多，我觉得比较重要的
- 作为一款工具，就十分的简单方便，越壳抓包  
- 使用注入的方式将验证函数前后侧进行截取，通用型足够强强  
- 比较亮眼 ✨：能够拉取证书

**总结**  
 
- 学习逆向的过程中，“一切都是数据，只是格式不同”，妙哇～
- 学习协议分析的过程中，“应用运转的方式就是各种通信的结合，搞定通信就能搞定应用”，妙哇～
- 学习抓包的过程中，“无论什么加解密手段，只要你做了这件事，一定有这个函数，只要你有函数，我就能 Hook”，妙哇～

简直是妙蛙种子吃妙脆角，妙到家了

目前在学习证书格式相关内容，学完再写点小笔记

----
### 参考资料
书籍：
- 图解 HTTP  
- HTTP 权威指南   
- HTTP/2 基础教程  
- HTTPS 权威指南

HTTP 笔记
- https://firmianay.gitbooks.io/ctf-all-in-one/content/doc/1.4.2_http_basic.html

HTTP & HTTPS
https://so.csdn.net/so/search?q=HTTP&t=blog&p=1&s=0&tm=0&lv=-1&ft=0&l=&u=qq_34827674


**HTTP 属性**
各版本差异
- https://www.cnblogs.com/andashu/p/6441271.html

Content-Type 列表：
- https://www.iana.org/assignments/media-types/media-types.xhtml  
- https://www.runoob.com/http/http-content-type.html

传输数据方式
socket/WebSocket/WebService/http/https
- https://halfrost.com/ios_weixin_qq_websocket/
- https://www.huaweicloud.com/articles/1ff62a735fddbb09cd89a1ff7b8f9187.html