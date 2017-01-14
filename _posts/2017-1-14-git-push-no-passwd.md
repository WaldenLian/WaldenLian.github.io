---
layout:         post
title:          Github无密码git push
categories:     [GitHub, Git]
description:    Github上项目根据最初git clone方式的不同，可能每次从本地git push需要Github用户名和密码验证，长期来看非常繁琐，本文介绍了相应的解决方法和基本原因。
keywords:       git, github, git push
---

这两天Github玩的比较多，在[Explore](https://github.com/explore)上找到了不少感兴趣的项目，也遇到了些问题。

### 问题

最近对自己的代码仓库 `git push` 时每次都需要Github的用户名和密码认证，如下图：

![git push with no passwd](/images/posts/git/git-push-passwd.png)

长期来看非常繁琐，影响效率，就研究了下相应的解决方法和基本原因。

### 解决方法

这段时间在深入学习Python，大部分脚本使用python3实现，以下是对应代码，基本就是更换了传输方式，在此之前请在Github上配置好[ssh key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)，这里不再详述。

```python
import subprocess

# Github Username
username = "WaldenLan"
# Repository Name 
repository = "Walden-DevOps"
# Update git command
remove = "git remote rm origin"
update = "git remote add origin git@github.com:" + username + "/" + repository + ".git"

# Check whether push is based on http/https request
subprocess.call("git remote -v", shell=True)

# Modification
subprocess.call(remove, shell=True)
subprocess.call(update, shell=True)

# Check whether push is based on git request
subprocess.call("git remote -v", shell=True)
```

如果觉得每次push到Github上的三步很繁琐，可以使用以下脚本，直接复制到对应代码仓库目录中，通过 `chmod +x ./push.py` 设置执行权限，即可以 `./push.py 'commit info'` 完成push。

```python
import sys
import subprocess

# Commit content
commit = "git commit -m '" + sys.argv[1] + "'"

# Git push procedures
subprocess.call("git add .", shell=True)
subprocess.call(commit, shell=True)
subprocess.call("git push -u origin master", shell=True)
```

### 原因

对于这个问题，首先需要了解下 `git clone` 的方式，从传输协议上大致分为两类：ssh和http/https，基于这两种协议通常会有以下两种clone方式:

<table width="100%">
    <tbody >
        <tr>
            <th width="20%">方式</th>
            <th width="100%">命令</th>
        </tr>
        <tr>
            <td>http/https</td>
            <td><code class="v-code">git clone https://github.com/username/repository</code></td>
        </tr>
        <tr>
            <td>ssh</td>
            <td><code class="v-code">git clone git@github.com:username/repository.git</code></td>
        </tr>
    </tbody>
</table>

两种方式中，通过ssh完成的 `git clone` 在后续 `git push` 时不需要反复输入用户名和密码认证，只需要在最初git clone时输入密码验证即可，当然，在此之前必须在Github上配置好ssh key，这里就不再详述这部分内容了。与之相对的通过http/https协议完成的 `git clone` 则需要反复进行密码验证。

其主要原因是因为ssh是通过传递RSA公钥完成身份验证和信息传递，也就是说在最初配置完成ssh key并授权认证之后，无需再进行身份验证即可完成后续操作。而通过http/https保存身份验证信息虽然可行但十分麻烦，需要借用[密码缓存机制](https://help.github.com/articles/caching-your-github-password-in-git/)，其中还存在一个时效性的问题，因此这种方式每次请求都需要身份验证，当然，这也因此省去了最初配置的麻烦，直接使用即可。

### 引用

* Basics of http/https & git [https://segmentfault.com/q/1010000008103122](https://segmentfault.com/q/1010000008103122)

在这里安利一个类似国内版的Stack Overflow，叫[segmentfault](https://segmentfault.com/),虽说目前社区规模不大，但还是值得期待一下它的发展。顺带吐槽一下Stack Overflow，平时在学校用或者开着Vpn用极为顺畅，但是不开Vpn的情况下需要等半分钟多才能显示出内容，原因是其中一个JS文件需要翻墙才能加载出来也是很迷。类似的还有三大MOOC平台之一的edX，在不挂Vpn的情况下可以看课程介绍但无法加载出Course Register的button。。。 = =.