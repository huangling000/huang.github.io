### 使用SpringMVC方式开发用户信息

mvc：

controller层

UserController类：

首先添加@Controller（“user”）标记，用于被spring扫描到，Controller的name叫user。

指定@RequestMapping("/user")，指需要通过/user的方式访问到

service层







dataObject只是对数据库的映射，service的java层的model才是真正对数据逻辑上的处理