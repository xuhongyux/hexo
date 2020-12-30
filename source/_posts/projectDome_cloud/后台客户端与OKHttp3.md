---
title: 后台客户端与OKHttp3
date: 2019-10-28 11:04:20
categories:
	- projectDome-cloud

---

# MyShopPlus 创建后台客户端

## 概述

创建了微服务以后，我们就可以使用 Vue 开发的客户端来访问我们的服务了，这里采用 [Vue Element Admin](https://github.com/PanJiaChen/vue-element-admin) 作为我们的客户端模板，开发过程中我们使用 [基础模板](https://github.com/PanJiaChen/vue-admin-template) 创建项目并在实际开发中不断完善相关功能

## 构建步骤

```shell
# 克隆项目
git clone https://github.com/PanJiaChen/vue-admin-template.git
# 进入项目目录
cd vue-admin-template
# 安装依赖
npm install --registry=https://registry.npm.taobao.org
# 启动服务
npm run dev
```

浏览器访问 [http://localhost:9528]()

## 发布

```shell
# 构建测试环境
npm run build:stage
# 构建生产环境
npm run build:prod
```

## 其它

```shell
# 预览发布环境效果
npm run preview
# 预览发布环境效果 + 静态资源分析
npm run preview -- --report
# 代码格式检查
npm run lint
# 代码格式检查并自动修复
npm run lint -- --fix
```

# MyShopPlus OKHttp3

## 概述

OKHttp 是一个当前主流的网络请求的开源框架，由 Square 公司开发，用于替代 **HttpUrlConnection** 和 **Apache HttpClient**

## 特性

- 支持 HTTP2，对一台机器的所有请求共享同一个 Socket
- 内置连接池，支持连接复用，减少延迟
- 支持透明的 gzip 压缩响应体
- 通过缓存避免重复的请求
- 请求失败时自动重试主机的其他 IP，自动重定向

## 功能

- PUT，DELETE，POST，GET 等请求
- 文件的上传下载
- 加载图片 (内部会图片大小自动压缩)
- 支持请求回调，直接返回对象、对象集合
- 支持 Session 的保持

## 依赖

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.0.1</version>
</dependency>
```

## 测试请求

### GET

测试 Get 请求

```java
@Test
public void testGet() {
    String url = "https://www.baidu.com";
    OkHttpClient client = new OkHttpClient();
    Request request = new Request.Builder()
        .url(url)
        .build();
    Call call = client.newCall(request);
    try {
        Response response = call.execute();
        System.out.println(response.body().string());
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### POST

测试 POST 请求

```java
@Test
public void testPost() {
    String url = "http://localhost:9001/oauth/token";
    OkHttpClient client = new OkHttpClient();
    RequestBody body = new FormBody.Builder()
        .add("username", "admin")
        .add("password", "123456")
        .add("grant_type", "password")
        .add("client_id", "client")
        .add("client_secret", "secret")
        .build();
    Request request = new Request.Builder()
        .url(url)
        .post(body)
        .build();
    Call call = client.newCall(request);
    try {
        Response response = call.execute();
        System.out.println(response.body().string());
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## 工具类

### OKHttp

为了简化我们的构建过程，可以封装如下工具类

```java
package com.funtl.myshop.plus.commons.utils;
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.MediaType;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;
import java.io.IOException;
import java.util.Iterator;
import java.util.Map;
import java.util.concurrent.TimeUnit;
/**
 * OKHttp3
 * <p>
 * Description:
 * </p>
 *
 * @author Lusifer
 * @version v1.0.0
 * @date 2019-07-29 14:05:08
 * @see com.funtl.myshop.plus.commons.utils
 */
public class OkHttpClientUtil {
    private static final int READ_TIMEOUT = 100;
    private static final int CONNECT_TIMEOUT = 60;
    private static final int WRITE_TIMEOUT = 60;
    private static final MediaType JSON = MediaType.parse("application/json; charset=utf-8");
    private static final byte[] LOCKER = new byte[0];
    private static OkHttpClientUtil mInstance;
    private OkHttpClient okHttpClient;
    private OkHttpClientUtil() {
        okhttp3.OkHttpClient.Builder clientBuilder = new okhttp3.OkHttpClient.Builder();
        // 读取超时
        clientBuilder.readTimeout(READ_TIMEOUT, TimeUnit.SECONDS);
        // 连接超时
        clientBuilder.connectTimeout(CONNECT_TIMEOUT, TimeUnit.SECONDS);
        //写入超时
        clientBuilder.writeTimeout(WRITE_TIMEOUT, TimeUnit.SECONDS);
        okHttpClient = clientBuilder.build();
    }
    /**
     * 单例模式获取 NetUtils
     *
     * @return {@link OkHttpClientUtil}
     */
    public static OkHttpClientUtil getInstance() {
        if (mInstance == null) {
            synchronized (LOCKER) {
                if (mInstance == null) {
                    mInstance = new OkHttpClientUtil();
                }
            }
        }
        return mInstance;
    }
    /**
     * GET，同步方式，获取网络数据
     *
     * @param url 请求地址
     * @return {@link Response}
     */
    public Response getData(String url) {
        // 构造 Request
        Request.Builder builder = new Request.Builder();
        Request request = builder.get().url(url).build();
        // 将 Request 封装为 Call
        Call call = okHttpClient.newCall(request);
        // 执行 Call，得到 Response
        Response response = null;
        try {
            response = call.execute();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return response;
    }
    /**
     * POST 请求，同步方式，提交数据
     *
     * @param url        请求地址
     * @param bodyParams 请求参数
     * @return {@link Response}
     */
    public Response postData(String url, Map<String, String> bodyParams) {
        // 构造 RequestBody
        RequestBody body = setRequestBody(bodyParams);
        // 构造 Request
        Request.Builder requestBuilder = new Request.Builder();
        Request request = requestBuilder.post(body).url(url).build();
        // 将 Request 封装为 Call
        Call call = okHttpClient.newCall(request);
        // 执行 Call，得到 Response
        Response response = null;
        try {
            response = call.execute();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return response;
    }
    /**
     * GET 请求，异步方式，获取网络数据
     *
     * @param url       请求地址
     * @param myNetCall 回调函数
     */
    public void getDataAsync(String url, final MyNetCall myNetCall) {
        // 构造 Request
        Request.Builder builder = new Request.Builder();
        Request request = builder.get().url(url).build();
        // 将 Request 封装为 Call
        Call call = okHttpClient.newCall(request);
        // 执行 Call
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                myNetCall.failed(call, e);
            }
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                myNetCall.success(call, response);
            }
        });
    }
    /**
     * POST 请求，异步方式，提交数据
     *
     * @param url        请求地址
     * @param bodyParams 请求参数
     * @param myNetCall  回调函数
     */
    public void postDataAsync(String url, Map<String, String> bodyParams, final MyNetCall myNetCall) {
        // 构造 RequestBody
        RequestBody body = setRequestBody(bodyParams);
        // 构造 Request
        buildRequest(url, myNetCall, body);
    }
    /**
     * 同步 POST 请求，使用 JSON 格式作为参数
     *
     * @param url  请求地址
     * @param json JSON 格式参数
     * @return 响应结果
     * @throws IOException 异常
     */
    public String postJson(String url, String json) throws IOException {
        RequestBody body = RequestBody.create(json, JSON);
        Request request = new Request.Builder()
                .url(url)
                .post(body)
                .build();
        Response response = okHttpClient.newCall(request).execute();
        if (response.isSuccessful()) {
            return response.body().string();
        } else {
            throw new IOException("Unexpected code " + response);
        }
    }
    /**
     * 异步 POST 请求，使用 JSON 格式作为参数
     *
     * @param url       请求地址
     * @param json      JSON 格式参数
     * @param myNetCall 回调函数
     * @throws IOException 异常
     */
    public void postJsonAsync(String url, String json, final MyNetCall myNetCall) throws IOException {
        RequestBody body = RequestBody.create(json, JSON);
        // 构造 Request
        buildRequest(url, myNetCall, body);
    }
    /**
     * 构造 POST 请求参数
     *
     * @param bodyParams 请求参数
     * @return {@link RequestBody}
     */
    private RequestBody setRequestBody(Map<String, String> bodyParams) {
        RequestBody body = null;
        okhttp3.FormBody.Builder formEncodingBuilder = new okhttp3.FormBody.Builder();
        if (bodyParams != null) {
            Iterator<String> iterator = bodyParams.keySet().iterator();
            String key = "";
            while (iterator.hasNext()) {
                key = iterator.next().toString();
                formEncodingBuilder.add(key, bodyParams.get(key));
            }
        }
        body = formEncodingBuilder.build();
        return body;
    }
    /**
     * 构造 Request 发起异步请求
     *
     * @param url       请求地址
     * @param myNetCall 回调函数
     * @param body      {@link RequestBody}
     */
    private void buildRequest(String url, MyNetCall myNetCall, RequestBody body) {
        Request.Builder requestBuilder = new Request.Builder();
        Request request = requestBuilder.post(body).url(url).build();
        // 将 Request 封装为 Call
        Call call = okHttpClient.newCall(request);
        // 执行 Call
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                myNetCall.failed(call, e);
            }
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                myNetCall.success(call, response);
            }
        });
    }
    /**
     * 自定义网络回调接口
     */
    public interface MyNetCall {
        /**
         * 请求成功的回调处理
         *
         * @param call     {@link Call}
         * @param response {@link Response}
         * @throws IOException 异常
         */
        void success(Call call, Response response) throws IOException;
        /**
         * 请求失败的回调处理
         *
         * @param call {@link Call}
         * @param e    异常
         */
        void failed(Call call, IOException e);
    }
}
```

### Jackson

请求的响应结果通常是 JSON 数据格式，此时我们还可以封装一个 Jackson 工具类，简化我们的 JSON 解析过程

```java
package com.funtl.myshop.plus.commons.utils;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
/**
 * Jackson 工具类
 * <p>Title: MapperUtils</p>
 * <p>Description: </p>
 *
 * @author Lusifer
 * @version 1.0.0
 * @date 2018/3/4 21:50
 */
