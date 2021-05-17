# yml文件识别失效

## 起因

新配置了电脑环境,发现我的`yml`配置文件报了一堆`warn`, 并且输入的是时候也没有提示了.

在代码中看到屎黄色的warn真是太让人难受了,而且配置没有提示,很容易输入错误.

## 解决

于是,我就去网上查了下发现`properties`和`yml`文件的自动识别是依赖于,spring项目包中的`spring-configuration-metadata.json` 这个文件来识别的.

![spring-configuration-metadata.json](http://image-djx.test.upcdn.net/md/notes/spring-configuration-metadatajson.jpg)



然后检查了idea的设置 settings->Editor->file types. 发现`json`文件都没有识别`*.json`

? 黑人问号脸, 真奇了怪了. 然后加上这个 `*.json`. `yml`文件没有warn报错了, 也可以做自动补全了.

![配置json文件识别](http://image-djx.test.upcdn.net/md/notes/settingFileTypes.png)



## 其他

既然都看到`spring-configuration-metadata.json`这个配置信息元数据文件了,那肯定要看下这个文件是何方神圣了?

![配置信息元数据](http://image-djx.test.upcdn.net/md/notes/springdoc-configuration%20metadata.png)

> 是spring在jar包中的添加的元数据文件，这些文件提供了所有支持的配置属性的详细信息。这些文件被设计用于让`IDE`开发人员在用户使用`application.properties`或`application.yml`文件时提供上下文帮助和“代码的自动补全”。
>
> 大多数元数据文件是在编译时通过处理带有@ConfigurationProperties注释的所有项自动生成的。但是，也可以为角落案例或更高级的用例手动编写部分元数据



下面是我自己写的一个notify包

```java
@Setter
@Getter
@Component
@ConfigurationProperties(prefix = "work-we-chat")
public class NotifyProperties {

    // TODO: 2021/4/15 渠道配置
    /**
     * 1. 企业微信机器人
     * 2. 企业微信应用(收,发消息)
     */

    private NotifyUser user;
    private String robotKey;
    private boolean enabledSend = false;

    @Data
    public static class NotifyUser {

        /**
         * 企业微信用户id, @all发送全员
         */
        private List<String> userId;

        /**
         * 企业微信用户手机号
         */
        private List<String> mobile;

    }

}
```



编译完, jar包中生成的 `spring-configuration-metadata.json`文件

```json
{
  "groups": [
    {
      "name": "work-we-chat",
      "type": "com.yutown.notify.config.NotifyProperties",
      "sourceType": "com.yutown.notify.config.NotifyProperties"
    },
    {
      "name": "work-we-chat.user",
      "type": "com.yutown.notify.config.NotifyProperties$NotifyUser",
      "sourceType": "com.yutown.notify.config.NotifyProperties",
      "sourceMethod": "getUser()"
    }
  ],
  "properties": [
    {
      "name": "work-we-chat.enabled-send",
      "type": "java.lang.Boolean",
      "sourceType": "com.yutown.notify.config.NotifyProperties",
      "defaultValue": false
    },
    {
      "name": "work-we-chat.robot-key",
      "type": "java.lang.String",
      "sourceType": "com.yutown.notify.config.NotifyProperties"
    }
  ],
  "hints": []
}
```



在其他项目中使用, 在`pom`文件中引入配置后.  在`yml`文件中编写相关配置,就可以出现对应的配置信息,包括注释信息.

![yml文件代码提示](http://image-djx.test.upcdn.net/md/notes/yml%E4%BB%A3%E7%A0%81%E6%8F%90%E7%A4%BA.png)





# 参考文章

1. [spring-boot 文档-Configuration Metadata](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-configuration-metadata.html)
2. [SpringBoot 如何让yml,properties配置文件有提示](https://my.oschina.net/u/3555293/blog/3005527)