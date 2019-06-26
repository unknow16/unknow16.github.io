---
title: 04-属性和Profile
toc: true
date: 2019-05-07 20:29:27
tags:
categories:
---


## Maven属性
通常通过<properties>元素自定义maven属性，然后在pom文件的其他地方使用${属性名}的方式引用该属性。

另外maven还有一些内置属性：
1. 内置属性

- ${basedir}  表示项目根目录，即包含pom.xml文件的目录
- ${version} 表示项目版本

2. pom属性: 都对应一个pom元素，有些元素默认值是在超级pom中定义的
- ${project.groupId}
- ${project.artifactId}
- ${project.version}
- 
- ${project.build.finalName} 项目打包输出文件的名称，默认为${project.artifactId}-${project.version}
- ${project.build.sourceDirectory} 项目的主源码目录，默认为src/main/java
- ${project.build.testSourceDirectory} 项目的测试源码目录，默认为src/test/java
- ${project.build.directory} 项目构建输出目录，默认为target/
- 
- ${project.outputDirectory} 项目主代码编译输出目录，默认为target/classes/
- ${project.testOutputDirectory} 项目测试代码编译输出目录，默认为target/test-classes/

3. setting属性：以setting.开头，引用setting.xml文件中的元素，如${setting.localRepository} 指向用户本地仓库的地址
4. Java系统属性：如${user.home}指向用户目录，可用mvn help:system查看所有的Java系统属性
5. 环境变量属性：以env.开头，如${env.JAVA_HOME}指向JAVA_HOME环境变量的值。可用mvn help:system查看所有的环境变量


Maven属性默认只有在pom中才会被解析，如果在src/main/resource/目录下的文件中，构建时不会被解析到，因此需要让maven解析资源文件中的maven属性，资源文件是由maven-resources-plugin来处理的，它的默认行为只是将项目主资源目录文件复制到主代码编译输出目录，将测试资源文件复制到测试代码编译输出目录中。要使它能够解析资源文件中的maven属性，开启资源过滤即可。如下：
```
<resources>
    <resource>
        <directory>${project.basedir}/src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```


## Profile
不同环境采用不同构建配置，一个profiel表示一个特定环境配置，一般会有dev、test、prod等环境配置。

- 激活profile的方式
1. 命令行

mvn的-P参数表示在命令行激活一个profile。如下
```
mvn clean install -Pdev
```

2. setting文件显式激活

表示其配置对所有项目都处于激活状态
```
<setting>
	<activeProfiles>
		<activeProfile>dev</activeProfile>
	</activeProfiles>
</settings>
```

3. 系统属性激活

可以配置当某系统属性存在时，自动激活profile

```
<profiles>
    <profile>
        <activation>
            <property>
                <name>test</name>
            </property>
        </activation>
    </profile>
</profiles>
```
进一步可配置当某系统属性值存在，且值相等时激活

```
<profiles>
    <profile>
        <activation>
            <property>
                <name>test</name>
                <value>x</value>
            </property>
        </activation>
    </profile>
</profiles>
```
此时可通过mvn clean install -Dtest=x设置属性

4. 操作系统环境激活
5. 文件存在与否激活
6. 默认激活

可在定义profile时，指定其默认激活，但如果pom中任何一个profile通过以上任意一种方式被激活了，所有的默认激活配置都将失效。
```
</profiles>
	<profile>
		<id>local</id>
		<activation>
			<activeByDefault>true</activeByDefault>
		</activation>
	</profile>
</profiles>
```

- 查看当前激活的profile

```
mvn help:active-profiles
```

- 列出所有的profile

```
mvn help:all-profiles
```

- profile的种类

根据具体需要，可以在以下位置声明profile

1. pom.xml : 只对当前项目有效
2. 用户setting.xml : 用户目录下.m2/setting.xml中的profile，只对本机该用户的Mavne项目有效
3. 全居setting.xml：maven安装目录下conf/setting.xml中的profile,对本机上的所有Maven项目有效

- 不同类型的profile可声明的pom元素

1. pom.xml中的profile能被提交到代码仓库、安装到本地仓库、部署到远程仓库，所有可存在的元素很多，如下：
```
<repositories></repositories>
<pluginRepositories></pluginRepositories>
<distributionManagement></distributionManagement>

<dependencies></dependencies>
<dependencyManagement></dependencyManagement>

<modules></modules>
<properties></properties>
<reporting></>

<build>
    <plugins></plugins>
    <defaultGoal></defaultGoal>
    <resources></resources>
    <testResources></testResources>
    <finalName></finalName>
</build>
```
2. 在pom.xml外部的profile只能够声明很少的元素
```
<repositories></repositories>
<pluginRepositories></pluginRepositories>
<properties></properties>
```



## 参考资料
> - [Maven实战(许晓斌)]()
