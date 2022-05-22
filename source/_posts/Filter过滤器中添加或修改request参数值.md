---
layout: java
title: Filter过滤器中添加或修改request参数值
date: 2022-05-22 21:26:33
tags:
  - 笔记
  - 过滤器
  - HttpServletRequestWrapper
---

在传递数据的过程中,有可能需要对request参数进行修改，而Filter的doFilter方法中的request只有setAttribute而没有setParameter也没有add方法, 所以是没办法直接添加或修改参数的。只能重写HttpServletRequestWrapper, 然后用自己重写的HttpServletRequestWrapper来修改参数, 再替换doFilter的request.原本request参数类型是ServletRequest接口, HttpServletRequestWrapper是ServletRequest的一个实现类.

## 代码实现

### 一、普通参数:ParameterRequestWrapper

```java
import javax.servlet.http.HttpServletRequest;

import javax.servlet.http.HttpServletRequestWrapper;

import java.util.Enumeration;

import java.util.Map;

import java.util.Vector;

/**

 * 普通参数设置

 *

 */

public class ParameterRequestWrapper extends HttpServletRequestWrapper {

    private Map params;

    public ParameterRequestWrapper(HttpServletRequest request, Map newParams) {

        super(request);

        this.params = newParams;

    }

    @Override

    public Map getParameterMap() {

        return params;

    }

    @Override

    public Enumeration getParameterNames() {

        Vector l = new Vector(params.keySet());

        return l.elements();

    }

    @Override

    public String[] getParameterValues(String name) {

        Object v = params.get(name);

        if (v == null) {

            return null;

        } else if (v instanceof String[]) {

            return (String[]) v;

        } else if (v instanceof String) {

            return new String[]{(String) v};

        } else {

            return new String[]{v.toString()};

        }

    }

    @Override

    public String getParameter(String name) {

        Object v = params.get(name);

        if (v == null) {

            return null;

        } else if (v instanceof String[]) {

            String[] strArr = (String[]) v;

            if (strArr.length > 0) {

                return strArr[0];

            } else {

                return null;

            }

        } else if (v instanceof String) {

            return (String) v;

        } else {

            return v.toString();

        }

    }

}
```

### 二、HttpHelper请求处理字符串工具类

```java
import javax.servlet.ServletRequest;

import java.io.BufferedReader;

import java.io.IOException;

import java.io.InputStream;

import java.io.InputStreamReader;

import java.nio.charset.Charset;

/**

 * 请求处理工具类

 *

 */

public class HttpHelper {

    public static String getBodyString(ServletRequest request) {

        StringBuilder sb = new StringBuilder();

        InputStream inputStream = null;

        BufferedReader reader = null;

        try {

            inputStream = request.getInputStream();

            reader = new BufferedReader(new InputStreamReader(inputStream, Charset.forName("UTF-8")));

            String line = "";

            while ((line = reader.readLine()) != null) {

                sb.append(line);

            }

        } catch (IOException e) {

            LogUtils.error(e);

        } finally {

            if (inputStream != null) {

                try {

                    inputStream.close();

                } catch (IOException e) {

                    LogUtils.error(e);

                }

            }

            if (reader != null) {

                try {

                    reader.close();

                } catch (IOException e) {

                    LogUtils.error(e);

                }

            }

        }

        return sb.toString()/*.replaceAll(" ","")*/;

    }

}
```

### 三、实体json参数：RequestWrapper

request.getInputStream()执行一次后（可正常读取body数据），之后再执行就无效了。 所以需要重写获取 输入流的方法，保证流可写可读多次

```java
import com.hean.iot.platform.utils.HttpHelper;

import javax.servlet.ServletInputStream;

import javax.servlet.http.HttpServletRequest;

import javax.servlet.http.HttpServletRequestWrapper;

import java.io.BufferedReader;

import java.io.ByteArrayInputStream;

import java.io.IOException;

import java.io.InputStreamReader;

import java.nio.charset.Charset;

/**

 * 请求参数重写

 *

 */

public class RequestWrapper extends HttpServletRequestWrapper {

    private byte[] body;

    public RequestWrapper(HttpServletRequest request) {

        super(request);

        body = HttpHelper.getBodyString(request).getBytes(Charset.forName("UTF-8"));

    }

    @Override

    public BufferedReader getReader() throws IOException {

        return new BufferedReader(new InputStreamReader(getInputStream()));

    }

    /**

     * 重写获取 输入流的方法，保证流可写可读多次

     * @return

     * @throws IOException

     */

    @Override

    public ServletInputStream getInputStream() throws IOException {

        final ByteArrayInputStream bais = new ByteArrayInputStream(body);

        return new ServletInputStream() {

            @Override

            public int read() throws IOException {

                return bais.read();

            }

        };

    }

    public byte[] getBody() {

        return body;

    }

    public void setBody(byte[] body) {

        this.body = body;

    }

}
```

### 四、过滤器：PostFilter

get请求可以使用普通形式的参数填充，而post请求则需要通过读取输入流填充信息

```java
import com.alibaba.fastjson.JSON;

import com.alibaba.fastjson.JSONObject;

import com.hean.iot.platform.model.RequestWrapper;

import com.hean.iot.platform.session.SessionBeanService;

import org.springframework.context.annotation.Configuration;

import javax.servlet.*;

import javax.servlet.annotation.WebFilter;

import javax.servlet.http.HttpServletRequest;

import java.io.BufferedReader;

import java.io.IOException;

import java.util.HashMap;

/**

 * 过滤器

 *

 */

@Configuration

@WebFilter(filterName = "authFilter", urlPatterns = {"/*"})

public class PostFilter implements Filter {

    @Override

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)

            throws IOException, ServletException {

        /*

         * @Date: 2021/1/18 14:59

         * Step 1: 重写 RequestWrapper，重写获取流的方法

         */

        RequestWrapper requestWrapper = new RequestWrapper((HttpServletRequest) request);

        /*

         * @Date: 2021/1/18 14:59

         * Step 2: 读取输入流，将所需信息写入

         * json形式参数填充（这里新增customerId的键值）

         */

        StringBuffer buffer = new StringBuffer();

        String line = null;

        BufferedReader reader = null;

        reader = requestWrapper.getReader();

        while ((line = reader.readLine()) != null) {

            buffer.append(line);

        }

        JSONObject object = JSON.parseObject(buffer.toString());

        object.put("customerId", SessionBeanService.getCustomerId());

        requestWrapper.setBody(object.toString().getBytes());

        /*

         * @Date: 2021/1/18 15:00

         * Step 3: 普通形式参数填充（这里新增customerId的键值）

         */

        HashMap parameterMap = new HashMap(requestWrapper.getParameterMap());

        parameterMap.put("customerId", new String[]{SessionBeanService.getCustomerId().toString()});

        ParameterRequestWrapper newRequest = new ParameterRequestWrapper(requestWrapper, parameterMap);

        /**

         * 过滤跳转

         */

        chain.doFilter(newRequest, response);

    }

    @Override

    public void destroy() {

    }

    @Override

    public void init(FilterConfig config) throws ServletException {

    }

}
```

### 五、Controller

```java
@RequestMapping("/findDropDown.do")

@ResponseBody

public BaseResult findAlarmGradeDropDown(@RequestBody DropDownDto condition, String customerId) throws Exception {

    return rslt;

}
```

这样，不管是@RequestBody 参数，还是普通参数，都能得到自己设置的值。

也就可以通过这样的方式设置所有请求的公共参数。
