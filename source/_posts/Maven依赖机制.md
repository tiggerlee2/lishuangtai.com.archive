---
title: Maven依赖机制
date: 2017-04-30 14:27:43
tags:
---
# Maven依赖管理

本文主要介绍 Maven依赖管理 的基础，以及借助 Maven2.0.9 之后新增 import dependencies 实现依赖信息集中化管理 


## 基础介绍

* 依赖传递 Transitive dependency 
	
		依赖包的依赖，自动加入依赖树 - 这是 Maven依赖机制 的核心。

* 依赖调解 Dependency mediation 

		由于传递依赖，产品版本冲突时，采用最近声明的版本:
		A->B-C2.0  与 A->C1.0  采用 C1.0

<!-- more -->

* 依赖范围 Dependency Scope

	___尽量设置 Scope，避免引入不必要的依赖包___
		
		* compile
		
			默认，该依赖会传递。
		* provided
		
			容器或者jdk已经提供，该依赖只用在 compile 和 test, 不传递。
		* runtime

			只在运行时需要，在 compile 时不会加入 compile classpath，传递。
		* test

			test compile 和 execution 生效，不传递。
		* system
	
			不推荐
		* import	 
		
			见下节
		
* dependencyManagement

	* 用于维护依赖信息: scope, type, version
	* 比依赖调解优先级高 例如：
	
			parent pom <dependencyManagement> 指定 A1.0
			B 继承 parent, 项目依赖 C项目
			C 项目指定 A2.0
			最终 B 项目会使用 A1.0


## 依赖管理实践

为了保证依赖信息的一致性与可维护性，需要将**依赖信息集中化管理**。依赖信息不仅仅包括 version，还包括：scope, type。

具体操作：

1. 创建依赖信息管理项目，\<packaging\>为 pom;
2. 在 \<dependencyManagement\> 维护依赖信息
3. 使用处借助 `<scope>import</scope>` 引入之前创建的项目, 只要指定依赖的 groupId 与 artifactId 即可。
	


**参照:**

[Spring Boot](http://docs.spring.io/spring-boot/docs/1.5.3.RELEASE/reference/htmlsingle/#using-boot-maven-without-a-parent) 导入依赖就是使用了该方式。

```
<dependencyManagement>
     <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```




	
## 参考链接
* [Maven 依赖机制](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
* [Maven 官方文档](https://maven.apache.org/guides/index.html)
* [Maven 常用插件](https://maven.apache.org/plugins/index.html)


    