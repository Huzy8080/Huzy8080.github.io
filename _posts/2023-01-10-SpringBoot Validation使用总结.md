---
layout: post
title: SpringBoot Validation使用总结
# subtitle: subtitle
date: 2023-01-10
author: redme
header-img: img/post-bg-cook.jpg
catalog: true
tags:
  - SpringBoot
  - Validation使用总结
---

## 简单使用

项目 Pom.xml 引入依赖

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

```

Controller 的参数使用 `@Validated` ​或者 `@Valid`​,UserVo 的字段使用 Hibernate validator 提供的注解进行校验

```java
@RestController
@Slf4j
@RequestMapping("valid")
public class UserController {
    @RequestMapping("/sample")
    public ResponseEntity sample(@Validated @RequestBody UserVo userVo){
        log.info("accept request: {}",userVo);
        return ResponseEntity.ok().build();
    }
}

@Data
public class UserVo {
    @NotBlank(message = "username must not be blank!")
    private String username;
    private String password;
    private String nickname;
    private Integer age;
    private LocalDate registerDate;
    private LocalDateTime lastLoginTime;
}
```

校验不通过时，会抛出 `MethodArgumentNotValidException` ​异常，可通过 `ControllerAdvice` ​捕获处理

```java
@Slf4j
@ControllerAdvice
@Order(Ordered.HIGHEST_PRECEDENCE)
public class GlobalControllerAdvice {
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    ResponseEntity handle(HttpServletRequest req, MethodArgumentNotValidException exception){
        log.error("Error: ",exception);
        BindingResult bindingResult = exception.getBindingResult();
        if (bindingResult.hasErrors()) {
            List<ObjectError> allErrors = exception.getAllErrors();
            if (!allErrors.isEmpty()) {
                String message = (allErrors.get(0)).getDefaultMessage();
                return ResponseEntity.badRequest().body(message);
            }
        }
        return ResponseEntity.badRequest().body(exception.getMessage());
    }
}
```

## 常用的校验注解

|@AssertFalse|Boolean,boolean|验证注解的元素值是false|
| :---------------------------------------------| :------------------------------------------------------------------------------------------------| :------------------------------------------------------------------------------------------------------------------------------------|
|@AssertTrue|Boolean,boolean|验证注解的元素值是true|
|@NotNull|任意类型|验证注解的元素值不是null|
|@Null|任意类型|验证注解的元素值是null|
|@Min(value=值)|BigDecimal，BigInteger, byte,short, int, long，等任何Number或CharSequence（存储的是数字）子类型|验证注解的元素值大于等于@Min指定的value值|
|@Max（value=值）|和@Min要求一样|验证注解的元素值小于等于@Max指定的value值|
|@DecimalMin(value=值)|和@Min要求一样|验证注解的元素值大于等于@ DecimalMin指定的value值|
|@DecimalMax(value=值)|和@Min要求一样|验证注解的元素值小于等于@ DecimalMax指定的value值|
|@Digits(integer=整数位数, fraction=小数位数)|和@Min要求一样|验证注解的元素值的整数位数和小数位数上限|
|@Size(min=下限, max=上限)|字符串、Collection、Map、数组等|验证注解的元素值的在min和max（包含）指定区间之内，如字符长度、集合大小|
|@Past|java.util.Date,java.util.Calendar;Joda Time类库的日期类型|验证注解的元素值（日期类型）比当前时间早|
|@Future|与@Past要求一样|验证注解的元素值（日期类型）比当前时间晚|
|@NotBlank|CharSequence子类型|验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的首位空格|
|@Length(min=下限, max=上限)|CharSequence子类型|验证注解的元素值长度在min和max区间内|
|@NotEmpty|CharSequence子类型、Collection、Map、数组|验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0）|
|@Range(min=最小值, max=最大值)|BigDecimal,BigInteger,CharSequence, byte, short, int, long等原子类型和包装类型|验证注解的元素值在最小值和最大值之间|
|@Email(regexp=正则表达式,flag=标志的模式)|CharSequence子类型（如String）|验证注解的元素值是Email，也可以通过regexp和flag指定自定义的email格式|
|@Pattern(regexp=正则表达式,flag=标志的模式)|String，任何CharSequence的子类型|验证注解的元素值与指定的正则表达式匹配|
|@Valid|任何非原子类型|嵌套验证关联的对象，如校验对象中包含其他子对象字段并需要验证，则在子对象上加@Valid注解，可以实现嵌套验证<br />|

## 对于 RequestParam 和 PathParam 的校验

```java
@Validated
@RestController
@Slf4j
@RequestMapping("get")
public class GetController {

