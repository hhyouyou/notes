[TOC]

# 使用spring validation完成数据后端校验

## 前言

* 避免用户绕过前端校验，直接向后端发送违法数据

## 引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

我们只需要引入spring-boot-starter-web依赖即可，如果查看其子依赖，可以发现如下的依赖：

web模块使用了hibernate-validation，并且databind模块也提供了相应的数据绑定功能。

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

## 在传入req上加上相应注解

```java
public class userReq {

    @NotBlank
    private String name;

    @Min(18)
    private Integer age;

    @Pattern(regexp = "^1(3|4|5|7|8)\\d{9}$",message = "手机号码格式错误")
    @NotBlank(message = "手机号码不能为空")
    private String phone;

    @Email(message = "邮箱格式错误")
    private String email;

}

// 常用注解
org.hibernate.validator.constraints
javax.validation.constraints

@AssertTrue     被注释的元素必须为 true    
@AssertFalse    被注释的元素必须为 false 
@DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@Digits (integer, fraction)    被注释的元素必须是一个数字，其值必须在可接受的范围内
@Email  被注释的元素必须是电子邮箱地址
@Future     被注释的元素必须是一个将来的日期
@FutureOrPresent 被注释的元素必须是一个将来的日期或者现在
@Max(value)     被注释的元素必须是一个数字，其值必须小于等于指定的最大值
@Min(value)     被注释的元素必须是一个数字，其值必须大于等于指定的最小值
@Negative       被注释的元素必须是一个数字，其值必须为负值
@NotBlank(message =)   验证字符串非null，且长度必须大于0
@NotEmpty   被注释的字符串的必须非空  
@NotNull    被注释的元素必须不为 null    
@Null   被注释的元素必须为 null 
@Past   被注释的元素必须是一个过去的日期     
@Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式
@Size(max=, min=)   被注释的元素的大小必须在指定的范围内  
@Length(min=,max=)  被注释的字符串的大小必须在指定的范围内    
@Range(min=,max=,message=)  被注释的元素必须在合适的范围内
```



## 在@Controller中校验数据

```java
    @PostMapping("/test")
    @ResponseBody
    public ServerResponse test(@Validated  TestReq user, BindingResult result) {
        if(bindingResult.hasErrors()){
            for (FieldError fieldError : bindingResult.getFieldErrors()) {
                //...
            }
            return ServerResponse.fail();
        }
        return ServerResponse.ok("ok");
    }

```

![1566147182726](C:\Users\dongj\Desktop\result.png)





## 自定义注解校验

自定义spring validation非常简单，主要分为两步。

1 自定义校验注解 
我们尝试添加一个“字符串不能包含空格”的限制。

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {CannotHaveBlankValidator.class}) 
//自定义注解中指定了这个注解真正的验证者类。 CannotHaveBlankValidator.class
public @interface CannotHaveBlank {
    //默认错误消息
    String message() default "不能包含空格";
    //分组
    Class<?>[] groups() default {};
    //负载
    Class<? extends Payload>[] payload() default {};
    //指定多个时使用
    @Target({FIELD, METHOD, PARAMETER, ANNOTATION_TYPE})
    @Retention(RUNTIME)
    @Documented
    @interface List {
        CannotHaveBlank[] value();
    }
}

```

我们不需要关注太多东西，使用spring validation的原则便是便捷我们的开发，例如payload，List ，groups，都可以忽略。

2 编写真正的校验者类

```java
/**
所有的验证者都需要实现ConstraintValidator接口，它的接口也很形象，包含一个初始化事件方法，和一个判断是否合法的方法。
*/
public class CannotHaveBlankValidator implements ConstraintValidator<CannotHaveBlank, String> {

    @Override
    public void initialize(CannotHaveBlank constraintAnnotation) {
        System.out.println("进入校验器");
    }

	/**
	ConstraintValidatorContext 这个上下文包含了认证中所有的信息，我们可以利用这个上下文实现获取默认错  误提示信息，禁用错误提示信息，改写错误提示信息等操作。
	*/

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        //null时不进行校验
        if (value != null && value.contains(" ")) {
            //获取默认提示信息
            String defaultConstraintMessageTemplate = context.getDefaultConstraintMessageTemplate();
            System.out.println("default message :" + defaultConstraintMessageTemplate);
            //禁用默认提示信息
            context.disableDefaultConstraintViolation();
            //设置提示语
            context.buildConstraintViolationWithTemplate("can not contains blank").addConstraintViolation();
            return false;
        }
        return true;
    }
}
```

 