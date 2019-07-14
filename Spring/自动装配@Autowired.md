
1. Spring怎么知道注入哪个实现?  
As long as there is only a single implementation of the interface and that implementation is annotated with @Component with Spring’s component scan enabled, Spring framework can find out the (interface, implementation) pair. If component scan is not enabled, then you have to define the bean explicitly in your application-config.xml (or equivalent spring configuration file).     
如果Spring配置了component scan，并且要注入的接口只有一个实现的话，那么spring框架可以自动将interface于实现组装起来。如果没有配置component scan，那么你必须在application-config.xml（或等同的配置文件）定义这个bean。

2. 需要@Qualifier和@Resource注解吗?  
Once you have more than one implementation, then you need to qualify each of them and during auto-wiring, you would need to use the @Qualifier annotation to inject the right implementation, along with @Autowired annotation. If you are using @Resource (J2EE semantics), then you should specify the bean name using the name attribute of this annotation.   
一旦一个接口有多个实现，那么就需要每个特殊化识别并且在自动装载过程中使用@Qualifier和@Autowired一起使用来标明。如果是使用@Resource注解，那么你应该使用resource中属性名称来标注@Autowired.

3. 为什么@Autowired使用在interface上而不是实现类上？  
Firstly, it is always a good practice to code to interfaces in general. Secondly, in case of spring, you can inject any implementation at runtime. A typical use case is to inject mock implementation during testing stage.   
首先，一般而言面向接口编程是一种好的编程实践。其次，在spring中，你可以在运行过程中注入各种实现。一个很经典的情况就是在测试阶段，注入模拟的实现类。 
```java
interface IA { 
    public void someFunction(); 
}

 
class B implements IA { 
    public void someFunction() { 
        //busy code block 
    } 
    public void someBfunc() { 
        //doing b things 
    } 
}

 
class C implements IA { 
    public void someFunction() { 
        //busy code block 
    } 
    public void someCfunc() { 
        //doing C things 
    } 
}

class MyRunner { 
    @Autowire 
    @Qualifier(“b”)
    IA worker;
    // ....
    worker.someFunction();
} 

```
Your bean configuration should look like this:
 
Alternatively, if you enabled component scan on the package where these are present, then you should qualify each class with @Component as follows:
 
```java
interface IA { 
    public void someFunction(); 
}
 
 
@Component(value="b") 
class B implements IA { 
    public void someFunction() { 
        //busy code block 
    } 
    public void someBfunc() { 
        //doing b things 
    } 
}
 
 
@Component(value="c") 
class C implements IA { 
    public void someFunction() 
    { 
        //busy code block 
    } 
public void someCfunc() 
{ 
//doing C things 
} 
}
 
 
@Component 
class MyRunner { 
    @Autowire 
    @Qualifier(“b”)
    IA worker;
    // ....
    worker.someFunction();
}
``` 
