---
title: Spring Cloud-07-Zuul-熔断回退
date: 2018-02-27 17:24:50
tags: Spring Cloud
---


### 定义FallbackProvider
- 自定义MyFallbackProvider实现ZuulFallbackProvider接口
- getRoute()返回值为需要熔断回退的服务
```
public class MyFallbackProvider implements ZuulFallbackProvider {
    @Override
    public String getRoute() {
        return "users-service"; //该值为注册中心的服务名
    }

    @Override
    public ClientHttpResponse fallbackResponse() {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("<h1>fallback </h1>".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders httpHeaders = new HttpHeaders();
                httpHeaders.setContentType(MediaType.APPLICATION_JSON);
                return httpHeaders;
            }
        };
    }
}
```

### 全局熔断降级
* 如上一样，只是getRoute() 返回“ * ”或者null即可
