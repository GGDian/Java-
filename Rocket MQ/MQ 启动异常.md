MQ 启动异常

原因：内存默认设置太大了，导致一开启就分配担保不了报出异常

解决：将runbroker 和 runserver 中的配置修改下

```
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn125m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

