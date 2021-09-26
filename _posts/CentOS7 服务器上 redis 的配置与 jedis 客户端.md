## CentOS7 服务器上 redis 的配置与 jedis 客户端

[toc]

### 服务端

1. 使用清华源安装 redis

```shell
sudo yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum --enablerepo=remi install redis
```

2. 简单配置一下

```shell
vim /etc/redis.conf

# 把 bind 127.0.0.1 注释了
# 保护模式关掉
protected-mode no
# 端口号默认6379 有需求可以自己更改
port 6379
# 有需求可以设置密码
requirepass xxx
```

3. 开启 HTTP 端口号 $80$ 和 redis 端口号 $6379$ 
4. 启动、重启与关闭服务

```shell
systemctl start redis # 启动服务
systemctl stop redis # 关闭服务
systemctl restart redis # 重启服务
systemctl status redis # 查看redis状态
systemctl enable redis # 设置开机自启动
netstat -lnp|grep 6379 # 查看端口
```

5. 进入 redis

```shell
redis-cli -h 127.0.0.1 -p 6379
```

6. 插入一个键值对与查询键

```shell
set 'key' 'value'
get 'key'
```

7. 查询所有的键值对

```shell
keys *
```



### 客户端

使用 JAVA + [jedis](https://github.com/xetorthio/jedis) 作为客户端访问数据库，照着 Github 上的整就行。

1. Maven 里加上 jedis 的依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>
```

2. 然后就可以简单地访问量了

```java
import redis.clients.jedis.Jedis;

public class TestRedis
{
    public void show()
    {
        Jedis jedis=new Jedis("xxx.xxx.xxx.xxx"); // 改成自己服务器的 IP，不用加上端口号
        jedis.set("v","k");
        System.out.println(jedis.get("v"));
        jedis.del("v");
    }
}
```

