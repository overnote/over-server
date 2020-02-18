## 一 DockerMaven插件

微服务项目的部署是一大难题，比如Java项目，打包后的jar包手动部署在各个容器中，是非常麻烦的。使用DockerMaven插件可以实现自动部署。    

DockerMaven使用步骤：
```
# 修改宿主机docker配置，让其支持远程访问
vim /lib/systemd/system/docker.service
# 在 ExecStart这段配置后添加配置 -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

# 刷新配置，重启服务
systemctl daemon-reload
systemctl restart docker
docker start registry
```

在各模块中的pom文件中加入以下配置：
```xml
<plugin>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-maven-plugin</artifactId>
   <configuration>
      <includeSystemScope>true</includeSystemScope>
   </configuration>
</plugin>

<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <configuration>
        <imageName>${project.name}:${project.version}</imageName>
        <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
        <skipDockerBuild>false</skipDockerBuild>
        <resources>
            <resource>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```
此时在java项目的某个模块中就可以构建镜像，并执行上传了：
```
mvn docker:build -DpushImage
```