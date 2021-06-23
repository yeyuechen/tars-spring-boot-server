###  基础条件：
   ````
   1、tars环境安装：详情请参考tars环境部署文档.md
   2、jdk环境安装与配置(可以自行百度搜索安装步骤JDK8及以上，windows环境参考：https://jingyan.baidu.com/article/48b37f8d231ca61a65648869.html)
   3、maven环境安装与配置(可以自行百度搜索安装最新即可。windows环境参考：https://jingyan.baidu.com/article/3052f5a1e8f86397f21f8671.html)
   ````
### 基于tars框架开发springboot项目

### 一、 服务端代码编写

注意初学者可以直接下载我的代码，直接跳到第二大步骤，进行部署；
如果想要了解整个springboot的代码编写过程可以走下面流程

##springboot生成方法基于springboot官网自动生成一个最基础的项目，然后进行调整
 http地址：https://start.spring.io/
![Image text](https://github.com/yeyuechen/image/blob/main/20210623102852.png)



##2、在pom.xml文件中添加一下配置:
````
  <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>com.tencent.tars</groupId>
            <artifactId>tars-spring-boot-starter</artifactId>
            <version>1.7.1</version>
        </dependency>
    </dependencies>
    
    <!--插件依赖-->
     <build>
     <finalName>HelloServer</finalName>
            <plugins>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.2</version>
                    <configuration>
                        <source>1.8</source>
                        <target>1.8</target>
                        <encoding>UTF-8</encoding>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>com.tencent.tars</groupId>
                    <artifactId>tars-maven-plugin</artifactId>
                    <version>1.7.2</version>
                    <configuration>
                        <tars2JavaConfig>
                            <tarsFiles>
                                <tarsFile>${basedir}/src/main/resources/hello.tars</tarsFile>
                            </tarsFiles>
                            <tarsFileCharset>UTF-8</tarsFileCharset>
                            <!--false为客户端-->
                            <servant>true</servant>
                            <srcPath>${basedir}/src/main/java</srcPath>
                            <charset>UTF-8</charset>
                            <tup>true</tup>
                            <packagePrefixName>com.qq.tars.quickstart.server.</packagePrefixName>
                        </tars2JavaConfig>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-jar-plugin</artifactId>
                    <version>2.6</version>
                    <configuration>
                        <archive>
                            <manifestEntries>
                                <Class-Path>conf/</Class-Path>
                            </manifestEntries>
                        </archive>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <!--  springboot 启动类 -->
                    <configuration>
                        <mainClass>com.qq.tars.quickstart.server.QuickStartApplication</mainClass>
                    </configuration>
                    <executions>
                        <execution>
                            <goals>
                                <goal>repackage</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
````

##3、建立tars文件，生成接口文件
   去resource目录下新加hello.tars文件，利用tars工具分两次生成接口，一次生成服务端接口（一个服务确定要有服务端接口，除了 http 服务），一次生成客户端接口（客户端是为了方便别的服务来调用当前服务）java
   ````
   module TestApp
   {
   	interface Hello
   	{
   	    string hello(int no, string name);
   	};
   };
   ````
   使用tars插件生成server接口类：mvn tars: tars2java
   生成后可以去com.qq.tars.quickstart.server目录下查看生成的代码，内容如下代表生成成功：
   ````
   @Servant
   public interface HelloServant {
   
       public String hello(int no, String name);
   }
   ````
##4、添加tars实现类
````
@TarsServant("HelloObj")
public class HelloServantImpl implements HelloServant {

    @Override
    public String hello(int no, String name) {
        return String.format("hello no=%s, name=%s, time=%s", no, name, System.currentTimeMillis());
    }
}
````

##5、最后添加tars服务的启动
````
@SpringBootApplication
@EnableTarsServer
public class QuickStartApplication {
    public static void main(String[] args) {
        SpringApplication.run(QuickStartApplication.class, args);
    }
}
````

项目结构代码如下：
   ````
   ├── pom.xml
   └── src
      └── main
          ├── java
          │   └── com.qq.tars.quickstart.server
          │       └── testapp
          │          ├── HelloServant.java
          │          ├── QuickStartApplication.java
          │          └── impl
          │                └── HelloServantImpl.java
          └── resources
              └── hello.tars
   ````


### 二、部署
登录tars框架web平台

1、点击运维管理
2、如果想要自己加个应用名称可以先点击应用管理自己填写个应用名称比如我的叫TestApp，否则直接进入3
3、点击服务部署，进行配置，如下图所示，点击确定

![Image text](https://github.com/yeyuechen/image/blob/main/20210623113647.png)

4、发布服务
将你的项目通过maven install进行打包，会生成HelloServer.jar的springboot服务
点击服务管理-->发布服务-->发布选中节点-->上传服务包
选择刚才上传的服务包，点击发布即可


![Image text](https://github.com/yeyuechen/image/blob/main/20210623114434.png)
