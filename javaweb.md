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
