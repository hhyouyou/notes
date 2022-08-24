[TOC]







# 读书笔记 —— 《SpringBoot 源码分析》



掘金上购买的小册

[SpringBoot 源码解读与原理分析](https://juejin.cn/book/6844733814560784397/section)





## 启动引导



### @SpringBootApplication

```java

@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    
}
```



@SpringBootConfiguration: 没什么实际意义，标注下 @Configuration

@ComponentScan： 扫包， 然后排除指定类