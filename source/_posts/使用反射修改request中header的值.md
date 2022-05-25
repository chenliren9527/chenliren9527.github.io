---
title: 使用反射修改request中header的值
tags:
  - 笔记
  - JavaWeb
categories:
  - JavaWeb
date: 2022-05-25 19:23:39
---
由于业务有统一的鉴权系统，页面请求时在header中带过来gsid，正常业务没有问题，但是当需要下载文件时，前端统一用json解析响应，当响应文件时，对于前端来说不好处理，就决定使用简单的get请求下载文件，将gsid通过url带过来，这样的话后端鉴权就需要处理，当header中没有gsid时，从参数中取，为了尽可能少的改变公用的业务代码（指sso），就在当前项目中自定义权限拦截器。

## 实现方式
新建拦截器类，继承原有的拦截器，重写其preHandle方法
```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object o) throws Exception {
    String gsid = request.getHeader("GSID");
    if(StringUtils.isBlank(gsid)){
        String gsid= request.getParameter("GSID");
        //使用反射，将gsid设置到request中的的header中去
        reflectSetparam(request,"GSID",gsid);
        log.info("请求连接中的gsid={}",request.getHeader("GSID"));
    }
    return super.preHandle(request, response, o);
}
```

1. 先进行header信息判断，如果header中没有GSID，就去请求参数中拿
`gsid= request.getParameter("GSID");`
2. 通过反射将参数中的GSID键值对儿：“GSID”：“376645354562335”加入到header中去

```java
/**
 * 修改header信息，key-value键值对儿加入到header中
 * @param request
 * @param key
 * @param value
 */
private void reflectSetparam(HttpServletRequest request,String key,String value){
    Class<? extends HttpServletRequest> requestClass = request.getClass();
    System.out.println("request实现类="+requestClass.getName());
    try {
        Field request1 = requestClass.getDeclaredField("request");
        request1.setAccessible(true);
        Object o = request1.get(request);
        Field coyoteRequest = o.getClass().getDeclaredField("coyoteRequest");
        coyoteRequest.setAccessible(true);
        Object o1 = coyoteRequest.get(o);
        System.out.println("coyoteRequest实现类="+o1.getClass().getName());
        Field headers = o1.getClass().getDeclaredField("headers");
        headers.setAccessible(true);
        MimeHeaders o2 = (MimeHeaders)headers.get(o1);
        o2.addValue(key).setString(value);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

执行打印信息如下：
```
request实现类=org.apache.catalina.connector.RequestFacade
coyoteRequest实现类=org.apache.coyote.Request
```
## 具体过程如下

### 步骤一：先找到具体的Request对象是哪个类，根据打印信息看源码

进入其中找到getHeader()方法，如下
```java
public String getHeader(String name) {
    if (this.request == null) {
        throw new IllegalStateException(sm.getString("requestFacade.nullRequest"));
    } else {
        return this.request.getHeader(name);
    }
}
```
然后我们找this.request
```java
public class RequestFacade implements HttpServletRequest {
    protected Request request = null;
    protected static final StringManager sm = StringManager.getManager(RequestFacade.class);
```
这个request，我们找到它的类型，点击去
```java
public class Request implements HttpServletRequest {
    private static final String HTTP_UPGRADE_HEADER_NAME = "upgrade";
    private static final Log log = LogFactory.getLog(Request.class);
    protected org.apache.coyote.Request coyoteRequest;
    /** @deprecated */
    @Deprecated
    protected static final TimeZone GMT_ZONE = TimeZone.getTimeZone("GMT");
    protected static final StringManager sm = StringManager.getManager(Request.class);
    protected Cookie[] cookies = null;
    /** @deprecated */
    @Deprecated
    protected final SimpleDateFormat[] formats;
    /** @deprecated */
    @Deprecated
    private static final SimpleDateFormat[] formatsTemplate;
    protected static final Locale defaultLocale;
    private final Map<String, Object> attributes = new ConcurrentHashMap();
    protected boolean sslAttributesParsed = false;
    protected final ArrayList<Locale> locales = new ArrayList();
    private final transient HashMap<String, Object> notes = new HashMap();
    protected String authType = null;
    protected DispatcherType internalDispatcherType = null;

```
这个类的全路径是：org.apache.catalina.connector.Request

这个类中找getHeader方法
```java
public String getHeader(String name) {
    return this.coyoteRequest.getHeader(name);
}
```
找到这个类中的coyoteRequest
`protected org.apache.coyote.Request coyoteRequest;`
是这样的
```java
public final class Request {
    private static final StringManager sm = StringManager.getManager(Request.class);
    private static final int INITIAL_COOKIE_SIZE = 4;
    private int serverPort = -1;
    private final MessageBytes serverNameMB = MessageBytes.newInstance();
    private int remotePort;
    private int localPort;
    private final MessageBytes schemeMB = MessageBytes.newInstance();
    private final MessageBytes methodMB = MessageBytes.newInstance();
    private final MessageBytes uriMB = MessageBytes.newInstance();
    private final MessageBytes decodedUriMB = MessageBytes.newInstance();
    private final MessageBytes queryMB = MessageBytes.newInstance();
    private final MessageBytes protoMB = MessageBytes.newInstance();
    private final MessageBytes remoteAddrMB = MessageBytes.newInstance();
    private final MessageBytes peerAddrMB = MessageBytes.newInstance();
    private final MessageBytes localNameMB = MessageBytes.newInstance();
    private final MessageBytes remoteHostMB = MessageBytes.newInstance();
    private final MessageBytes localAddrMB = MessageBytes.newInstance();
    private final MimeHeaders headers = new MimeHeaders();
    private final Map<String, String> trailerFields = new HashMap();
    private final Map<String, String> pathParameters = new HashMap();
    private final Object[] notes = new Object[32];
    private InputBuffer inputBuffer = null;
    private final UDecoder urlDecoder = new UDecoder();
```
再找到getHeader()
```
public String getHeader(String name) {
    return this.headers.getHeader(name);
}
```
好了，终于见到属性了`private final MimeHeaders headers = new MimeHeaders();`
找到MineHeaders中的getHeader方法，
```java
public String getHeader(String name) {
    MessageBytes mh = this.getValue(name);
    return mh != null ? mh.toString() : null;
}
```
看到最终header是一个MessageBytes对象，好找到这个对象进去，发现只能setValue，那就在MineHeaders中找在哪里实例化MessageBytes对象的

找了半天找到在createHeader（）方法中实例化MimeHeaderField对象，然后这个对象实例化时会实例化MessageBytes对象
```java
class MimeHeaderField {
    private final MessageBytes nameB = MessageBytes.newInstance();
    private final MessageBytes valueB = MessageBytes.newInstance();

    public MimeHeaderField() {
    }

    public void recycle() {
        this.nameB.recycle();
        this.valueB.recycle();
    }

    public MessageBytes getName() {
        return this.nameB;
    }

    public MessageBytes getValue() {
        return this.valueB;
    }

    public String toString() {
        return this.nameB + ": " + this.valueB;
    }
}

```
这里有name，value，靠谱

然后再在MimeHeader中找在addValue方法中有调用，就用这个试一下吧，然后就是最上面的我自定义的方法，在heade中加入了一个键值对儿