    //get 请求使用注解校验，需要在Controller类上添加注解@Validated,否则不能进行校验
    //如校验不通过，需要全局处理ConstraintViolationException异常
    @GetMapping("/req_param")
    public ResponseEntity get(@Length(max = 3, message = "name max length is 3") @RequestParam String name) {
        log.info("accept request: {}", name);
        return ResponseEntity.ok().build();
    }

    @GetMapping("/req_path_param/{name}")
    public ResponseEntity path(@Length(max = 3, message = "name max length is 3") @PathVariable("name") String name) {
        log.info("accept request: {}", name);
        return ResponseEntity.ok().build();
    }
}

@Slf4j
@ControllerAdvice
@Order(Ordered.HIGHEST_PRECEDENCE)
public class GlobalControllerAdvice {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(ConstraintViolationException.class)
    ResponseEntity handle(HttpServletRequest req, ConstraintViolationException exception){
        log.error("Error: ",exception);
        return ResponseEntity.badRequest().body(exception.getMessage());
    }
}
```

‍

## 分组校验

当多个接口使用同一个 VO 作为入参时，各接口之间要校验的字段可能并不一样，这时需要使用分组校验。Validated 接口定义如下，其中 value 就是分组依据

```java
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Validated {

	/**
	 * Specify one or more validation groups to apply to the validation step
	 * kicked off by this annotation.
	 * <p>JSR-303 defines validation groups as custom annotations which an application declares
	 * for the sole purpose of using them as type-safe group arguments, as implemented in
	 * {@link org.springframework.validation.beanvalidation.SpringValidatorAdapter}.
	 * <p>Other {@link org.springframework.validation.SmartValidator} implementations may
	 * support class arguments in other ways as well.
	 */
	Class<?>[] value() default {};

}
```

实现步骤

1. 创建分组接口

   ```java
   public class ValidationGroups {

       public interface Create {}

       public interface Update{}
   }
   ```

2. UserVo 校验注解增加 groups 参数

   ```java
   @Data
   public class UserVo {
       //当groups为默认值时才校验
       @NotBlank(message = "username must not be blank!")
       private String username;

       //仅create接口校验
       @NotBlank(message = "password must not be blank!",groups={ValidationGroups.Create.class}) 
       private String password;

       //仅update接口校验
       @NotBlank(message = "nickname must not be blank!",groups={ValidationGroups.Update.class}) 
       private String nickname;

       //create接口和update接口都需要校验age参数
       @Min(value = 0,message = "age must more than 0!", groups = {ValidationGroups.Create.class,ValidationGroups.Update.class})
       private Integer age;

       private String gender;

       private LocalDate registerDate;

       private LocalDateTime lastLoginTime;
   }
   ```

3. Controller 接口校验注解增加 groups 参数

   ```java
       /**
        * 只校验UserVo里面groups={ValidationGroups.Create.class}的字段的值
        * @param userVo
        * @return
        */
       @RequestMapping("/create")
       public ResponseEntity create(@Validated(ValidationGroups.Create.class) @RequestBody UserVo userVo){
           log.info("accept request: {}",userVo);
           return ResponseEntity.ok().build();
       }

       /**
        * 只校验UserVo里面groups={ValidationGroups.Update.class}的字段的值
        * @param userVo
        * @return
        */
       @RequestMapping("/update")
       public ResponseEntity update(@Validated(ValidationGroups.Update.class) @RequestBody UserVo userVo){
           log.info("accept request: {}",userVo);
           return ResponseEntity.ok().build();
       }
   ```

如果需要校验的分组情况比较复杂，为了简化代码，可以将共同校验的字段的 group 定义为公共接口，并让其他 group 继承它即可，代码如下

```java
public class ValidationGroups {

