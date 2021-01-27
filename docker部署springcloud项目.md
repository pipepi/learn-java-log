# docker 部署springcloud项目【windows】
## 遗留问题
1. nginx 部署vue前端项目本地无法使用80端口
  - 临时方案：
    - nginx容器80端口映射主机8080端口
2. delivery后端服务不加载nocos配置
  - 临时方案：
    - 将配置打包到jar包中，先不使用nacos配置
      - 问题
        - 不能用nacos来更新配置，
        - 修改配置，需要修改工程中的配置文件，重新maven打包镜像，重启服务
3. 登录请求9010节点信息错误
    - 修改dmp-security TokenChantroler.java
    ```java
        /*
        * 通过请求获取本地PC的IP地址
        */
        public String getRemoteAddr(HttpServletRequest request) {
            String ip = request.getHeader("X-Forwarded-For");
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("Proxy-Client-IP");
            }
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("WL-Proxy-Client-IP");
            }
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("HTTP_CLIENT_IP");
            }
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getHeader("HTTP_X_FORWARDED_FOR");
            }
            if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getRemoteAddr();
            }
            return ip;
        }

        @PostMapping("/login")
        @ApiOperation(value = "登录", notes = "登录, username:用户名，password:密码，platformType:平台类型")
        public CommonResult login(HttpServletRequest request) throws UnsupportedEncodingException {
            logger.info("请求 login 接口，进行登录操作");
            String header = request.getHeader("Authorization");
            if (header == null && !header.startsWith("Basic")) {
                return RestHelper.error(400, "请求头中缺少参数");
            }
    //        String code = request.getParameter("code");
            String username = request.getParameter("username");
    //        if(code==null){
    //            return RestHelper.error(500,"验证码缺失");
    //        }
    //        String old_code =redisTemplate.opsForValue().get(username+"_code");
    //        if(old_code==null){
    //            return RestHelper.error(500,"验证码不存在或者已经过期");
    //        }
    //        if(!code.equals(old_code)){
    //            return RestHelper.error(500,"验证码错误");
    //        }
            String url = "http://" + getRemoteAddr(request) + ":" + request.getServerPort() + "/oauth/token";
    ```
4. 网关403错误
  - 请求http://192.168.1.96:9009/delivery/leadNewsDirectional/getPageList 
    - Authorization
5. docking连接redis异常
  - 日志
    ```shell
    Caused by: org.redisson.client.RedisException: ERR Client sent AUTH, but no password is set. channel: [id: 0x192cee26, L:/172.18.0.8:49050 - R:redis/172.18.0.4:6379] command: (AUTH), params: (password masked)
    ```
  - 解决方法
    - 启动redis指定密码
    ```yaml
      redis:
        image: redis:5
        container_name: redis
        command: redis-server --appendonly yes --requirepass yxt123456
        volumes:
          - /mydata/redis/data:/data #数据文件挂载
        ports:
          - 6379:6379
      nacos-registry:
        image: nacos/nacos-server:1.3.0
        container_name: nacos-registry
        environment:
          - "MODE=standalone"
        ports:
          - 8848:8848
    ``` 
## 安装docker，安装本地镜像仓库registry:2
- 安装docker-desktop-windows 
  - 官网下载安装
- 安装registry:2
  - 参考 http://www.macrozheng.com/#/../reference/docker_maven
  ```shell
  docker run -d -p 5000:5000 --restart=always --name registry2 registry:2
  ```
  - 修改docker设置 dockerEngine
  ```json
  {
    "registry-mirrors": [
      "https://registry.docker-cn.com"
    ],
    "insecure-registries": [
      "192.168.1.96:5000"  // 192.168.1.96 替换为自己的本地ip
    ],
    "debug": false,
    "experimental": false,
    "features": {
      "buildkit": true
    }
  }
  ```
## 配置maven docker插件
- 父项目pom.xml配置maven doker插件
  ```xml
  <project>
    <properties>
      <docker.host>http://192.168.1.96:2375</docker.host>
      <docker.maven.plugin.version>1.2.2</docker.maven.plugin.version>
    </properties>
    <build>
      <pluginManagement>
        <plugins>
          <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>${docker.maven.plugin.version}</version>
            <executions>
                <execution>
                    <id>build-image</id>
                    <phase>package</phase>
                    <goals>
                        <goal>build</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <imageName>dmp/${project.artifactId}:${project.version}</imageName>
                <dockerHost>${docker.host}</dockerHost>
                <baseImage>java:8</baseImage>
                <entryPoint>["java", "-jar",
                    "-Dspring.profiles.active=prod","/${project.build.finalName}.jar"]
                </entryPoint>
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
          </plugin>
        </plugins>
      </pluginManagement>
    </build>
  </project>
  ```
  - 子项目pom.xml添加docker build插件
  ```xml
  <build>
    <plugins>
      <plugin>
        <groupId>com.spotify</groupId>
        <artifactId>docker-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
  ```