public class MapperUtils {
    private final static ObjectMapper objectMapper = new ObjectMapper();
    public static ObjectMapper getInstance() {
        return objectMapper;
    }
    /**
     * 转换为 JSON 字符串
     *
     * @param obj
     * @return
     * @throws Exception
     */
    public static String obj2json(Object obj) throws Exception {
        return objectMapper.writeValueAsString(obj);
    }
    /**
     * 转换为 JSON 字符串，忽略空值
     *
     * @param obj
     * @return
     * @throws Exception
     */
    public static String obj2jsonIgnoreNull(Object obj) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        return mapper.writeValueAsString(obj);
    }
    /**
     * 转换为 JavaBean
     *
     * @param jsonString
     * @param clazz
     * @return
     * @throws Exception
     */
    public static <T> T json2pojo(String jsonString, Class<T> clazz) throws Exception {
        objectMapper.configure(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY, true);
        return objectMapper.readValue(jsonString, clazz);
    }
    /**
     * 字符串转换为 Map<String, Object>
     *
     * @param jsonString
     * @return
     * @throws Exception
     */
    public static <T> Map<String, Object> json2map(String jsonString) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        return mapper.readValue(jsonString, Map.class);
    }
    /**
     * 字符串转换为 Map<String, T>
     */
    public static <T> Map<String, T> json2map(String jsonString, Class<T> clazz) throws Exception {
        Map<String, Map<String, Object>> map = objectMapper.readValue(jsonString, new TypeReference<Map<String, T>>() {
        });
        Map<String, T> result = new HashMap<String, T>();
        for (Map.Entry<String, Map<String, Object>> entry : map.entrySet()) {
            result.put(entry.getKey(), map2pojo(entry.getValue(), clazz));
        }
        return result;
    }
    /**
     * 深度转换 JSON 成 Map
     *
     * @param json
     * @return
     */
    public static Map<String, Object> json2mapDeeply(String json) throws Exception {
        return json2MapRecursion(json, objectMapper);
    }
    /**
     * 把 JSON 解析成 List，如果 List 内部的元素存在 jsonString，继续解析
     *
     * @param json
     * @param mapper 解析工具
     * @return
     * @throws Exception
     */
    private static List<Object> json2ListRecursion(String json, ObjectMapper mapper) throws Exception {
        if (json == null) {
            return null;
        }
        List<Object> list = mapper.readValue(json, List.class);
        for (Object obj : list) {
            if (obj != null && obj instanceof String) {
                String str = (String) obj;
                if (str.startsWith("[")) {
                    obj = json2ListRecursion(str, mapper);
                } else if (obj.toString().startsWith("{")) {
                    obj = json2MapRecursion(str, mapper);
                }
            }
        }
        return list;
    }
    /**
     * 把 JSON 解析成 Map，如果 Map 内部的 Value 存在 jsonString，继续解析
     *
     * @param json
     * @param mapper
     * @return
     * @throws Exception
     */
    private static Map<String, Object> json2MapRecursion(String json, ObjectMapper mapper) throws Exception {
        if (json == null) {
            return null;
        }
        Map<String, Object> map = mapper.readValue(json, Map.class);
        for (Map.Entry<String, Object> entry : map.entrySet()) {
            Object obj = entry.getValue();
            if (obj != null && obj instanceof String) {
                String str = ((String) obj);
                if (str.startsWith("[")) {
                    List<?> list = json2ListRecursion(str, mapper);
                    map.put(entry.getKey(), list);
                } else if (str.startsWith("{")) {
                    Map<String, Object> mapRecursion = json2MapRecursion(str, mapper);
                    map.put(entry.getKey(), mapRecursion);
                }
            }
        }
        return map;
    }
    /**
     * 将 JSON 数组转换为集合
     *
     * @param jsonArrayStr
     * @param clazz
     * @return
     * @throws Exception
     */
    public static <T> List<T> json2list(String jsonArrayStr, Class<T> clazz) throws Exception {
        JavaType javaType = getCollectionType(ArrayList.class, clazz);
        List<T> list = (List<T>) objectMapper.readValue(jsonArrayStr, javaType);
        return list;
    }
    /**
     * 获取泛型的 Collection Type
     *
     * @param collectionClass 泛型的Collection
     * @param elementClasses  元素类
     * @return JavaType Java类型
     * @since 1.0
     */
    public static JavaType getCollectionType(Class<?> collectionClass, Class<?>... elementClasses) {
        return objectMapper.getTypeFactory().constructParametricType(collectionClass, elementClasses);
    }
    /**
     * 将 Map 转换为 JavaBean
     *
     * @param map
     * @param clazz
     * @return
     */
    public static <T> T map2pojo(Map map, Class<T> clazz) {
        return objectMapper.convertValue(map, clazz);
    }
    /**
     * 将 Map 转换为 JSON
     *
     * @param map
     * @return
     */
    public static String mapToJson(Map map) {
        try {
            return objectMapper.writeValueAsString(map);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "";
    }
    /**
     * 将 JSON 对象转换为 JavaBean
     *
     * @param obj
     * @param clazz
     * @return
     */
    public static <T> T obj2pojo(Object obj, Class<T> clazz) {
        return objectMapper.convertValue(obj, clazz);
    }
    /**
     * 将指定节点的 JSON 数据转换为 JavaBean
     *
     * @param jsonString
     * @param clazz
     * @return
     * @throws Exception
     */
    public static <T> T json2pojoByTree(String jsonString, String treeNode, Class<T> clazz) throws Exception {
        JsonNode jsonNode = objectMapper.readTree(jsonString);
        JsonNode data = jsonNode.findPath(treeNode);
        return json2pojo(data.toString(), clazz);
    }
    /**
     * 将指定节点的 JSON 数组转换为集合
     *
     * @param jsonStr  JSON 字符串
     * @param treeNode 查找 JSON 中的节点
     * @return
     * @throws Exception
     */
    public static <T> List<T> json2listByTree(String jsonStr, String treeNode, Class<T> clazz) throws Exception {
        JsonNode jsonNode = objectMapper.readTree(jsonStr);
        JsonNode data = jsonNode.findPath(treeNode);
        return json2list(data.toString(), clazz);
    }
}
```