    //继承自Common，会同时校验groups为Create和Common的字段
    public interface Create extends Common {
    }
    //继承自Common，会同时校验groups为Update和Common的字段
    public interface Update extends Common {
    }

    public interface Common {
    }
}

@Data
public class UserVo {
    //当groups为默认值时才校验
    @NotBlank(message = "username must not be blank!")
    private String username;

    //仅create接口校验
    @NotBlank(message = "password must not be blank!",groups={ValidationGroups.Create.class})
    private String password;

    //仅update接口校验
    @NotBlank(message = "nickname must not be blank!",groups={ValidationGroups.Update.class})
    private String nickname;

    //create接口和update接口都需要校验age参数
    @Min(value = 0,message = "age must more than 0!", groups = {ValidationGroups.Create.class,ValidationGroups.Update.class})
    private Integer age;

    //使用组合接口进行分组校验
    @NotBlank(message = "gender must not be blank!",groups={ValidationGroups.Common.class})
    private String gender;

    private LocalDate registerDate;

    private LocalDateTime lastLoginTime;
}
```

Controller 接口不需要作修改，当调用 Create 接口时，也会校验 gender 字段

## 嵌套对象的校验

@Valid 注解支持嵌套对象校验

```java
@Data
public class UserStatusVo {

    @NotBlank(message = "username must not be blank!")
    private String username;

    @Valid //嵌套对象校验，需要加上此注解
    @NotNull(message = "status must not be null!")
    private Status status;

    @Valid //嵌套对象校验，需要加上此注解
    @NotEmpty(message = "statusList must not be null!")
    private List<Status> statusList;

}
```

## Controller 接口参数为 List 的校验

当 Post 接口是以 List 为参数时，正常设计的接口如下，经测试，不会进行校验

```java
@RestController
@Slf4j
@RequestMapping("list")
public class ListController {
    //经测试，这种方式是不会进行校验的
    @RequestMapping("/notwork")
    public ResponseEntity sample(@Valid @RequestBody List<UserVo> userVoList) {
        log.info("accept request: {}", userVoList);
        return ResponseEntity.ok().build();
    }

}
```

解决方案一

在 Controller 类上加 `@Validated`​

```java
@Validated
@RestController
@Slf4j
@RequestMapping("list")
public class ListController {
    //经测试，这种方式是不会进行校验的
    @RequestMapping("/notwork")
    public ResponseEntity sample(@Valid @RequestBody List<UserVo> userVoList) {
        log.info("accept request: {}", userVoList);
        return ResponseEntity.ok().build();
    }

}
```

解决方案二

实现一个 List，使用嵌套对象的校验方式

```java
/**
 * 可校验的List
 * @param <E>
 */
public class ValidList<E> implements List<E> {

    @Valid
    private List<E> list;

    public ValidList() {
        list = new ArrayList<>();
    }
    public ValidList( List<E> list) {
        this.list =list;
    }

    @Override
    public int size() {
        return this.list.size();
    }

