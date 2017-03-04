---
layout:         post
title:          Background Process Issues related to SSH
categories:     [Linux]
description:    When launching a process in the background, the process log can't be caught in the foreground after exiting and re-logining the SSH.
keywords:       SSH, Linux
---

### Problem Introduction

During my internship, SSH is always involved in my daily work. So one issue occurs to me:

``` 
I launch the Django server in the background with the command: python3 manage.py runserver &
Then after doing some coding for my project, I exit the SSH. 
When I refresh the site page of my project, it's successively loaded as expected.

But when I relogin the SSH, although the server is running, I can neither catch the log of the server nor see the server process listed in the background processes with the command: jobs
```

```python
@requires_authorizationdsssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss
class SomeClass:
    pass

if __name__ == '__main__':
    # A comment
    print 'hello world'
```
### Principles

For the underlying principles, first we need to be clear about what do our background processes belong to. Why can we list the background process when we don't exit the SSH? That's because those background processes belong to the SSH process. When our SSH exits, we lose the control of those processes.
 
When I exit the SSH, the previously background processes will be mounted under the system level. In this case, the terminal won't have the permission to output the standard output/error from the server process. Thus we have to grep the server process, kill it, and relaunch it to catch the server log.

So can we drag that process back to the foreground? I think actually there may exist such strategies because if we can mount the process to the system level, the OS may also provide a way for us to take it back. But such strategies are not recommended. Because when those previously background processes are mounted to the system level, the OS won't maintain them in the future. Thus even if we drag those processes back, they may contain messy information or resources that we don't need, which share certain similarity with so-called [zombie processes](https://en.wikipedia.org/wiki/Zombie_process). So there is no need for us to take those processes back.


### Little Deeper

After discussing this issue with my internship mentor, he tells me that this issue is similar to the situation called ***Double Fork***, which is a common method to create daemons. That is, if we fork twice on a parent process, the resulted process *p* will go to the state where *p* is mounted under the system level, and the parent process has no control on *p*. I haven't dived into its study, but I think it's worthy to be tried.


### Basic Solutions

When we want to keep the server log after exiting our client-side remote server access tools, SSH can't do that. The alternative way is to apply the [VNC](https://www.realvnc.com/) instead. I have not tried that before, readers can follow the link if needed.