---
title: Spring-04-Resource体系
date: 2018-03-01 09:38:47
tags: Spring
---

### Resource接口
Spring的Resource接口代表了对底层外部资源的抽象，提供了对底层外部资源的统一访问接口。

```
public interface InputStreamSource {
    InputStream getInputStream() throws IOException;
}


public interface Resource extends InputStreamSource {
	boolean exists();
	boolean isReadable();
	boolean isOpen();
	URL getURL() throws IOException;
	URI getURI() throws IOException;
	File getFile() throws IOException;
	long contentLength() throws IOException;
	long lastModified() throws IOException;
	Resource createRelative(String relativePath) throws IOException;
	String getFilename();
	String getDescription();
}

```
默认实现有:
* ByteArrayResource
* InputStreamResource
* ClassPathResource
* UrlResource
* ServletContextResource
* FileSystemResource
* VfsResource

### ResourceLoader接口
该接口用于返回一个Resource对象，它的实现可以看作是一个Resouce的工厂类。

```
public interface ResourceLoader {
	Resource getResource(String location); //根据location参数返回相应的Resource
	ClassLoader getClassLoader(); //返回加载这些Resource的ClassLoader
}

```
默认实现有：
* DefaultResourceLoader: 可返回ClassPathResource,UrlResource
* ServletContextResourceLoader: 继承前者，增加支持返回ServletContextResource
* ApplicationContext接口继承了该接口，AbstractApplicationContext类继承了DefaultResourceLoader，默认可以使用来加载资源

### ResourceLoaderAware接口
该接口为一个标记接口，用于通过ApplicationContext注入一个ResourceLoader

```
public interface ResourceLoaderAware {
   void setResourceLoader(ResourceLoader resourceLoader);
}
```

### ResourcePatternResolver接口
继承自ResourceLoader接口，新增一个用来加载多个Resource方法
```
public interface ResourcePatternResolver extends ResourceLoader {
	String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
	Resource[] getResources(String locationPattern) throws IOException;
}
```
默认实现有：
* PathMatchingResourcePatternResolver: 基于模式匹配，默认采用AntPathMatcher进行路径匹配。除了支持ResourceLoader支持的前缀，还支持“classpath*:”,来加载所有匹配的类路径下Resource.

