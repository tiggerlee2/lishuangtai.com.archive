---
title: Maven依赖机制
date: 2017-04-30 14:27:43
tags:
---
# Maven

* 简化构建过程: 编译，打包都有对应的插件

* 简化依赖管理: 依赖的传递性

* 提供最佳实践指导: 目录


## Maven Plugins

基于插件，所有任务由下边两种插件完成:

	Build Plugins: <build/>中配置

	Reporting Plugins: <reporting/>中配置
	
每个插件都是由不同的 goals 组成。

\<pluginManagement\>

	* 不是为本身配置插件，为了让子项目继承
	* 子项目如果不显示引入生命的插件，则不会生效
	* 子项目引入插件时，可以覆盖 \<pluginManagement\> 中配置

<!-- more -->

## Maven 生命周期

* lifecycle 
	
	三种: clean, default, site
	每个 lifecycle 由 phases 组成

* phase，生命周期中特定步骤 
	
	* 组成生命周期
	* 按序执行
	* 执行 phase: mvn install
* 几种常见 phase
	* compile 编译源码
	* package 将编译后代码打包
	* install 将项目安装到本地仓库，供其他本地项目使用
	* deploy 将包发布到远程版本库，与其他开发者共享
	
* goal，插件中特定的任务
	* 插件由 goals 组成
	* phase 至少绑定一个 goal 才会生效，不绑定不执行
	* phase 会默认绑定 goal，goal 也会默认绑定 phase
	* goal 单独执行 mvn pmd:check 
	
**注意：** 

单独执行 [goal: mvn pmd:check]，[pmd:check] 默认绑定 [lifecycle phase: verify]，所以 verify 之前的 phases 都会被执行?

	错误，该默认设置生效处是 POM 的 <plugin> 标签中，如果配置了 <goal> 但是没有配置 <phase> 就会以默认的为准
	
### 设置 phases

Packaging
	
	war, jar 等不同的包类型，对应不同的 phases

Plugins

**\<phase\>process-test-resources\</phase\> 没有时，使用 goal 的默认 phase**
	
```
<plugin>
   <groupId>com.mycompany.example</groupId>
   <artifactId>display-maven-plugin</artifactId>
   <version>1.0</version>
   <executions>
     <execution>
       <phase>process-test-resources</phase>
       <goals>
         <goal>time</goal>
       </goals>
     </execution>
   </executions>
</plugin>
```

## POM - Project Object Model

包含 Maven 构建用的项目和详细配置信息


### 项目继承 - Project Inheritence ###

Maven 项目默认继承 Super POM，也可以在 module POM 中设置 <parent> 完成继承。

```
<parent>
    <artifactId>sample-parent</artifactId>
    <groupId>com.sample.dependency.mechanism</groupId>
    <version>1.0-SNAPSHOT</version>
</parent>
```

  	
可继承元素：

	* dependencies
	* developers and contributors
	* plugin lists (including reports)
	* plugin executions with matching ids
	* plugin configuration
	* resources

应用场景:

	多个项目需要共享配置时。
### 项目聚合 - Project Aggregation ###

与 Project Inheritance 不同的是：在 Parent POM 中指定 Module 并且 \<packaging\> 为 pom 类型。

```
<groupId>com.sample.dependency.mechanism</groupId>
<artifactId>sample-parent</artifactId>
<packaging>pom</packaging>
<version>1.0-SNAPSHOT</version>
<modules>
    <module>child-project-x</module>
    <module>child-project-y</module>
</modules>
```

应用场景:
 	
 	多个项目一起构建
 	
### 同时使用 Inheritence 和 Aggregation ###

Intellij 创建的项目，都是这种结构。

	* 在 child POM 中指定  parent
	* 修改 parent POMs <packaging> 为 pom
	* 在 parent POM 中指定 child module 路径

## Maven Profiles ##

根据不同环境，使用不同的参数。

### 激活方式 ###
	
#### 1. 使用 profileName
			
	mvn groupId:artifactId:goal -P profileName1, profileName2
	
	