## 2375 端口调试
```shell
telnet 127.0.0.1 2375
 netstat -ant | select-string -pattern "2375"
```
## 安装环境依赖mysql ,redis, nacos
```shell
docker-compose -f docker-compose-env.yml up -d
```
```yaml
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root #设置root帐号密码
    ports:
      - 3306:3306
    volumes:
      - /mydata/mysql/data/db:/var/lib/mysql #数据文件挂载
      - /mydata/mysql/data/conf:/etc/mysql/conf.d #配置文件挂载
      - /mydata/mysql/log:/var/log/mysql #日志文件挂载
  redis:
    image: redis:5
    container_name: redis
    command: redis-server --appendonly yes
    volumes:
      - /mydata/redis/data:/data #数据文件挂载
    ports:
      - 6379:6379
  nacos-registry:
    image: nacos/nacos-server:1.3.0
    container_name: nacos-registry
    environment:
      - "MODE=standalone"
    ports:
      - 8848:8848
```
### mysql 数据库配置
```shell
docker cp /mydata/dmp.sql mysql:/
```
- 进入mysql容器并执行如下操作
```sql
#进入mysql容器
docker exec -it mysql /bin/bash
#连接到mysql服务
mysql -uroot -proot --default-character-set=utf8
#创建远程访问用户 [运维人员须关注安全问题而修改命令]
grant all privileges on *.* to 'dmp' @'%' identified by '123456yxt';
#创建digital_marketing_platform数据库
create database digital_marketing_platform character set utf8;
#使用digital_marketing_platform数据库
use digital_marketing_platform;
#导入digital_marketing_platform.sql脚本
source /sql/digital_marketing_platform.sql;
```
- 修改用户
```sql
SELECT User, Host FROM mysql.user;
DROP USER 'dmp'@'%';
DROP DATABASE dmp;
```
- 新建xxx-job-test数据库
```sql
#创建xxx-job-test数据库
create database xxx-job-test character set utf8;
#使用xxx-job-test数据库
use xxx-job-test;
#导入xxx-job-test.sql脚本
source /sql/xxx-job-test.sql;
```
## 安装应用 投放端，第三方接口，管理端，认证中心，网关
### 服务依赖关系
- dmp-platform-delivery
  - 依赖
    - mysql:3306
    - redis:6379
    - nacos-registry:8848
    - dmp-security:9010
    - xxx-job-admin:9007
### 部署网关服务
```yaml
version: '3'
services:
  dmp-platform-delivery:
    image: dmp/dmp-platform-delivery:0.0.1-SNAPSHOT
    container_name: dmp-platform-delivery
    ports:
      - 9012:9012
    volumes:
      - /mydata/app/dmp-platform-delivery/logs:/data/applogs
      - /etc/localtime:/etc/localtime
    environment:
      - 'TZ="Asia/Shanghai"'
#    depends_on:
#      - dmp-security #dmp-platform-delivery在dmp-security启动之后再启动
#      - xxl-job-admin
    external_links:
      - mysql:db #可以用db这个域名访问mysql服务
      - redis:redis #可以用redis这个域名访问redis服务
      - nacos-registry:nacos-registry #可以用nacos-registry这个域名访问nacos服务
      - dmp-security:dmp-security
      - xxl-job-admin:xxl-job-admin
  dmp-gateway:
    image: dmp/dmp-gateway:0.0.1-SNAPSHOT
    container_name: dmp-gateway
    ports:
      - 9009:9009
    volumes:
      - /mydata/app/dmp-gateway/logs:/data/applogs
      - /etc/localtime:/etc/localtime
    environment:
      - 'TZ="Asia/Shanghai"'
    depends_on:
      - dmp-platform-delivery #dmp-gateway在dmp-platform-delivery启动之后再启动
    external_links:
      - nacos-registry:nacos-registry #可以用nacos-registry这个域名访问nacos服务
```
### 部署vue前端
- docker-compose-nginx.yml
```yaml
version: '3'
services:
  nginx:
    image: nginx:1.10
    container_name: nginx
    volumes:
      - /mydata/nginx/conf/nginx.conf:/etc/nginx/nginx.conf #配置文件挂载
      - /mydata/nginx/html:/usr/share/nginx/html #静态资源根目录挂载
      - /mydata/nginx/log:/var/log/nginx #日志文件挂载
    ports:
      - 80:80
```
- nginx.conf
```shell
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    server {
	    listen       80;
	    server_name  localhost;

	    location / {
	        root   /usr/share/nginx/html;
	        index  index.html index.htm;
	    }

	    error_page   500 502 503 504  /50x.html;
	    location = /50x.html {
	        root   /usr/share/nginx/html;
	    }
	}
}

```
- 构建前端工程
- 将dist移动到html目录
### 部署dmp-platform-docking
- 项目依赖
  - mysql:db #可以用db这个域名访问mysql服务
  - redis:redis #可以用redis这个域名访问redis服务
  - nacos-registry:nacos-registry #可以用nacos-registry这个域名访问nacos服务
  - dmp-security:dmp-security
  - xxl-job-admin:xxl-job-admin 
  - minioadmin 
    - http://192.168.1.140:9000/
  - hbase 
    - zookeeper
      - 47.115.171.129:2181
