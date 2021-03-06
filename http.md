可参考：https://developer.mozilla.org/zh-CN/docs/Web/HTTP

HTTP协议工作在应用层，端口号是80。HTTP协议被用于网络中两台计算机间的通信，相比于TCP/IP这些底层协议，HTTP协议更像是高层标记型语言，浏览器根据从服务器得到的HTTP响应体中分别得到报文头，响应头和信息体（HTML正文等），之后将HTML文件解析并呈现在浏览器上。同样，我们在浏览器地址栏输入网址之后，浏览器相当于用户代理帮助我们组织好报文头，请求头和信息体（可选），之后通过网络发送到服务器，服务器根据请求的内容准备数据。所以如果想要完全弄明白HTTP协议，你需要写一个浏览器 + 一个Web服务器，一侧来生成请求信息，一侧生成响应信息。

从网络分层模型来看，HTTP工作在应用层，其在传输层由TCP协议为其提供服务。所以可以猜到，HTTP请求前，客户机和服务器之间一定已经通过三次握手建立起连接，其中套接字中服务器一侧的端口号为HTTP周知端口80。在请求和传输数据时也是有讲究的，通常一个页面上不只有文本数据，有时会内嵌很多图片，这时候有两种选择可以考虑。
- HTTP 1.0：一种是对每一个文件都建立一个TCP连接，传送完数据后立马断开，通过多次这样的操作获取引用的所有数据，但是这样一个页面的打开需要建立多次连接，效率会低很多。
- HTTP 1.1：另一种是对于有多个资源的页面，传送完一个数据后不立即断开连接，在同一次连接下多次传输数据直至传完，但这种情况有可能会长时间占用服务器资源，降低吞吐率。

- HTTP工作流程

    一次完整的HTTP请求事务包含以下四个环节：

    - 建立起客户机和服务器连接。

    - 建立连接后，客户机发送一个请求(**request**)给服务器。

    - 服务器收到请求给予响应(**response**)信息。

    - 客户端浏览器将返回的内容解析并呈现，断开连接。