#### 2. 默认激活

```
<activation>
  <activeByDefault>true</activeByDefault>
</activation>
``` 

#### 3. 根据传入的 property 激活

```
mvn groupId:artifactId:goal -Denvironment=test
```

```
profiles>
  <profile>
    <activation>
      <property>
        <name>environment</name>
        <value>test</value>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```



#### 4. 通过 Maven Settings 文件中 \<activeProfiles\>
		
```
	<profiles>
		<!-- maven 私服配置 -->
		<profile>
			<id>maven-private</id>
			<repositories>
				<!-- artifactory 私服配置 -->
				<repository>
					<id>artifactory-snapshots</id>
					<name>artifactory repository</name>
					<url>http://umavenprivate.mqadmin.site/artifactory/manqian-private</url>
				</repository>
			</repositories>
			<pluginRepositories>
				<pluginRepository>
					<id>artifactory-snapshots</id>
					<name>artifactory plugin repository</name>
					<url>http://umavenprivate.mqadmin.site/artifactory/manqian-private</url>
				</pluginRepository>
			</pluginRepositories>
		</profile>
</profiles>	
<activeProfiles>
	<activeProfile>maven-private</activeProfile>
</activeProfiles>
```

#### 5. 基于环境变量

探测到 jdk1.4 时，自动激活

```
<profiles>
  <profile>
    <activation>
      <jdk>1.4</jdk>
    </activation>
    ...
  </profile>
</profiles>
```

#### 6. 基于操作系统

```
<activation>
  <os>
    <name>Windows XP</name>
    <family>Windows</family>
    <arch>x86</arch>
    <version>5.1.2600</version>
  </os>
</activation>
```

#### 7. 根据文件存在或不存在

```
<activation>
  <file>
    <missing>target/generated-sources/axistools/wsdl2java/org/apache/maven</missing>
  </file>
</activation>
```

### 查看当前生效的 profile

	mvn help:active-profiles
	
	
## 仓库 Repositories
保存构建好的包和依赖

Remote Repository

Local Repository

	缓存远程仓库下载的包；
	
	保存未发布的临时构建包;
	
	.m2/repositories 已有的包不会重新下载。
	
### 加快依赖包下载速度

设置 Mirror

```
<!-- 使用阿里的库作为中央库，提高下载速度-->
<mirrors>
	<mirror>
      <id>aliyun</id>
      <name>aliyun Central</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
</mirrors>
```

使用 Maven 私服

* [Artifactory (Prefer)](https://www.jfrog.com/open-source/)
* [Maven Nexus](http://www.sonatype.org/nexus/)
	


## 依赖机制

* 传递依赖 Transitive dependency - 依赖包的依赖，自动加入

* 依赖调解 Dependency mediation - 由于传递依赖，产品版本冲突时，采用最近声明的版本:
	A->B-C2.0  与 A->C1.0  采用 C1.0
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
		
## 依赖管理 Denpendency Management

集中化管理依赖信息，包括 scope、type (默认为jar)、version，保证信息的一致性，易维护性。

<dependencyManagement> 优先级比依赖调解优先级高。

例如：
	parent pom <dependencyManagement> 指定 A1.0
	B 继承 parent, 项目依赖 C项目
	C 项目指定 A2.0
	最终 B 项目会使用 A1.0


## 导入依赖 Import Dependencies＊

每个项目只能由一个父工程，在大的项目中，所有项目都依赖同一个父工程完成依赖管理不现实。

Maven 2.0.9 之后，可以借助 `<scope>import</scope>` 从别的项目中引入管理的依赖。


**参照:**

[1. 慢钱项目版本管理](https://radevio.com/manqian/manqian-commons-parent/src/master/manqian-dependencies)

[2. Spring Boot](http://docs.spring.io/spring-boot/docs/1.5.3.RELEASE/reference/htmlsingle/#using-boot-maven-without-a-parent) 导入依赖就是使用了该方式。

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


    
