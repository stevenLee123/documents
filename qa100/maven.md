# maven的一些命令

## 下载源码包命令
mvn dependency:sources
mvn dependency:resolve -Dclassifier=javadoc

## 执行打包
mvn clean package -Dmaven.test.skip=true