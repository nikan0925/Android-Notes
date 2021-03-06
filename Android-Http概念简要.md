##网络编程整理

###Android网络编程概述
***
网络编程:http,socket,主流是http
http,socket两者有什么区别?(面试题)

http是一种无状态的协议,就是每次发现胡请求第一步都是客户端与服务端建立连接,连接成功之后吧数据传过去,服务器收到数据进行响应,然后再回传结果给客户端
这一次请求之后连接就断开了,下次再发新的请求就会再重新建立连接

Socket是长连接,发出数据之后,连接不会断开,一直保持连接
	
**优缺点**
	
Socket,一直保持连接,高效   很多人同时连接,耗费带宽

http,

**应用场景**
	
Socket,即时通讯(IM),类似微信 QQ

http,浏览器网页,大多数手机App(请求数据渲染界面,页面渲染完就结束了)

###请求和响应
***	
请求和响应(request和response)
请求:比如手机端发送一个网络请求,浏览器输入一个网址
request包含请求的地址,参数,方法(Method),请求头(header)

参数包含两块.对比get和post  
get:直接跟在网址后面如:http://www.baidu.com/?key1=value&key2=value2(参数的一种形式,query params)
一般来说一个url (就比如是http://www.baidu.com)问号后面的都是参数

post:网页提交表单,比如用户名,密码提交到服务器,这种参数是不会直接显示的,通过body的方式传递给服务端(body params),默认是key-value的键值对的样式

http请求的方法包含get post delete put head patch trace options 总共8种

除get外,其他6中都是基于post方法衍生的,
最常见的是get和post
了解get post put delete 这四种最重要,对应数据库的增删改查

**方法**

方法(Method)
查询学生数量,增加学生,修改学生信息,删除学生信息
	
* get /api/student/index                   查询接口
* post /api/student/ 携带一些参数           查询接口
* post  /api/student/update  携带一些参数   修改接口
* post  /api/student/delete 携带参数        删除接口
	
上述只用了 get 和 post 方法，然后 api 的 url 定义的不太一样
但如果使用restful方法,就会变成下面这样
	
* get  /api/student/index  查询接口 不变
* post  /api/student/   携带一些 params 是创建接口  不变
* delete  /api/student/携带参数  删除接口
	
	修改跟删除直接用 http 的 method 来指定，url 哪怕一样，就可以轻松知道你的动作
	这就是所谓的 restful 概念最重要的一个理念

**请求头**
	
请求头(header)
作用分类 为下面两类

分类1.客户端告诉服务端的信息举例:通过header传递,都是key-value的形式
		
* 我发起的请求是什么语言
* 我发起的请求带的参数的格式,是json还是哈希
* 我发起的请求需不需要带缓存
* 我发起的请求需要压缩等等 

分类2.服务端告诉客户端,同样在header里传递

* 返回数据的格式
* 我这次请求是不支持缓存的
*我这次返回的数据没有压缩等

	header里的信息都是一个个的哈希，即key value对
	比如大家常见的一些 header里的信息(Content-Type: application/json Cache-Control : true )
	header里默认的有很多字段，这是规定
	当然你也可以自定义一些字段在里面传递,只要跟你们服务端约定好就行
	
	
	关于Connection: keep-alive 字段的介绍
	http能不能实现socket一样的
	http如果每次都请求断开未免太没效率了
	后来有人就发现  一般我在发起一个请求之后，可能紧接着就发起第二个请求了
	如果频繁的断开重连效率太低了于是就有了这个字段
	这个字段顾名思义，就是声明连接的类型
	这个字段有两个值close 和 keep-live
	如果你指定close,那么每次请求就会立马断开连接
	如果你指明了 keep-live ,就可以在一段时间内保持长连接
	这个时间服务端可以自行配置,但是时间也不会太长，不然服务器会很吃紧
	比如设置个30s,意味着这30s的时间内不会重新断开重连
	
	header里有很多字段  只不过你们平时基本不会用到
	比如 声明是否压缩，比如是否支持缓存，比如缓存过期时间，比如系统时间等等

**响应**	

response除了header外还会有返回值(比如请求百度,返回的就是一段 html代码,只不过浏览器渲染成了一个页面而已)

App中,目前大部分的response都是json格式的数据

json数据解析器:Android中有各种各样的json数据解析器如gson、jackson、FastJson等等

除了返回的数据之外还有一个比较重要的叫做状态码( http  status),一般是 200—500

常见的状态码
正常的时候返回的就是200   404:表示找不到  500:服务器挂了

一般是 200开头的大多是成功  
 
30x代表重定向

304比较特殊,是服务端告诉你内容没有改变，意为缓存的内容

40x一般是请求找不到，大部分是你url写错了

50x那就是服务器挂了

###http缓存
***
缓存是跟开发中最常见的一环,可以算是性能优化的一个大点

缓存有两部分，客户端和服务端

缓存有两种机制
	
**第一种最常见的就是时间缓存**
		
	例子说明:比如我查询中国的城市信息,中国的城市信息可能很久都不会变更,对于这部分假设我就设置7天的缓存时间,
	意味着客户端每次请求，服务端一次查询数据库，7天之内都不会再查询数据库，直接返回给你上一次的结果
	对应的 状态码(status code) 就是304,这对服务器性能优化是很大的一环
	
	缓存是否生效的判断方式:
	缓存有没有生效一般用 header里的 cache-control 字段控制
	如果客户端传递 cache-control 字段为 true 那么服务端有缓存的话就会启动缓存
	如果传递 false 那么会强制服务端每次都去数据库里查询
	
	缓存的时间:
	缓存的时间有一个字段:expired 告诉你缓存过期的时间(性能优化的一个点)
	客户端发起一个请求,服务端返回数据，并且告诉客户端我这个请求有缓存的，缓存时间是7天,
	那么客户端就可以把这个请求结果缓存在本地，然后7天之内都不再发起请求，直接读取本地的缓存数据，那么会极大的节省服务器资源,
	那么客户端就可以把这个请求结果缓存在本地，然后7天之内都不再发起请求，直接读取本地的缓存数据，那么会极大的节省服务器资源

		
**第二种缓存叫做 Etag 缓存**

	为什么要有Etag缓存
	因为以时间为缓存的大多为那些请求数据不常变化的,但大多数场景是数据经常变化,比如我刷个微博

	Etag 你们可以理解成就是一个唯一的字符串,下次请求客户端要携带这个 Etag 传递给服务端
	
	这个字符串是服务端根据时间、用户标识、内容索引等生成的一个字符串,一旦用户的内容变化，这个Etag就会变化,如果用户内容不变化，那么Etag也不会变化
	
	刷新微博的机制
	客户端发起一个请求,
	服务端返回了一个数据，告诉客户端，我是有缓存的，缓存的 Etag 值也返回给你了，然后我把返回的数据以及 Etag 缓存到本地,这样下次请求的话
	如果数据没有变化，那么服务端不会返回给你数据了，而只会返回一个 Etag值,拿到这个Etag值之后到本地去找这个Etag缓存对应的数据,直接渲染
	
	etag的生成与比较都是在服务端做的,直接比较 Etag 是不是一样就好了
	
	主流网络库都会默认支持这两种缓存
	
	提问:只有本地缓存和服务器内容完全一致，etag字符才会一致？
	以volley为例
	他内部实现了缓存机制
	每次请求之后如果服务端支持缓存的话，他自己只直接在本地进行缓存的
	也就是说  如果服务端返回了 304 ，并返回了etag,volley会自动去本地寻找上次缓存过的数据进行渲染
	这些他内部进行了实现，你们不需要关心,但是大家要知道这个原理与机制
	
	提问:客户端一般使用什么方法来做缓存呢？数据库吗？
	一般都是文件缓存，比如volley就是用数据库,数据库也不是不可以，但是有点大材小用，简单的缓存直接文件缓存就搞定了
	volley的缓存机制是这样的 你发起一个请求，服务端有缓存了,然后缓存在本地，下次发起请求，服务端返回 304 ，那么volley就默认直接到本地缓存去找
	问题是，如果用户没有网,那么发起请求，服务端根本没有响应,这时候volley默认的是不知道去本地去找的
	因为他没有接受到304的信号,所以说如果想要没网的时候去本地读取缓存，这样用户体验会更好点,这个时候需要自己去做处理的
	
	第一次也没有缓存，服务端会直接返回给你数据以及etag
	第二次之后会把etag请求给服务端
	但是把etag提交给服务端网络库会自动携带的

***	
**补充说明:** 其实最好的用户体验是这样的：
	用户打开一个页面，先判断本地缓存有没有，有的话直接显示
	然后显示下拉刷新状态去请求，获取到最新的数据再刷新页面，如果没有数据就不做处理
	也就是说用户一进来最好能让他看到一些数据做占位符,这个才是最好的用户体验
	
	

[百度百科-RESTfu](http://baike.baidu.com/link?url=6POcQHUGjOIvrKXjxEplbj9gMv3aTiWBl8woZOOMlSaEe68ZadSpomqWX1muJBHK_gAIZ-jmNNJKUV9ecC2Jka)

[github群友共享-学习资料整理](https://github.com/Freelander/Android_Data)

需翻墙查看[android官网 volley training](https://developer.android.com/training/volley/index.html)

[浅谈Http协议](http://gityuan.com/2015/06/20/http-agreement/)