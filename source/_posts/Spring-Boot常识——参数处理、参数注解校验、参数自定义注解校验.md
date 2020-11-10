---
title: Spring Boot常识——参数处理、参数注解校验、参数自定义注解校验
date: 2020-05-25 19:21:17
tags: [Spring Boot, 后端]
---

## spring boot 常见http get ,post请求参数处理

　在定义一个Rest接口时通常会利用GET、POST、PUT、DELETE来实现数据的增删改查；这几种方式有的需要传递参数，后台开发人员必须对接收到的参数进行参数验证来确保程序的健壮性

- GET
  一般用于查询数据，采用明文进行传输，一般用来获取一些无关用户信息的数据
- POST
  一般用于插入数据
- PUT
  一般用于数据更新
- DELETE
  一般用于数据删除
  一般都是进行逻辑删除（即：仅仅改变记录的状态，而并非真正的删除数据）

`@PathVaribale` 获取url中的数据
`@RequestParam` 获取请求参数的值
`@GetMapping` 组合注解，是 @RequestMapping(method = RequestMethod.GET) 的缩写
`@RequestBody` 利用一个对象去获取前端传过来的数据

<!-- more -->

## PathVaribale 获取url路径的数据

请求URL：
localhost:8080/hello/id 获取id值

实现代码如下：

```java
@RestController
public class HelloController {
    @RequestMapping(value="/hello/{id}/{name}",method= RequestMethod.GET)
    public String sayHello(@PathVariable("id") Integer id,@PathVariable("name") String name){
        return "id:"+id+" name:"+name;
    }
}
```

在浏览器中 输入地址：
localhost：8080/hello/100/hello

输出：
id:81name:hello



##RequestParam 获取请求参数的值

获取url参数值，默认方式，需要方法参数名称和url参数保持一致
**localhost:8080/hello?id=1000**

```java
@RestController
public class HelloController {
    @RequestMapping(value="/hello",method= RequestMethod.GET)
    public String sayHello(@RequestParam Integer id){
        return "id:"+id;
    }
}
```

**输出：**
id：100

url中有多个参数时，如：localhost:8080/hello?id=98&&name=helloworld
具体代码如下：

```java
@RestController
public class HelloController {
    @RequestMapping(value="/hello",method= RequestMethod.GET)
    public String sayHello(@RequestParam Integer id,@RequestParam String name){
        return "id:"+id+ " name:"+name;
    }
}
```

获取url参数值，执行参数名称方式localhost:8080/hello?userId=1000

```java
@RestController
public class HelloController {
    @RequestMapping(value="/hello",method= RequestMethod.GET)
    public String sayHello(@RequestParam("userId") Integer id){
        return "id:"+id;
    }
}
```

> ⚠️注意：不输入id的具体值，此时返回的结果为null。具体测试结果如下：
> id：null
> 不输入id参数，则会报如下错误：
> whitelable Error Page错误



## GET参数校验

用法：不输入id时，使用默认值
具体代码如下：
localhost：8080/hello

```java
@RestController
public class HelloController {
    @RequestMapping(value="/hello",method= RequestMethod.GET)
    //required=false 表示url中可以无id参数，此时就使用默认参数
    public String sayHello(@RequestParam(value="id",required = false,defaultValue = "1") Integer id){
        return "id:"+id;
    }
}
```

**输出**
id:1

## POST JSON参数校验

常用校验注解

![image.png](https://ae04.alicdn.com/kf/Hd289cedc071141828658a02bedec27ad1.png)

**注意：**
接收到的参数默认都是字符串类型的
有的注解只能用在String类型的属性上

`@JsonProperty`可以实现前端的属性名和后台实体类的属性名不一致问题

校验方式：
使用`@RequestBody` `@Valid` 对JSON参数进行获取和校验。
通过`BindingResult` 去获取校验结果。

技巧01：利用BindingResult对象的hasErrors方法判断是否有参数错误
技巧02：利用BindingResult对象的getFieldErrors方法获取所有有参数错误的属性
技巧03：利用错误属性对象的getDefaultMessage去获取错误提示信息

```java
@RequestMapping(value = "/demo5",produces = MediaType.TEXT_PLAIN_VALUE)
    @ResponseBody
    public String test5(@RequestBody @Valid  User user , BindingResult bindingResult){
        if(bindingResult.hasErrors()){
            List<ObjectError> objectErrors = bindingResult.getAllErrors();
            System.out.println(objectErrors.toString());
            for(ObjectError objectError: objectErrors){
                System.out.println("objectError = " + objectError.getObjectName());
                System.out.println("objectError = " + objectError.getDefaultMessage());
                System.out.println("objectError = " + objectError.getCode());
                System.out.println("objectError = " + objectError.getArguments());

            }
        }
        String str =  user.toString();

        return str;
    }
```

对应User实体类代码：

```java
public class User {
    @NotEmpty(message = "ID不能为空")
    @NotBlank(message = "ID不能为空哟")
    @Getter @Setter private String id;

    @Min(value = 18)
    @Max(value = 30)
    @Getter @Setter private Integer age;

    @NotEmpty(message = "昵称不能为空")
    @NotBlank(message = "昵称不能为空哟")
    @JsonProperty("nickname") //  当前端属性为nick后台接收对象的属性为nickName时可以用@JsonProperty来保持一致
    @Getter @Setter private String name;
}
```

## 自定义注解校验

1. 定义一个校验注解

```java
import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER, ElementType.FIELD})
@Constraint(validatedBy = MyFormValidatorClass.class)
public @interface MyFormValidator {
    String value();
    String message() default "name can be test";
    Class<?>[] groups() default {};
    Class<? extends Payload>[]  payload() default {};
}
```

2. 定义一个约束校验

```java
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.lang.annotation.Annotation;

public class MyFormValidatorClass implements ConstraintValidator<MyFormValidator, Object>, Annotation {
    private String values;

    @Override
    public void initialize(MyFormValidator myFormValidator) {
        this.values = myFormValidator.value();
    }

    @Override
    public boolean isValid(Object value, ConstraintValidatorContext context) {
        if("test".equals((String)value)){
            return true;
        }
        return false;
    }

    @Override
    public Class<? extends Annotation> annotationType() {
        return null;
    }
}
```

3. 实体类中使用

```java
public class User2 {
    @NotEmpty(message = "ID不能为空")
    @NotBlank(message = "ID不能为空哟")
    //自定义校验注解-校验id是否为test
    @MyFormValidator(value = "abc",message = "dd")
    private String id;

    @Min(value = 18)
    @Max(value = 30)
    private Integer age;

    @NotEmpty(message = "昵称不能为空")
    @NotBlank(message = "昵称不能为空哟")
    @JsonProperty("nickname") //  当前端属性为nick后台接收对象的属性为nickName时可以用@JsonProperty来保持一致
```

4. 测试代码

```java
@RequestMapping(value = "/demo6",produces = MediaType.TEXT_PLAIN_VALUE)
    @ResponseBody
    public String test6(@RequestBody @Valid User2 user , BindingResult bindingResult){
        if(bindingResult.hasErrors()){
            List<ObjectError> objectErrors = bindingResult.getAllErrors();
            System.out.println(objectErrors.toString());
            for(ObjectError objectError: objectErrors){
                System.out.println("objectError = " + objectError.getObjectName());
                System.out.println("objectError = " + objectError.getDefaultMessage());
                System.out.println("objectError = " + objectError.getCode());
                System.out.println("objectError = " + objectError.getArguments());

            }
        }
        String str =  user.toString();

        return str;
    }
```

