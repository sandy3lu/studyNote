
# 简介
Maven仓库就是放置所有JAR文件（WAR，ZIP，POM等等）的地方，所有Maven项目可以从同一个Maven仓库中获取自己所需要的依赖JAR，这节省了磁盘资源。此外，由于Maven仓库中所有的JAR都有其自己的坐标，该坐标告诉Maven它的组ID，构件ID，版本，打包方式等等，因此Maven项目可以方便的进行依赖版本管理

## 仓库
运行Maven的时候，Maven所需要的任何构件都是直接从本地仓库获取的。如果本地仓库没有，它会首先尝试从远程仓库下载构件至本地仓库，然后再使用本地仓库的构件。如果尝试过所有远程仓库之后，Maven还是没能够下载到该文件，它就会报错。
Maven缺省的本地仓库地址为`${user.home}/.m2/repository` 。也就是说，一个用户会对应的拥有一个本地仓库。

### 配置本地仓库
可以自定义本地仓库的位置，修改${user.home}/.m2/settings.xml ：
```xml
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository-->
  <localRepository>D:/.m2/repository</localRepository>
```
还可以在运行时指定本地仓库位置：
mvn clean install -Dmaven.repo.local=/home/juven/myrepo/

> 当我们运行install的时候，Maven实际上是将项目生成的构件安装到了本地仓库，也就是说，只有install了之后，其它项目才能使用此项目生成的构件。


### 默认中央仓库
maven安装目录下的：/lib/maven-model-builder-${version}.jar中

打开该文件，能找到超级POM：\org\apache\maven\model\pom-4.0.0.xml ，它是所有Maven POM的父POM，所有Maven项目继承该配置，你可以在这个POM中发现如下配置：
```xml
 <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
```
中央仓库的id为central，远程url地址为http://repo.maven.apache.org/maven2，它关闭了snapshot版本构件下载的支持

### 在POM中配置远程仓库
比如你有一个局域网的远程仓库，使用该仓库能大大提高下载速度，继而提高构建速度，也有可能你依赖的一个jar在central中找不到，它只存在于某个特定的公共仓库，这样你也不得不添加那个远程仓库的配置。

```xml
<repositories>
<!--用来配置maven项目的远程仓库. 每个<repository>都有它唯一的ID，一个描述性的name，以及远程仓库的url -->
<repository>
    <id>maven-net-cn</id>
    <name>Maven China Mirror</name>
    <url>http://maven.net.cn/content/groups/public/</url>
    <!-- 可以从这个仓库下载releases版本的构件 -->
    <releases>
    <enabled>true</enabled>
    </releases>
    <!-- 不要从这个仓库下载snapshot版本的构件 -->
    <snapshots>
    <enabled>false</enabled>
    </snapshots>
</repository>
</repositories>

<pluginRepositories>
<!-- 用来配置maven插件的远程仓库 -->
<pluginRepository>
    <id>maven-net-cn</id>
    <name>Maven China Mirror</name>
    <url>http://maven.net.cn/content/groups/public/</url>
    <releases>
    <enabled>true</enabled>
    </releases>
    <snapshots>
    <enabled>false</enabled>
    </snapshots>    
</pluginRepository>
</pluginRepositories>
```

### 在settings.xml中配置远程仓库
定义一个id为dev的profile，将所有repositories以及pluginRepositories元素放到这个profile中，然后，使用<activeProfiles>元素自动激活该profile。这样，你就不用再为每个POM重复配置仓库。
使用profile为settings.xml添加仓库提供了一种用户全局范围的仓库配置。
```xml
<profiles>
    <profile>
        <id>dev</id>
        <!-- repositories and pluginRepositories here-->
    </profile>
</profiles>
<activeProfiles>
    <activeProfile>dev</activeProfile>
</activeProfiles>
```

### 在settings.xml中使用镜像
如果你的地理位置附近有一个速度更快的central镜像，或者你想覆盖central仓库配置，或者你想为所有POM使用唯一的一个远程仓库（这个远程仓库代理的所有必要的其它仓库），你可以使用settings.xml中的mirror配置。
以下的mirror配置用maven.NET.cn覆盖了Maven自带的central：
```xml
<mirrors>
    <mirror>
        <id>maven-net-cn</id>
        <name>Maven China Mirror</name>
        <url>http://maven.net.cn/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>
</mirrors>
```

### 分发构件至远程仓库
mvn deploy用来将项目生成的构件分发到远程Maven仓库。本地Maven仓库的构件只能供当前用户使用，在分发到远程Maven仓库之后，所有能访问该仓库的用户都能使用你的构件
配置POM的distributionManagement来指定Maven分发构件的位置
```xml
<distributionManagement>    
<!-- snapshot为开发过程中的版本，实时，但不稳定，release版本则比较稳定。Maven会根据你项目的版本来判断将构件分发到哪个仓库 -->
    <repository>    
      <id>nexus-releases</id>    
      <name>Nexus Release Repository</name>    
      <url>http://127.0.0.1:8080/nexus/content/repositories/releases/</url>    
    </repository>    
    <snapshotRepository>    
      <id>nexus-snapshots</id>    
      <name>Nexus Snapshot Repository</name>    
      <url>http://127.0.0.1:8080/nexus/content/repositories/snapshots/</url>    
    </snapshotRepository>    
  </distributionManagement>    
```

一般来说，分发构件到远程仓库需要认证。如下在settings.xml中配置认证信息：
```xml
<servers>    
<!-- server元素下id的值必须与POM中repository或snapshotRepository下id的值完全一致。将认证信息放到settings下而非POM中，是因为POM是可见的，而settings.xml是本地的。 -->
    <server>    
      <id>nexus-releases</id>    
      <username>admin</username>    
      <password>admin123</password>    
    </server>    
    <server>    
      <id>nexus-snapshots</id>    
      <username>admin</username>    
      <password>admin123</password>    
    </server>      
</servers>    
```
