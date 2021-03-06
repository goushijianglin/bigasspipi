* 普通的javaweb 很多servlet，一个需求一个servlet类=》
* 一类数据用一个大的servlet 用switch，将一个关键字放到request里，在java中取出，用switch区分，分别调用这个大的servlet的不同方法=》

* 一个servlet，用反射技术，根据传过来的关键字，分别调用这个大的servlet的不同方法 可以方便追加新的业务需求，method.invoke（this,req,resp).=》

* 很多servlet的外面加一层dispatcherservlet（中央控制器，核心控制器）。后面的不同的servlet都可以叫做XXXcontroller。
dispatcherservlet(中央控制器）：1。根据请求的url，获取servletpath，之后截取字符串，解析一个名字（也就是bean的名字）
                             2.加载解析配置文件(xml)，从配置文件中获得一堆bean，之后实例化（反射），放到一个map里<beanname,bean实例>.
                             3.根据第一步的servletpath得到的bean名字找到bean实例， 再通过request带来的关键字，确定调用的method.invoke（this,req,resp)方法。=》
                             
* 还可以继续优化=〉
 将数据的读取（getparameter）放到dispatcherservlet，让xxxcontroller的页面跳转也放到dispatcherservlet中（让controller不再是servlet，只是一个普通类）
 
                             1.xxxcontroller直接return想要跳转或者重定向的一个字符串（不是在controller内跳转，是在中央控制器中统一跳转。）
                                 加载解析配置文件，从配置文件中获得一堆bean，之后实例化（反射），放到一个map里<beanname,bean实例>.
                                 根据第一步的servletpath得到的bean名字找到bean实例， 再通过request带来的关键字，确定调用的method方法。反射查找
                             2。在dispatcherservlet中
                             
                                   1。加载解析配置文件(xml)，从配置文件中获得一堆bean，之后实例化（反射），放到一个map里<beanname,bean实例>.
                                   2. 根据第一步的servletpath得到的bean名字找到bean实例， 再通过request带来的关键字，确定调用的method方法。
                                   3.（jdk 8开始 反射能获取形参变量）反射查找method的形参，放到一个数组里面 再从request.getparameter拿到数据 放到对应数组。
                                   4.执行invoke.
                                   
                             3。在dispatcherservlet中 的视图处理。1中return来的字符串。截取，执行重定向 
    常见错误 argument type mismatch 参数类型不匹配
    
    
**model1** **model2** =》 jsp 里面包含：1.html，css,js 2.java代码 和数据库连接的 3.java代码 将java数据展示在页面上的。

# **mvc**
Model模型（1.pojo,vo 2.DAO 3.Bo） 
    区分业务对象BO和数据访问对象DAO  
        1）DAO中的方法都是单精度方法，一个方法只考虑一个操作，添加查询。。。  
        2）BO中的方法属于业务方法，实际的业务是比较复杂的，都是DAO里面的方法组合而成 粒度比较粗  
        
View视图（数据展示，交互）   
Controller控制器（处理通过视图发的请求，具体业务还是要借助模型）  

# **IOC**
  ### 1.耦合，依赖
   依赖指的是xxx离不开xxx
   在软件系统中，层与层依赖，也称为耦合。  系统构架或设计的原则是：高内聚 低耦合.  
    层内部应该是高度聚合的，层与层之间应该是低耦合,理想情况下是0耦合。 
  ### 2.Ioc控制反转/DI依赖注入  
  ##### 控制反转：  
  1）之前的servlet中 要创建service对象，fruitService= new FruitServiceImpl();  
    如果在servlet中的方法内部创建，那么这个new的对象的作用域（生命周期）就应该是这个方法级别。  
    如果是在类的内部创建，则new的对象时servlet的成员变量，那么他的生命周期就是这个servlet的实例级别。
  2）之后在applicationContext.xml中定义了这个fruitService。然后通过解析xml，反射创建fruitService实例，存放在beanMap（容器）中，这个beanMap在一个BeanFactory中。
     因此 我们改变了service，dao实例的生命周期，控制权从程序员转移到BeanFactory中。即为控制反转。  
  ##### 依赖注入：
  1）之前我们在控制层的代码 FruitService fruitService= new FruitServiceImpl();  则控制层和service层存在耦合  
  2）之后 我们将控制层的代码改为 FruitService fruitService=null；在配置文件（applicationContext.xml）中配置：  
           ```xml 
           <bean id="fruit class="FruitController">  
              <property name="fruitService" ref="fruitService"/>  
           </bean>
           ```   
  通过解析xml,将beanMap里生成的ref（fruitService）对应的实例，注入（反射赋值）到FruitController内的名为name（fruitService）的field内。实现注入。



# **filter**  

##### 使用  
implements filter接口   
重写dofilter init doFilter(里面的加上放行的语句) destroy    
```java 
//放行
filterChain.doFilter(servletRequest,servletResponse)  
```
配置filter可以用注解@WebFilter，也可以是用xml<filter><filter-mapping>  

##### 过滤器链  
  执行顺序 ABC Demo C2 B2 A2
  多个过滤器按照注解配置的时候，是按照全类名的排序  
  如果是按照配置文件配置，是按照顺序谁先配置谁先过滤
  
# **事务管理**
  
#### 事务管理以业务层为单位，不能以dao层的单精度方法为单位
  service是一个整体 其中dao操作要一起成功，如果有失败的则一起回滚。
  
  
  简单版 在service中开启事物管理
  ```java
  //开启事务
  try{
  service;//一连串业务操作dao
  commit();//提交
  }catch(Exception e){
  rollback()//回滚
  }
  ```
  
  实用版 用过滤器，实现事务管理（OpenSessionInViewFilter)
  
  ```java
  try{
  autocommit(false);//关闭自动提交
  放行();//放行到servlet->service->一堆dao
  commit();
  }
 catch(exception e){
  rollback();
  }
  ```
**但是上述伪代码有问题，一堆dao要用一个connection，如果不用一个的话 随着connection关闭，就自动commit**  
  解决办法-> 我们要用Threadlocal(下面详细描述）中有两个方法 set(conn);get(conn);  
  ->使用它可以让一堆dao使用同一个connectionId。   
#### 用到的组件(pro20里面）  
  -OpenSessionInViewFilter  
  -TransactionManager  
  -ConnUtil  
  -BaseDAO  

#### ThreadLocal  
  -get();  
  -set(obj);
 ThreadLocal称之为本地线程,set将ThreadLocal对象存在线程的map（线程都维护着一个独自的map容器）上，get方法早当前线程获取数据。  
  
# *监听器Listener*  
  
1. ServletContextListener - (观察者模式）监听ServletContext对象的创建和销毁    
2. HttpSessionListener - 监听HttpSession对象的创建和销毁   
3. ServletRequestListener - 监听ServletRequest对象的创建和销毁  
4. ServletContextAttributeListener
5. HttpSessionAttributeListener - 监听HttpSession对象的创建和销毁   
6. RequestAttributeListener - 监听ServletRequest对象的创建和销毁

 
  


    
  
  
  
