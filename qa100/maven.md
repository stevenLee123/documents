# maven的一些命令

## 下载源码包命令
mvn dependency:sources
mvn dependency:resolve -Dclassifier=javadoc

## 执行打包
mvn clean package -Dmaven.test.skip=true

## deploy https证书报错问题
deploy -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -f pom.xml