    @Override
    public boolean isEmpty() {
        return this.list.isEmpty();
    }
    // 其他Override方法同上，调用this.list.***
}
```

## 自定义注解实现

对于某些字段有特殊的校验逻辑，现成的注解无法胜任，则可以自定义注解并实现 `ConstraintValidator` ​来完成需求

```java
@Constraint(validatedBy = CustomValidator.class)
@Target( { ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomCheck {
    //通用字段
    String message() default "Invalid field";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};

    //自定义检查
    Mode mode() default Mode.UPPER;
}


public class CustomValidator implements ConstraintValidator<CustomCheck, String> {
    private Mode mode;

    @Override
    public void initialize(CustomCheck customCheck) {
        mode=customCheck.mode();
    }

    @Override
    public boolean isValid(String customField, ConstraintValidatorContext cxt) {
//        throw new IllegalArgumentException("something error ");//will throw ValidationException, should handle ValidationException
        if(Mode.UPPER.equals(mode)){
            return "SZ".equals(customField) || "BJ".equals(customField);
        }
        if(Mode.LOWER.equals(mode)){
            return "sz".equals(customField) || "bj".equals(customField);
        }
        return false;
    }
}
```

此时，可以在 vo 中使用自定义注解进行校验

```java
@Data
public class UserVo {
    //当groups为默认值时才校验
    @NotBlank(message = "username must not be blank!")
    private String username;

    //仅create接口校验
    @NotBlank(message = "password must not be blank!",groups={ValidationGroups.Create.class})
    private String password;

    //仅update接口校验
    @NotBlank(message = "nickname must not be blank!",groups={ValidationGroups.Update.class})
    private String nickname;

    //create接口和update接口都需要校验age参数
    @Min(value = 0,message = "age must more than 0!", groups = {ValidationGroups.Create.class,ValidationGroups.Update.class})
    private Integer age;

    //使用组合接口进行分组校验
    @NotBlank(message = "gender must not be blank!",groups={ValidationGroups.Common.class})
    private String gender;

    //自定义注解
    @CustomCheck(mode = Mode.UPPER, message = "upperBranch should be SZ or BJ!")
    private String upperBranch;

    //自定义注解，仅create接口校验
    @CustomCheck(mode = Mode.LOWER, message = "lowerBranch should be sz or bj!",groups = {ValidationGroups.Create.class})
    private String lowerBranch;

    private LocalDate registerDate;

    private LocalDateTime lastLoginTime;
}


```

## 手动调用 API 进行校验

```java
@RequestMapping("/test")
public ResponseEntity manual( @RequestBody List<UserVo> userVoList) {
    log.info("accept request: {}", userVoList);
    userVoList.forEach(ValidUtil::valid);
    return ResponseEntity.ok().build();
}


/** 校验工具类 */
import jakarta.validation.*;
import lombok.extern.slf4j.Slf4j;

import java.util.Set;

@Slf4j
public class ValidUtil {

    private static Validator createValidator(){
        Configuration<?> configure = Validation.byDefaultProvider().configure();
        ValidatorFactory factory = configure.buildValidatorFactory();
        Validator validator = factory.getValidator();
        factory.close();
        return validator;
    }

    public static <T> void valid(T obj){
        Validator validator = createValidator();
        Set<ConstraintViolation<T>> violationSet = validator.validate(obj);
        if(!violationSet.isEmpty()){
            String message = violationSet.stream().findFirst().get().getMessage();
            throw new IllegalArgumentException(message);
        }
    }
}
```

## 校验 LocalDate 格式

方案一

```java
@DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
@JsonFormat(pattern = "MM/dd/yyyy")
private LocalDate startDate;


```

方案二

```java
public class DateDeSerializer extends StdDeserializer<Date> {

    public DateDeSerializer() {
        super(Date.class);
    }

    @Override
    public Date deserialize(JsonParser p, DeserializationContext ctxt)
        throws IOException, JsonProcessingException {
        String value = p.readValueAs(String.class);
        try {
            return new SimpleDateFormat("MM/dd/yyyy").parse(value);
        } catch (DateTimeParseException e) {
            //throw an error
        }
    }

}
```

方案三

```java
@CustomDateValidator
private LocalDate startDate;

@Documented
@Constraint(validatedBy = CustomDateValidator.class)
@Target( { ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomDateConstraint {
    String message() default "Invalid date format";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class CustomDateValidator implements
  ConstraintValidator<CustomDateConstraint, LocalDate> {

    private static final String DATE_PATTERN = "MM/dd/yyyy";

    @Override
    public void initialize(CustomDateConstraint customDate) {
    }

    @Override
    public boolean isValid(LocalDate customDateField,
      ConstraintValidatorContext cxt) {
          SimpleDateFormat sdf = new SimpleDateFormat(DATE_PATTERN);
           try
        {
            sdf.setLenient(false);
            Date d = sdf.parse(customDateField);
            return true;
        }
        catch (ParseException e)
        {
            return false;
        }
    }

}

```