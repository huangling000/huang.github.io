# Remote Method Invocation(远程方法调用)

​        **RMI与Web Service的异同：RMI可以传递方法，Web Service只能传递值。并且Web Service是通过HTTP协议进行数据传输，因此可以实现跨平台的调用，RMI传递一个包含方法的对象，RMI的客户端和服务端都只能是java平台。**

​         下图为客户端调用服务端的方法的过程：

![img](https://img-blog.csdn.net/20170227004328339?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3VpeGlhbmxvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## Server端实现

1. **定义接口**

定义接口用于描述服务端提供的方法，定义的是抽象方法，只包含方法名，参数，返回值等信息。这个借口被作为Stub。

接口用于远程调用，需要继承Remote类，且可能抛出RemoteException。若参数为某类的对象，该类需要继承Serializable。

2. **实现接口**

3. **公开接口**

   使实现该接口的类继承UnicastRemoteObject类或者通过UnicastRemoteObject.exportObject()方法将某个对象设置为公开接口对象。

4. **绑定接口**

   将Stub对象绑定至Registry。创建实现接口的类的对象，复制给接口对象。（绑定对象）。然后获取Registry，绑定Registry。

   ## Client端

   1. **添加接口**
   2. **调用接口**

