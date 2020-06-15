portainer注意下：

portainer  默认读取的是控制台日志，需要在logback-spring.xml里面打开控制台输出。

```xml
   <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="FILE_INFO"/>
        <appender-ref ref="FILE_ERROR"/>
    </root>
```

