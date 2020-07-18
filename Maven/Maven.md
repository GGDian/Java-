maven 添加私有仓库

```xml
<repositories>
		<repository>
			<id>Xdaoz</id>
			<name>Maven Xdaoz Mirror</name>
			<url>http://maven.xdaoz.com:8081/repository/maven-public/</url>
		</repository>
	</repositories>
```



可以用命令 mvn install 打成一个包，在项目中引入





```xml
<modelVersion>4.0.0</modelVersion>
	<groupId>com.stear.orchid</groupId>
	<artifactId>orchid</artifactId>
	<version>0.1</version>
	<packaging>pom</packaging>
	<name>orchid</name>
	<description>The orchid project of stear</description>
	<url>http://orchid.stear.com</url>
	<modules>
		<module>orchid-server</module>
		<module>orchid-support</module>
  	</modules>
```

这个pom.xml的配置关键在两点：（1）packaging类型为**pom**；（2）<modules>模块的引入。



表示在当前目录（也就是上面pom.xml所在目录）包括一个文件夹orchid-server。如果orchid-server和orchid在同一目录的话，那么配置应该是

```xml
<module>../orchid-server</module>
```



项目继承

```xml
	<modelVersion>4.0.0</modelVersion>
	<parent>
			<groupId>com.stear.orchid</groupId>
			<artifactId>orchid</artifactId>
			<version>1.0-SNAPSHOT</version>
	</parent>
	<artifactId>orchid-support</artifactId>
```

在<parent>中都忽略了<relativePath>标签，<relativePath>的默认值是**../pom.xml**，也就是**从父目录中寻找pom.xml**



如果你的父项目在其他地方，那么要**手工加入**改配置

```xml
<parent>
			<groupId>com.stear.orchid</groupId>
			<artifactId>orchid</artifactId>
			<version>1.0-SNAPSHOT</version>
			<relativePath>parent project orchid directory/pom.xml</relativePath>
	</parent>
```



**`<dependencyManagement/>` 和 `<dependencie/>` 区别是什么？**

- `<dependencyManagement />` ， 统一了 Maven 中依赖的版本号，定义在 `<dependencie />` 中的依赖，在不指定具体版本号时，就会沿着上层找到 `<dependencyManagement />` 中的依赖，并使用它的版本号。这样的话，当有多个子项目引用同一个依赖时，就**不需要重复声明各自的版本号**，只需统一使用 `<dependencyManagement />` 中的版本号即可。
- 还有个不同点，`<dependencyManagement />` 中出现的依赖，并不一定会在项目中使用，而 `<dependencie />` 中的依赖，肯定是包含在项目中的。



**Maven 依赖原则？**

1、**依赖路径**最短优先原则。

2、pom 文件中**申明顺序**优先。

3、覆写优先。



#### Maven 中的仓库分为两种，SNAPSHOT 快照仓库和 RELEASE 发布仓库。

* SNAPSHOT 快照仓库用于保存开发过程中的不稳定版本，RELEASE 正式仓库则是用来保存稳定的发行版本。定义一个组件/模块为快照版本，只需要在 `pom` 文件中在该模块的版本号后加上 `-SNAPSHOT` 即可(注意这里必须是大写)，如下：

```xml
<groupId>cc.mzone</groupId>
<artifactId>m1</artifactId>
<version>0.1-SNAPSHOT</version>
<packaging>jar</packaging>
```

- Maven 会根据模块的版本号(`pom` 文件中的 `version`)中是否带有 `-SNAPSHOT` 来判断是快照版本还是正式版本。
  - 如果是快照版本，那么在 `mvn deploy` 时会自动发布到快照版本库中，会覆盖老的快照版本。而在使用快照版本的模块，在不更改版本号的情况下，直接**编译打包时**，Maven **会自动从镜像服务器上下载**最新的快照版本。
  - 如果是正式发布版本，那么在 `mvn deploy` 时会自动发布到正式版本库中，而使用正式版本的模块，在不更改版本号的情况下，编译打包时如果本地已经存在该版本的模块则不会主动去镜像服务器上下载。

所以，我们在开发阶段，可以**将公用库的版本设置为快照版本**，而被依赖组件则引用**快照版本**进行开发，在公用库的**快照版本更新**后，我们也**不需要修改** pom 文件提示版本号来下载新的版本，直接 mvn 执行相关编译、**打包命令即可重新下载最新的快照库了**，从而也方便了我们进行开发。

### 什么是私服？

私服是一种**特殊的远程仓库**，它是架设在**局域网内**的仓库服务，私服代理广域网上的远程仓库，供局域网内的 Maven 用户使用。