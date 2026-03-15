# 新机到手的配置（自用）

## ssh

全局服务器 ssh 配置 `/etc/ssh/sshd_config`
公钥放在**要连接的用户**文件夹下 `~/.ssh/`

### 体操 1（改 ssh 端口）


```
/etc/ssh/sshd_config

# Port 22
```
去注释改端口，避免暴露在公网后总被扫描（也可配置 iptables 处理）。


### 体操 2（开启 VSCode 相关配置）

开启 TCP 转发让 VSCode-Server 被连接时能正常用

```
/etc/ssh/sshd_config

AllowTcpForwarding yes
```


### 体操 3（客户端可选）

配置节点使用，减少 ssh 连接到海外机子连接带来的延迟。

```
Host 昵称
  HostName <机器IP>
  Port <ssh 开放的连接端口>
  User <连接机器的用户名>
  IdentityFile <私钥存放路径>
  ProxyCommand "<用 git bash 的 connect 路径>" -H 127.0.0.1:<端口> %h %p
```

此外记得配置代理组和规则
