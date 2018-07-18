modify pom.yml

增加:
```
<plugin>
        <groupId>com.spotify</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>1.1.0</version>
        <configuration>
            <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
            <baseImage>java</baseImage>
            <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
            <resources>
                <resource>
                    <targetPath>/</targetPath>
                    <directory>${project.build.directory}</directory>
                    <include>${project.build.finalName}.jar</include>
                </resource>
            </resources>
        </configuration>
</plugin>

```

配置数据库连接
vi src/main/resources/application.properties 
```
spring.datasource.url=jdbc:postgresql://pgset:5432/test # 注意这里的pgset:5432对应到k8s service中的server name 和 port
spring.datasource.username=user1
spring.datasource.password=123

```

运行：
```
mvn clean package docker:build
//手动build项目 docker build --build-arg JAR_FILE=./target/web-demo-0.0.1-SNAPSHOT.jar . -t springboot-demo
```

java 运行项目：
```
java -jar target/demo-test-0.0.1-SNAPSHOT.jar
```

运行项目：
docker run -p 8080:8080 -t xxx/xxx

访问地址：
http://127.0.0.1:8080/demo-test/person/list-all

## 配置 pg
安装pg后，配置项目



文档地址：
https://github.com/spotify/docker-maven-plugin

## K8S 部署:
创建deployment.yml
```
kubectl expose deployment springboot-demo-deployment --type=NodePort
```
