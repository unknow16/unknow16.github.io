---
title: Spring-12-缓存注解
date: 2018-07-11 17:51:55
tags: Spring
---

## @EnableCaching

- @EnableCaching： 启用Cache功能
- CachingConfigurerSupport： 自定义Cache配置

```
@Configuration
@EnableCaching
public class CacheConfig extends CachingConfigurerSupport {

    // 自定义CacheManager配置
    @Bean
	@Override
	public CacheManager cacheManager() {
		// TODO Auto-generated method stub
		return super.cacheManager();
	}
	
	// 自定义KeyGenerator配置
	@Bean
	@Override
	public KeyGenerator keyGenerator() {
		// TODO Auto-generated method stub
		return super.keyGenerator();
	}
}
```

## @CachePut 
应用到写数据的方法上，如新增/修改方法，调用方法时会自动把相应的数据放入缓存： 

```
@CachePut(value = "user", key = "#user.id")  
public User save(User user) {  
    users.add(user);  
    return user;  
}
```
 
即调用该方法时，会把user.id作为key，返回值作为value放入缓存；

@CachePut注解源码如下：
```
public @interface CachePut {  
    String[] value();              //缓存的名字，可以把数据写到多个缓存  
    String key() default "";       //缓存key，如果不指定将使用默认的KeyGenerator生成，后边介绍  
    String condition() default ""; //满足缓存条件的数据才会放入缓存，condition在调用方法之前和之后都会判断  
    String unless() default "";    //用于否决缓存更新的，不像condition，该表达只在方法执行之后判断，此时可以拿到返回值result进行判断了  
}
```

## @CacheEvict 
即应用到移除数据的方法上，如删除方法，调用方法时会从缓存中移除相应的数据：

```
@CacheEvict(value = "user", key = "#user.id") //移除指定key的数据  
public User delete(User user) {  
    users.remove(user);  
    return user;  
}  
@CacheEvict(value = "user", allEntries = true) //移除所有数据  
public void deleteAll() {  
    users.clear();  
}
```
@CacheEvict注解源码如下：
```
public @interface CacheEvict {  
    String[] value();                        //请参考@CachePut  
    String key() default "";                 //请参考@CachePut  
    String condition() default "";           //请参考@CachePut  
    boolean allEntries() default false;      //是否移除所有数据  
    boolean beforeInvocation() default false;//是调用方法之前移除/还是调用之后移除
}
```

## @Cacheable
应用到读取数据的方法上，即可缓存的方法，如查找方法：先从缓存中读取，如果没有再调用方法获取数据，然后把数据添加到缓存中：
```
@Cacheable(value = "user", key = "#id")  
 public User findById(final Long id) {  
     System.out.println("cache miss, invoke find by id, id:" + id);  
     for (User user : users) {  
         if (user.getId().equals(id)) {  
             return user;  
         }  
     }  
     return null;  
 }
```
@Cacheable注解源码如下：
```
public @interface Cacheable {  
    String[] value();             //请参考@CachePut  
    String key() default "";      //请参考@CachePut  
    String condition() default "";//请参考@CachePut  
    String unless() default "";   //请参考@CachePut   
}    
```

如果有@CachePut操作，即使有@Cacheable也不会从缓存中读取；问题很明显，如果要混合多个注解使用，不能组合使用@CachePut和@Cacheable；官方说应该避免这样使用，解释是如果带条件的注解相互排除的场景。

## @Caching
有时候我们可能组合多个Cache注解使用；比如用户新增成功后，我们要添加id-->user；username--->user；email--->user的缓存；此时就需要@Caching组合多个注解标签了。

如用户新增成功后，添加id-->user；username--->user；email--->user到缓存； 

```
@Caching(  
        put = {  
                @CachePut(value = "user", key = "#user.id"),  
                @CachePut(value = "user", key = "#user.username"),  
                @CachePut(value = "user", key = "#user.email")  
        }  
)  
public User save(User user) {
```


@Caching定义如下： 
```
public @interface Caching {  
    Cacheable[] cacheable() default {}; //声明多个@Cacheable  
    CachePut[] put() default {};        //声明多个@CachePut  
    CacheEvict[] evict() default {};    //声明多个@CacheEvict  
}
```

* 自定义缓存注解

比如之前的那个@Caching组合，会让方法上的注解显得整个代码比较乱，此时可以使用自定义注解把这些注解组合到一个注解中，如： 

```
@Caching(  
        put = {  
                @CachePut(value = "user", key = "#user.id"),  
                @CachePut(value = "user", key = "#user.username"),  
                @CachePut(value = "user", key = "#user.email")  
        }  
)  
@Target({ElementType.METHOD, ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Inherited  
public @interface UserSaveCache {  
}
```
 
 
这样我们在方法上使用如下代码即可，整个代码显得比较干净。 


```
@UserSaveCache  
public User save(User user)
```

## @CacheConfig
Spring 4.1后可以直接在类级别使用@CacheConfig指定一些公共配置，如下：

```
@Service  
@CacheConfig(cacheNames = {"user", "user2"})  
public class UserService {  
  
    Set<User> users = new HashSet<User>();  
  
    @CachePut(key = "#user.id")  
    public User save(User user) {  
        users.add(user);  
        return user;  
    }  
    
    @CacheEvict(key = "#user.id")  
    public User delete(User user) {  
        users.remove(user);  
        return user;  
    }
}
```

## Key生成器
如果在Cache注解上没有指定key的话,会使用KeyGenerator进行生成一个key。

* KeyGenerator接口如下：
```
public interface KeyGenerator {  
    Object generate(Object target, Method method, Object... params);  
} 
```
Spring 4之前，默认提供了DefaultKeyGenerator生成器，Spring 4之后使用SimpleKeyGenerator，前者已被标记为过时的。

SimpleKeyGenerator的实现中策略为：
1. 如果只有一个方法参数，就使用其作为key
2. 如果多个参数，则SimpleKey类用Arrays.deepHashCode()把参数列表生成key。

我们也可以自定义自己的key生成器，然后通过xml风格的<cache:annotation-driven key-generator=""/>或注解风格的CachingConfigurer中指定keyGenerator。 

## 示例
* 新增/修改数据时往缓存中写 

```
@Caching(  
        put = {  
                @CachePut(value = "user", key = "#user.id"),  
                @CachePut(value = "user", key = "#user.username"),  
                @CachePut(value = "user", key = "#user.email")  
        }  
)  
public User save(User user)
```

```
@Caching(  
        put = {  
                @CachePut(value = "user", key = "#user.id"),  
                @CachePut(value = "user", key = "#user.username"),  
                @CachePut(value = "user", key = "#user.email")  
        }  
)  
public User update(User user)
```
  
* 删除数据时从缓存中移除

```
@Caching(  
        evict = {  
                @CacheEvict(value = "user", key = "#user.id"),  
                @CacheEvict(value = "user", key = "#user.username"),  
                @CacheEvict(value = "user", key = "#user.email")  
        }  
)  
public User delete(User user)
```
   
```
@CacheEvict(value = "user", allEntries = true)  
 public void deleteAll()
```
 
 
* 查找时从缓存中读


```
@Caching(  
        cacheable = {  
                @Cacheable(value = "user", key = "#id")  
        }  
)  
public User findById(final Long id)
```


```
@Caching(  
         cacheable = {  
                 @Cacheable(value = "user", key = "#username")  
         }  
 )  
 public User findByUsername(final String username)
```



```
@Caching(  
          cacheable = {  
                  @Cacheable(value = "user", key = "#email")  
          }  
  )  
  public User findByEmail(final String email)
```
