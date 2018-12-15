# HttpClient4.x使用

## 1. pom 依赖

```
  <!-- HttpClient4.x相关依赖 -->
  <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
      <version>4.5.3</version>
  </dependency>
  <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpmime</artifactId>
      <version>4.5.2</version>
  </dependency>
  <dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
  </dependency>
  <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
      <version>1.1.1</version>
  </dependency>
  <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpcore</artifactId>
  </dependency>

  <!-- gson工具，封装http的时候使用 -->
  <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.8.0</version>
  </dependency>
```

## 2. 封装工具类的使用

#### 封装doGet doPost

```
/**
 * 封装 http get post
 */
public class HttpUtils {

    private static final Gson gson = new Gson();

    /**
     * get方法
     *
     * @param url
     * @return
     */
    public static Map<String, Object> doGet(String url) {
        HashMap<String, Object> map = new HashMap<>();

        CloseableHttpClient httpClient = HttpClients.createDefault();

        // 请求参数设置
        RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(5000) //连接超时
                .setConnectionRequestTimeout(5000) //请求超时
                .setSocketTimeout(5000)
                .setRedirectsEnabled(true) //允许自动重定向
                .build();

        HttpGet httpGet = new HttpGet(url);
        httpGet.setConfig(requestConfig);

        try {
            HttpResponse httpResponse = httpClient.execute(httpGet);
            if (httpResponse.getStatusLine().getStatusCode() == 200) {
                String result = EntityUtils.toString(httpResponse.getEntity());
                // 将返回结果转换成json
                map = gson.fromJson(result, map.getClass());
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                httpClient.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        return map;

    }

    /**
     * 封装post
     *
     * @param url
     * @param data
     * @param timeout
     * @return
     */
    public static String doPost(String url, String data, int timeout) {

        CloseableHttpClient httpClient = HttpClients.createDefault();

        // 请求参数设置
        RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(5000) //连接超时
                .setConnectionRequestTimeout(5000) //请求超时
                .setSocketTimeout(5000)
                .setRedirectsEnabled(true) //允许自动重定向
                .build();

        HttpPost httpPost = new HttpPost(url);
        httpPost.addHeader("content-Type", "text/html;charset=UTF-8");
        httpPost.setConfig(requestConfig);
        if (data != null && data instanceof String) { //使用字符串传参
            //将包含(json|map|object)的字符串 转换成entity
            StringEntity stringEntity = new StringEntity(data, "UTF-8");
            httpPost.setEntity(stringEntity);
        }

        try {
            CloseableHttpResponse httpResponse = httpClient.execute(httpPost);
            if (httpResponse.getStatusLine().getStatusCode() == 200) {
                String result = EntityUtils.toString(httpResponse.getEntity());
                return result;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                httpClient.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }

}
```
