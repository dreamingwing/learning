# 文件及其IO流、

## 1、new一个FileInputStream对象也会实际打开文件

## 2、MappedByteBuffer操作大文件，性能极高？

https://blog.csdn.net/qq_41969879/article/details/81629469

## 3、序列化和反序列化

那如果版本号一样，但实际的字段不匹配呢？Java会分情况自动进行处理，以尽量保持兼容性，大概分为三种情况：

❑ 字段删掉了：即流中有该字段，而类定义中没有，该字段会被忽略；

❑ 新增了字段：即类定义中有，而流中没有，该字段会被设为默认值；

❑ 字段类型变了：对于同名的字段，类型变了，会抛出InvalidClassException。

序列化的主要用途有两个：一个是对象持久化；另一个是跨网络的数据交换、远程过程调用。Java标准的序列化机制有很多优点，使用简单，可自动处理对象引用和循环引用，也可以方便地进行定制，处理版本问题等，但它也有一些重要的局限性。

1）Java序列化格式是一种私有格式，是一种Java特有的技术，不能被其他语言识别，不能实现跨语言的数据交换。

2）Java在序列化字节中保存了很多描述信息，使得序列化格式比较大。3）Java的默认序列化使用反射分析遍历对象结构，性能比较低。4）Java的序列化格式是二进制的，不方便查看和修改。

## 4、定制序列化

定制序列化

1．忽略字段

❑ @JsonIgnore：用于字段、getter或setter方法，任一地方的效果都一样。❑ @JsonIgnoreProperties：用于类声明，可指定忽略一个或多个字段。

2．引用同一个对象

@JsonIdentityInfo中指定了两个属性，property="id"表示在序列化输出中新增一个属性"id"以表示对象的唯一标示，generator表示对象唯一ID的产生方法，这里是使用整数顺序数产生器IntSequenceGenerator。

3．循环引用

如果序列化parent这个对象，Jackson会进入无限循环，最终抛出异常，解决这个问题，可以分别标记Parent类中的child和Child类中的parent字段，将其中一个标记为主引用，而另一个标记为反向引用，主引用使用@JsonManagedReference，反向引用使用@JsonBackReference，

4．反序列化时忽略未知字段

## 5．继承和多态

## 6、修改字段名称

可以使用@JsonProperty进行注解

## 7．格式化日期

## 8．配置构造方法