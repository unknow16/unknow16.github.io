---
title: jjwt使用
date: 2018-02-28 11:11:26
tags: Spring Security
---

项目地址：https://github.com/jwtk/jjwt
结合spring boot: https://www.jianshu.com/p/6307c89fe3fa
### dependency:
```
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.9</version>
</dependency>
```
### 生成jwt

```
Map map = new HashMap();
map.put("sub", "haha");
map.put("name", "fff");

String jws = Jwts.builder()
        .setClaims(map)
        .setExpiration(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse("2018-1-18 12:12:00"))
        .signWith(SignatureAlgorithm.HS512, "salt")
        .compact();
System.out.println(jws);

System.out.println(Jwts.parser().setSigningKey("salt").parseClaimsJws(jws).getBody().get("name"));
```
### 解析jwt

```
Claims getClaimsFromToken(String token) {
    Claims claims;
    try {
        claims = Jwts.parser()
                .setSigningKey(secret)
                .parseClaimsJws(token)
                .getBody();  
    } catch (Exception e) {
        claims = null;
    }
    return claims;
}
```

