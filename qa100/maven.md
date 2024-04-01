# maven的一些命令

## maven scope的用法
scope规定了依赖包的依赖传递特性以及有效范围（compile、runtime、test）
scope有五个取值范围：
compile：编译、运行时、测试都有效，可以依赖传递
privided：编译和测试有效，不会进行依赖传递
runtime：运行、测试有效，会进行依赖传递，jdbc驱动
test：测试范围内有效，不会进行依赖传递，junit
system：编译和测试有效，可以进行依赖传递


## 下载源码包命令
mvn dependency:sources
mvn dependency:resolve -Dclassifier=javadoc
## 下载源码
mvn dependency:resolve -Dclassifier=sources

## 执行打包
mvn clean package -Dmaven.test.skip=true

## deploy https证书报错问题
deploy -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -f pom.xml

## clean install  证书报错问题
mvn clean package -DskipTests=true -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true

## 参数详解
mvn clean package -DskipTests=true -pl stat-web -am
-am    also-make,在构建当前项目之前先构建它所依赖的项目
-pl 构建项目的特定模块或子模块，使用-pl时只会构建指定模块，而不是整个项目


