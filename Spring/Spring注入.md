## 注入方法

### 配置注入

1. set方法注入：提供属性set方法，在属性文件中配置让框架能够找到applicationContext.xml的beans标签。声明bean的id class（路径）  property（要初始化的属性）
2. 构造函数注入：声明bean的id class（路径）  constructor-argvalue（构造函数）

### 注解注入

1. 自动装配：在配置文件中声明bean的autowire属性，表示自动装配，可以是byname，可以是bytype。
2. 注解注入：

@Repository可以理解为配置文件中的bean，@Controller，@Resource等可以理解为autowire属性。

### 比较

相同： 两种实质都是使用了构造函数

不同：set注入，是调用了构造函数后用set方法进行初始化值，构造注入是使用带有特定参数的构造函数初始化，注解注入是使用默认构造函数进行初始化，如果想对对象的值进行初始化，需要在默认构造函数内初始化。

配置注入：可以把值和代码分开，方便管理，在配置文件中设置

注解注入：简单方便

如果需要值得注入，一般用配置注入，只是用方法得类，可以使用注解注入。