开放端口



重启防火墙

```
 systemctl restart firewalld
```

开放80端口

```
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

防火墙开放80端口

```
firewall-cmd --zone=public --add-port=80/tcp --permanent 
```

查看开放端口

```
 firewall-cmd --list-all
```

```
nohup sh bin/mqbroker -n 59.110.64.180:9876 -c conf/broker.conf autoCreateTopicEnable=true &

nohup ./bin/mqnamesrv -n 59.110.64.180:9876 &
```



查看某个端口是否开启中：

netstat -anlp | grep 8161