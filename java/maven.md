Maven通过lifecycle、phase和goal来提供标准的构建流程。使用Maven构建项目就是执行lifecycle，执行到指定的phase为止。每个phase会执行自己默认的一个或多个goal。goal是最小任务单元。

常用命令：

mvn clean

mvn clean compile

mvn clean test

mvn clean package

mvn exec:java -Dexec.mainClass="mvn.client.Client"

maven生命周期：

- validate
- initialize
- generate-sources
- process-sources
- generate-resources
- process-resources
- compile
- process-classes
- generate-test-sources
- process-test-sources
- generate-test-resources
- process-test-resources
- test-compile
- process-test-classes
- test
- prepare-package
- package
- pre-integration-test
- integration-test
- post-integration-test
- verify
- install
- deploy

命令：maven phase

在使用命令时，其实是通过插件中的goal来实现的。maven内置插件：clean/compiler/surefire/jar

Maven支持模块化管理，可以把一个大项目拆成几个模块：

- 可以通过继承在parent的`pom.xml`统一定义重复配置；
- 可以通过`<modules>`编译多个模块。

![image-20210205233543408](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210205233543408.png)





创建一个maven项目：

1. mvn archetype(generate -DarchetypeCatalog=internal)