- docker-compose-app.yaml添加dmp-platform-docking配置
```yaml
  dmp-platform-docking:
    image: dmp/dmp-platform-docking:0.0.1-SNAPSHOT
    container_name: dmp-platform-docking
    ports:
      - 9013:9013
    volumes:
      - /mydata/app/dmp-platform-docking/logs:/data/applogs
      - /etc/localtime:/etc/localtime
    environment:
      - 'TZ="Asia/Shanghai"'
    external_links:
      - mysql:db #可以用db这个域名访问mysql服务
      - redis:redis #可以用redis这个域名访问redis服务
      - nacos-registry:nacos-registry #可以用nacos-registry这个域名访问nacos服务
      - dmp-security:dmp-security
      - xxl-job-admin:xxl-job-admin
```
### 部署dmp-platform-management
- 项目依赖
  - http://192.168.1.140:9999 # file-server
  - 
## maven docker插件上传镜像到私有harbor仓库
1. 修改本地docker engine的配置，添加 insecure-registries，避免登录x509错误
```json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [
    "127.0.0.1:5000",
    "221.229.216.233:24443"
  ],
  "debug": false,
  "experimental": false,
  "features": {
    "buildkit": true
  }
}
```
2. 将harbor登录信息添加到maven setting.xml 中的server节点
```xml
<servers>
    <server>
       <id>yxtTestDockerRepo</id>
       <username>admin</username>
       <password>Yxt1108.</password>
    </server>
  </servers>
```
3. 修改根pom.xml 将tag_image，push_image目标绑定到packge阶段
```xml
<build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
<!--                    <version>${spring-boot-dependencies.version}</version>-->
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-resources-plugin</artifactId>
                    <configuration>
                        <delimiters>@</delimiters>
                        <useDefaultDelimiters>false</useDefaultDelimiters>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>com.spotify</groupId>
                    <artifactId>docker-maven-plugin</artifactId>
                    <version>${docker.maven.plugin.version}</version>
                    <executions>
                        <execution>
                            <id>build-image</id>
                            <phase>package</phase>
                            <goals>
                                <goal>build</goal>
                            </goals>
                        </execution>
                        <execution>
                            <id>tag-image</id>
                            <phase>package</phase>
                            <goals>
                                <goal>tag</goal>
                            </goals>
                            <configuration>
                                <image>dmp/${project.artifactId}:${project.version}</image>
                                <newName>221.229.216.233:24443/digitalmarketing-test/${project.artifactId}:${project.version}</newName>
                            </configuration>
                        </execution>
                        <execution>
                            <id>push-image</id>
                            <phase>package</phase>
                            <goals>
                                <goal>push</goal>
                            </goals>
                            <configuration>
                                <imageName>221.229.216.233:24443/digitalmarketing-test/${project.artifactId}:${project.version}</imageName>
                            </configuration>
                        </execution>
                    </executions>
                    <configuration>
                        <imageName>dmp/${project.artifactId}:${project.version}</imageName>
                        <dockerHost>${docker.host}</dockerHost>
                        <baseImage>java:8</baseImage>
                        <entryPoint>["java", "-jar","-Xms128m","-Xmx512m","/${project.build.finalName}.jar"]</entryPoint>
                        <resources>
                            <resource>
                                <targetPath>/</targetPath>
                                <directory>${project.build.directory}</directory>
                                <include>${project.build.finalName}.jar</include>
                            </resource>
                        </resources>
                        <serverId>yxtTestDockerRepo</serverId>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
```
4. 通过打包命令push镜像
```shell
mvn clean package -Ptest # build tag push
mvn clean package -Ptest -DskipDockerPush # build tag

```
