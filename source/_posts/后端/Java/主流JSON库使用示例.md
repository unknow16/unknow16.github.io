---
title: 主流JSON库使用示例
toc: true
date: 2019-08-19 16:16:10
tags:
categories:
---



## Jackson

1. 对象转json

```
	private ObjectMapper mapper = new ObjectMapper();
    
    @Test
    public void testJson() throws JsonProcessingException {
        User user = new User();
        user.setId(8L);
        user.setAge(21);
        user.setName("柳岩");
        user.setUserName("liuyan");
        // 序列化
        String json = mapper.writeValueAsString(user);
        System.out.println("json = " + json);
    }
```

2. json转普通对象

```
    private ObjectMapper mapper = new ObjectMapper();

    @Test
    public void testJson() throws IOException {
        User user = new User();
        user.setId(8L);
        user.setAge(21);
        user.setName("柳岩");
        user.setUserName("liuyan");
        // 序列化
        String json = mapper.writeValueAsString(user);

        // 反序列化，接收两个参数：json数据，反序列化的目标类字节码
        User result = mapper.readValue(json, User.class);
        System.out.println("result = " + result);
    }
```



3. json转集合

   ```
    private ObjectMapper mapper = new ObjectMapper();
   
       @Test
       public void testJson() throws IOException {
           User user = new User();
           user.setId(8L);
           user.setAge(21);
           user.setName("柳岩");
           user.setUserName("liuyan");
   
           // 序列化,得到对象集合的json字符串
           String json = mapper.writeValueAsString(Arrays.asList(user, user));
   
           // 反序列化，接收两个参数：json数据，反序列化的目标类字节码
           List<User> users = mapper.readValue(json, mapper.getTypeFactory().constructCollectionType(List.class, User.class));
           for (User u : users) {
               System.out.println("u = " + u);
           }
       }
   ```
   
4. json转任意复杂类型

```
    private ObjectMapper mapper = new ObjectMapper();

    @Test
    public void testJson() throws IOException {
        User user = new User();
        user.setId(8L);
        user.setAge(21);
        user.setName("柳岩");
        user.setUserName("liuyan");

        // 序列化,得到对象集合的json字符串
        String json = mapper.writeValueAsString(Arrays.asList(user, user));

        // 反序列化，接收两个参数：json数据，反序列化的目标类字节码
        List<User> users = mapper.readValue(json, new TypeReference<List<User>>(){});
        for (User u : users) {
            System.out.println("u = " + u);
        }
    }
```



## FastJSON

1. java对象转 json字符串 

```
	/**
     * java对象转 json字符串 
     */
    @Test
    public void objectTOJson(){
        //简单java类转json字符串
        User user = new User("dmego", "123456");
        String UserJson = JSON.toJSONString(user);
        System.out.println("简单java类转json字符串:"+UserJson);
        
        //List<Object>转json字符串
        User user1 = new User("zhangsan", "123123");
        User user2 = new User("lisi", "321321");
        List<User> users = new ArrayList<User>();
        users.add(user1);
        users.add(user2);
        String ListUserJson = JSON.toJSONString(users);
        System.out.println("List<Object>转json字符串:"+ListUserJson);   
        
        //复杂java类转json字符串
        UserGroup userGroup = new UserGroup("userGroup", users);
        String userGroupJson = JSON.toJSONString(userGroup);
        System.out.println("复杂java类转json字符串:"+userGroupJson);       
        
    }
```

2. json字符串转java对象

```
	/**
     * json字符串转java对象
     * 注：字符串中使用双引号需要转义 (" --> \"),这里使用的是单引号
     */
    @Test
    public void JsonTOObject(){
        /* json字符串转简单java对象
         * 字符串：{"password":"123456","username":"dmego"}*/
        
        String jsonStr1 = "{'password':'123456','username':'dmego'}";
        User user = JSON.parseObject(jsonStr1, User.class);
        System.out.println("json字符串转简单java对象:"+user.toString());
        
        /*
         * json字符串转List<Object>对象
         * 字符串：[{"password":"123123","username":"zhangsan"},{"password":"321321","username":"lisi"}]
         */
        String jsonStr2 = "[{'password':'123123','username':'zhangsan'},{'password':'321321','username':'lisi'}]";
        List<User> users = JSON.parseArray(jsonStr2, User.class);
        System.out.println("json字符串转List<Object>对象:"+users.toString());
            
        /*json字符串转复杂java对象
         * 字符串：{"name":"userGroup","users":[{"password":"123123","username":"zhangsan"},{"password":"321321","username":"lisi"}]}
         * */
        String jsonStr3 = "{'name':'userGroup','users':[{'password':'123123','username':'zhangsan'},{'password':'321321','username':'lisi'}]}";
        UserGroup userGroup = JSON.parseObject(jsonStr3, UserGroup.class);
        System.out.println("json字符串转复杂java对象:"+userGroup);  
    }
```

## GSON

todo



## 参考资料
> - []()
> - []()
