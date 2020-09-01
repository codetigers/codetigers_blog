## Linux防火墙Firewall和Iptables的使用

> Linux中有两种防火墙软件，ConterOS7.0以上使用的是firewall，ConterOS7.0以下使用的是iptables，本文将分别介绍两种防火墙软件的使用。

## Firewall

- 开启防火墙：

  ```shell
  systemctl start firewalldCopy to clipboardErrorCopied
  ```

- 关闭防火墙：

  ```shell
  systemctl stop firewalldCopy to clipboardErrorCopied
  ```

- 查看防火墙状态：

  ```shell
  systemctl status firewalldCopy to clipboardErrorCopied
  ```

- 设置开机启动：

  ```shell
  systemctl enable firewalldCopy to clipboardErrorCopied
  ```

- 禁用开机启动：

  ```shell
  systemctl disable firewalldCopy to clipboardErrorCopied
  ```

- 重启防火墙：

  ```shell
  firewall-cmd --reloadCopy to clipboardErrorCopied
  ```

- 开放端口（修改后需要重启防火墙方可生效）：

  ```shell
  firewall-cmd --zone=public --add-port=8080/tcp --permanentCopy to clipboardErrorCopied
  ```

  

- 查看开放的端口：

  ```shell
  firewall-cmd --list-portsCopy to clipboardErrorCopied
  ```

  

- 关闭端口：

  ```shell
  firewall-cmd --zone=public --remove-port=8080/tcp --permanentCopy to clipboardErrorCopied
  ```

  

## Iptables

### 安装

> 由于CenterOS7.0以上版本并没有预装Iptables,我们需要自行装。

- 安装前先关闭firewall防火墙 ![img](http://www.macrozheng.com/images/refer_screen_34.png)

- 安装iptables:

  ```shell
  yum install iptablesCopy to clipboardErrorCopied
  ```

- 安装iptables-services:

  ```shell
  yum install iptables-servicesCopy to clipboardErrorCopied
  ```

### 使用

- ```shell
  systemctl start iptables.serviceCopy to clipboardErrorCopied
  ```

  

- 关闭防火墙：

  ```shell
  systemctl stop iptables.serviceCopy to clipboardErrorCopied
  ```

- 查看防火墙状态：

  ```shell
  systemctl status iptables.serviceCopy to clipboardErrorCopied
  ```

- 设置开机启动：

  ```shell
  systemctl enable iptables.serviceCopy to clipboardErrorCopied
  ```

- 禁用开机启动：

  ```shell
  systemctl disable iptables.serviceCopy to clipboardErrorCopied
  ```

- 查看filter表的几条链规则(INPUT链可以看出开放了哪些端口)：

  ```shell
  iptables -L -nCopy to clipboardErrorCopied
  ```

  

- 查看NAT表的链规则：

  ```shell
  iptables -t nat -L -nCopy to clipboardErrorCopied
  ```

  

- 清除防火墙所有规则：

  ```shell
  iptables -FCopy to clipboardErrorCopied
  ```

  ```shell
  iptables -XCopy to clipboardErrorCopied
  ```

  ```shell
  iptables -ZCopy to clipboardErrorCopied
  ```

- 给INPUT链添加规则（开放8080端口）：

  ```shell
  iptables -I INPUT -p tcp --dport 8080 -j ACCEPTCopy to clipboardErrorCopied
  ```

  

- 查找规则所在行号：

  ```shell
  iptables -L INPUT --line-numbers -nCopy to clipboardErrorCopied
  ```

  

- 根据行号删除过滤规则（关闭8080端口）：

  ```shell
  iptables -D INPUT 1Copy to clipboardErrorCopied
  ```

  