- HTTP协议结构

    **请求报文**

    对于HTTP请求报文我们可以通过以下两种方式比较直观的看到：一是在浏览器调试模式下（F12）看请求响应信息，二是通过wireshark或者tcpdump抓包实现。通过前者看到的数据更加清晰直观，通过后者抓到的数据更真实。但无论是用哪种方式查看，得到的请求报文主题体信息都是相同的，对于请求报文，主要包含以下四个部分，每一行数据必须通过"\r\n"分割，这里可以理解为行末标识符。

    - Chrome浏览器查看请求报文方法：
        1. 打开调试模式
            F12，当被占用时，通过“选项-更多工具-开发者工具”
        2. 选择 Network
        3. 地址栏输入网址，如 baidu.com，或刷新网页
        4. 点击左侧 name，选择右侧 Request Headers-view source

    - 报文头(start line)

        结构：method  uri  version

        - method

            HTTP的请求方法，一共有9中，但GET和POST占了99%以上的使用频次。

            - **GET** 表示向服务器索取特定资源，所以请求往往没有主体部分,当然也能提交部分数据，它是通过改写URL的方式实现的。GET的数据利用URL?变量名＝变量值的方法传输。比如向 `http://127.0.0.1` 发送一个变量“q”，它的值为“a”。那么，实际的URL为 `http://127.0.0.1?q=a` 。服务器收到请求后，就可以知道"q"的值为"a"。

            - **POST** 通常用于客户端向服务器提交数据，使用POST方法时，URL不再被改写。数据位于http请求的主体。POST方法最用于提交HTML的form数据。服务器往往会对POST方法提交的数据进行一定的处理，比如存入服务器数据库。

        - uri

            - 用来指代请求的文件，指向服务器上的资源的路径，≠URL。

            - URI，全称是 Uniform Resource Identifiers，即统一资源标识符，用于在互联网上标识一个资源，而 URL 是uniform resource locator，即统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源。
            - URL是一种具体的URI，它不仅唯一标识资源，而且还提供了定位该资源的信息。URI 是一种语义上的抽象概念，可以是绝对的，也可以是相对的，而URL则必须提供足够的信息来定位，是绝对的。

            - 完整的 URI，由四个主要的部分构成：

                `<scheme>: //<authority><path>?<query>`
                例如：

                    mailto:mbox@domain
                    ftp://host/file
                    http://domain/path

                - **scheme 表示协议，比如 http，ftp 等等.
                - authority，用 : // 来和 scheme 区分。从字面意思看就是“认证”，“鉴权”的意思，引用 rfc2396#secion-3.2 的一句话：

                    > This authority component is typically defined by an Internet-based server or a  scheme-specific registry of naming authorities.

                    这个“认证”部分，由一个基于 Internet 的服务器定义或者由命名机关注册登记（和具体的协议有关）。

                    而常见的 authority 则是：“由基于 Internet 的服务器定义”，其格式如下：

                    \<userinfo>@\<host>:\<port>

                    userinfo 这个域用于填写一些用户相关的信息，比如可能会填写 “user:password”，当然这是不被建议的。抛开这个不讲，后面的 \<host>:\<port> 则是被熟知的服务器地址了，host 可以是域名，也可以是对应的 IP 地址，port 表示端口，这是一个可选项，如果不填写，会使用默认端口（也是和协议相关，比如 http 协议默认端口是 80）。

                - **path**，在 scheme 和 authority 确定下来的情况下标识资源，path 由几个段组成，每个段用 / 来分隔。注意，path 不等同于文件系统定义的路径。

                - query，查询串（或者说参数串），用 ? 和 path 区分开来，其具体的含义由这个具体资源来定义。


        - version

            HTTP协议的版本，该字段有HTTP/1.0和HTTP/1.1两种。
        - 举例：
        1. 百度主页： `https://www.baidu.com` (等同于 `https://www.baidu.com/`)

            Request header：startline: `GET / HTTP/1.1`

            其中 `path = /` 即默认路径，`host:www.baidu.com` 所以 URI (可能)为：`https://www.baidu.com/index.html`

        2. 搜索 “玉兔” ：`https://www.baidu.com/s?wd=%E7%8E%89%E5%85%94&rsv_spt=1&rsv_iqid=0xfd8fbe58003fcf66&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_dl=tb&rsv_sug3=6&rsv_sug1=4&rsv_sug7=100&rsv_sug2=0&rsv_btype=i&inputT=1799&rsv_sug4=3257`

            Request header：startline: `GET /s?wd=%E7%8E%89%E5%85%94&rsv_spt=1&rsv_iqid=0xfd8fbe58003fcf66&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_dl=tb&rsv_sug3=6&rsv_sug1=4&rsv_sug7=100&rsv_sug2=0&rsv_btype=i&inputT=1799&rsv_sug4=3257 HTTP/1.1`

            其中 `path = /s` , `query = wd=%E7%8E%89%E5%85%94&rsv_spt=1&rsv_iqid=0xfd8fbe58003fcf66&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_dl=tb&rsv_sug3=6&rsv_sug1=4&rsv_sug7=100&rsv_sug2=0&rsv_btype=i&inputT=1799&rsv_sug4=3257 HTTP/1.1`,为请求资源的信息，query 中 `wd=%E7%8E%89%E5%85%94` 为搜索内容“玉兔”的URL编码（UFT-编码加 % ）。

            path 与 query 之间以 `?`分割。


    - 请求头（多行）

        在HTTP/1.1中，请求头除了Host都是**可选**的。包含的头五花八门，这里只介绍部分。

        - Host：指定请求资源的主机（服务器域名）和端口号。端口号默认80。

        - Connection：值为keep-alive和close。keep-alive使客户端到服务器的连接持续有效，不需要每次重连，此功能为HTTP/1.1预设功能。

        - Accept：浏览器可接收的MIME类型。假设为text/html表示接收服务器回发的数据类型为text/html，如果服务器无法返回这种类型，返回406错误。

        - Cache-control：缓存控制，Public内容可以被任何缓存所缓存，Private内容只能被缓存到私有缓存，non-cache指所有内容都不会被缓存。

        - Cookie：将存储在本地的Cookie值发送给服务器，实现无状态的HTTP协议的会话跟踪。

        - Content-Length：请求消息正文长度。

        另有User-Agent、Accept-Encoding、Accept-Language、Accept-Charset、Content-Type等请求头这里不一一罗列。由此可见，请求报文是告知服务器请求的内容，而请求头是为了提供服务器一些关于客户机浏览器的基本信息，包括编码、是否缓存等。

    - 空行（一行）

    - 可选消息体（多行）

    **响应报文**

    响应报文是服务器对请求资源的响应，通过上面提到的方式同样可以看到，同样地，数据也是以"\r\n"来分割。

    - 报文头（一行）

        结构：version status_code status_message

        - version

            描述所遵循的HTTP版本。

        - status_code

            状态码，指明对请求处理的状态，常见的如下。

            - 200：成功。

            - 301：内容已经移动。

            - 302：重新定向(redirect)。

            - 400：请求不能被服务器理解。

            - 403：无权访问该文件。

            - 404：不能找到请求文件。

            - 500：服务器内部错误。

            - 501：服务器不支持请求的方法。

            - 505：服务器不支持请求的版本。

        - status_message

            显示和状态码等价的英文描述，它只是为了便于人类的阅读。电脑只关心三位的状态码(status code)。

    - 响应头（多行）

        这里只罗列部分。

        - Date：表示信息发送的时间。

        - Server：Web服务器用来处理请求的软件信息。

        - Content-Encoding：Web服务器表明了自己用什么压缩方法压缩对象。

        - Content-Length：服务器告知浏览器自己响应的对象长度。

        - Content-Type：告知浏览器响应对象类型。

    - 空行（一行）

    - 信息体（多行）

        实际有效数据，通常是HTML格式的文件，该文件被浏览器获取到之后解析呈现在浏览器中。
    
    **CGI与环境变量**

    **会话机制**

    HTTP作为**无状态协议**，无论是服务器还是客户都不会记住之前的交流。举个例子，仅依靠 HTTP，一个服务器不能记住你输入的密码或者你正处于业务中的哪一步。所以必然需要某种方式保持连接状态。这里简要介绍一下Cookie和Session。

    - 根据早期的HTTP协议（HTTP1.0），每次request-reponse时，都要重新建立TCP连接。TCP连接每次都重新建立，所以服务器无法知道上次请求和本次请求是否来自于同一个客户端。因此，HTTP通信是无状态(stateless)的。服务器认为每次请求都是一 个全新的请求，无论该请求是否来自同一地址。
    - 随着HTTP协议的发展（HTTP1.1），HTTP协议允许TCP连接复用，以节省建立连接所耗费的时间。但HTTP协议依然保持无状态的特性。

    - Cookie

        Cookie是客户端保持状态的方法。

        Cookie简单的理解就是存储由服务器发至客户端并由客户端保存的一段字符串。为了保持会话，服务器可以在响应客户端请求时将Cookie字符串放在Set-Cookie下，客户机收到Cookie之后保存这段字符串，之后再请求时候带上Cookie就可以被识别。

        除了上面提到的这些，Cookie在客户端的保存形式可以有两种，一种是**会话Cookie**，一种是**持久Cookie**。
        - 会话Cookie就是将服务器返回的Cookie字符串保持在内存中，关闭浏览器之后自动销毁。
        - 持久Cookie则是存储在客户端磁盘上，其有效时间在服务器响应头中被指定，在有效期内，客户端再次请求服务器时都可以直接从本地取出。需要说明的是，存储在磁盘中的Cookie是可以被多个浏览器代理所共享的。

    - Session

        Session是服务器保持状态的方法。

        首先需要明确的是，Session保存在服务器上，可以保存在数据库、文件或内存中，每个用户有独立的Session用户在客户端上记录用户的操作。我们可以理解为每个用户有一个独一无二的Session ID作为Session文件的Hash键，通过这个值可以锁定具体的Session结构的数据，这个Session结构中存储了用户操作行为。

        当服务器需要识别客户端时就需要结合Cookie了。每次HTTP请求的时候，客户端都会发送相应的Cookie信息到服务端。实际上大多数的应用都是用Cookie来实现Session跟踪的，第一次创建Session的时候，服务端会在HTTP协议中告诉客户端，需要在Cookie里面记录一个Session ID，以后每次请求把这个会话ID发送到服务器，我就知道你是谁了。如果客户端的浏览器禁用了Cookie，会使用一种叫做URL重写的技术来进行会话跟踪，即每次HTTP交互，URL后面都会被附加上一个诸如sid=xxxxx这样的参数，服务端据此来识别用户，这样就可以帮用户完成诸如用户名等信息自动填入的操作了。
