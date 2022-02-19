# SSH Key 配置问题 

<br/>

多个代码托管平台配置 SSH Key 时，GitHub SSH Key 配置失败



<br/>

###一、问题

我使用ssh-keygen生成了钥匙对，后把公钥分别复制到了gitee和github，然后我开始测试。

测试gitee时是OK的，测试github是出问题了，如下：

```shell
$ ssh -T git@gitee.com
The authenticity of host 'gitee.com (212.64.62.183)' can't be established.
ECDSA key fingerprint is SHA256:AQkjsKn/eya1W8icdBgrzp+KkGLAFgbVr17bmZjy0Wc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'gitee.com,212.64.62.183' (ECDSA) to the list of known hosts.
Enter passphrase for key '/Users/stone/.ssh/id_rsa':
Hi stone100! You've successfully authenticated, but GITEE.COM does not provide shell access.

$ ssh -T git@github.com
The authenticity of host 'github.com (140.82.114.4)' can't be established.
ECDSA key fingerprint is SHA256:p2QjAKc98/R1BUFWu3/LJKJM.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Host key verification failed.
```



<br/>

### 二、GiHub 的 SSH Key 无法配置成功的原因

问了一下度娘，有人说是缺少`~/.ssh`下缺少 known_hosts 文件导致的，我查看了一下这个目录下是有known_hosts文件的。

```shell
$ cat known_host # 查看known_hosts文件的内容, 如下:
gitee.com,212.64.62.183 一个算法名 AAAAE2VjZHNhLXkjdsItbmlzdHAyNTYAAAAIbmlzdHAyNTYdazfBBMuEoYdx6to5oxR60IWj8uoe1akjd1fKOHWztLqTg1tsLT1jkjdfmFjU46EzeMBV/6EmI1udRI6ljdkfjlszJHE=
```

查看了一下这个文件的内容，里面是gitee相关的配置，我猜是github无法从这个文件中找到要用的数据，因此而报错。事实上确实如此。

tips：第一次访问gitee是要求输入创建钥匙对时设置的密码，输入密码，按回车后会生成known_hosts文件。



<br/>

### 三、解决

```shell
$ cd ~/.ssh

# 1. 重命名known_hosts, 这样依赖访问github时发现没有known_hosts, 就会新建known_hosts文件.
$ mv known_host known_host.bak

$ ssh -T git@github.com
The authenticity of host 'github.com (140.82.114.4)' can't be established.
ECDSA key fingerprint is SHA256:p2QAMXNIC1TJYkfljf98/R1jksFWu3/LiykdjfkM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,140.82.114.4' (ECDSA) to the list of known hosts.
Enter passphrase for key '/Users/stone/.ssh/id_rsa':
Hi phoenix19900219! You've successfully authenticated, but GitHub does not provide shell access.

# 按照提示操作: 输入yes => 输入钥匙对的密码 => 验证成功!
# 此时, 你会发现当前目录下又重新生成了, known_hosts文件, 
# cat一下:
$ cat known_hosts # 输出如下:
github.com,140.82.114.4 ecdsa-sha2-nistp256 BAAAEJVjZHNhLXNoYTItbmlzAZNTYAAAAIbmlzdHAyNTYACABkaEmKSENjQEezOmxkZMy7opKgwKJkt5YRrYMjNuG5N87uRgg1jkbo5wAdT/y6v0mKVkjdd0WZ2YB/++Tiekzkg=

# 这时是github的相关信息


# 如何让 SSH key 的配置在两个网站上都有效呢?

# 把之前访问gitee时生成的known_hosts的文件 与 刚刚访问github时生成的known_host文件合并不就可以了?
# 确实是可以行的, 这就是之前为什么重命名known_hosts, 而不是直接删除它的原因.

# 合并两个known_hosts文件的内容: 
$ cat known_hosts.bak >> known_hosts # 把known_hosts.bak文件的内容追加到known_hosts文件

# 重定向符:
# > 表示新建, 如: echo hello > hello.txt
# >> 表示追加, 如: echo world >> hello.txt

# 然后测试一下gitee和github, 都验证通过:
$ ssh -T git@gitee.com
Enter passphrase for key '/Users/stone/.ssh/id_rsa':
Hi stone100! You've successfully authenticated, but GITEE.COM does not provide shell access.
$ ssh -T git@github.com
Enter passphrase for key '/Users/stone/.ssh/id_rsa':
Hi phoenix19900219! You've successfully authenticated, but GitHub does not provide shell access.
```



<br/>

### 四、总结

多个代码托管平台（gitee、github，gitcode，gitlab，自己架设的代码托管平台 ......）配置SSH Key的时候由于 `~/.ssh/known_hosts` 文件的冲突，可能导致一些托管平台无法访问。

解决办法是：

分别生成各个托管平台的known_hosts，然后把所有的known_hosts的内容合并到一个known_hosts中，这样所有的托管平台的 SSH key 都能配置成功了！

