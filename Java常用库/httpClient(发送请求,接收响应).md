[toc]

# HttpClient使用

## 基础知识

### 依赖

```
    <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
      <version>4.5.13</version>
    </dependency>
```

### 使用方法

1. 创建HttpClient对象。
2. 创建请求方法的实例，并指定请求URL。如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
3. 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HttpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。
4. 调用HttpClient对象的**execute(HttpUriRequest request)发送请求**，该方法返回一个HttpResponse。
5. 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。
6. **释放连接**。无论执行方法是否成功，都必须释放连接

## POST

### post请求例子

```
package com.jsq.learns.http.httpClient;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.HttpClient;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;

public class SimplePost {
    public static void main(String[] args) {
    //构建发送的参数
        ArrayList<NameValuePair> formParams = new ArrayList<>();
        formParams.add(new BasicNameValuePair("account", ""));
        formParams.add(new BasicNameValuePair("password", ""));
        try {
        //对发送的参数编码
            HttpEntity httpEntity = new UrlEncodedFormEntity(formParams, "utf-8");
            //设置请求的配置
            //ConnectTimeout：连接一个url的连接等待时间 
            //SocketTimeout：连接上一个url，获取response的返回等待时间
            RequestConfig requestConfig = RequestConfig.custom().setConnectionRequestTimeout(5000).setConnectTimeout(5000).build();
            //构建最简单的HttpClient
            HttpClient client = new DefaultHttpClient();
            //创建一个post，并设置配置和发送的参数
            HttpPost post = new HttpPost("https://www.baidu.com");
            post.setEntity(httpEntity);
            post.setConfig(requestConfig);
            //发送请求
            HttpResponse response = client.execute(post);
            //解析返回响应
            if (response.getStatusLine().getStatusCode()==200){
                String message = EntityUtils.toString(response.getEntity(), "utf-8");
                System.out.println(message);
            }else{
                System.out.println("请求失败");
            }
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

### get请求例子

```
public static void get() {
        CloseableHttpClient httpClient = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet("https://www.baidu.com");
        System.out.println(httpGet.getURI());
        CloseableHttpResponse response = null;
        try {
            response = httpClient.execute(httpGet);
            System.out.println(response.getStatusLine());
            System.out.println(Arrays.toString(response.getAllHeaders()));
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                System.out.println(entity.getContentLength());
                System.out.println(entity.getContentEncoding());
                System.out.println(EntityUtils.toString(entity));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                response.close();
                httpClient.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

