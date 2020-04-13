Nginx 高性能的HTTP和反向代理Web服务器；占用内存少，并发能力强 



命令应该在路径 /usr/local/nginx/sbin 下执行



开启

```
./nginx
```

查看版本

```
./nginx -v
```

关闭 nginx

```
./nginx -s stop
```

重载

```
./nginx -s reload
```

![image-20200409202158822](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200409202158822.png)





![image-20200409203356882](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200409203356882.png)







![image-20200409203526019](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200409203526019.png)





#### 负载均衡的策略：4种

轮循；权重 weight；hash(以 IP 地址算哈希)，同个IP后续固定访问同一服务器；fair，响应时间短的优先







![image-20200410112019588](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200410112019588.png)

